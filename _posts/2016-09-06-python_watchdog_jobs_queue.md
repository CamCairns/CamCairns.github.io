---
layout: post
title:  "A Python Watchdog with Threaded Queuing"
date:   2016-09-06
categories: [python]
---

So here's something cool I made this afternoon at work. Currently I am building an ETL pipeline that ingests some god-awful proprietary software data format type, decodes it into something useful, performs a number of validation and cleansing steps and then loads it into a speedy columnar database ready for some interesting analysis to be done. Once in production this pipeline will be handling 100's of individual files (each will be loaded as it's own separate table), ranging in sizes from 10MB through to ~0.5TB.

The database, while having a pretty damn fast write speed, doesn't seem to handle lots of concurrent write processes too well (if the DB is hit too hard it will rollback one or all jobs complaing of concurrency conflicts). For a robustness there is a real need to serialize the database load component of the pipeline. However we obviously don't want to constrain the entire ETL pipeline to a single process (the database load is only one of many processes that will utilise the transformed data). What we really need is a way to decouple the extract-transform from the load...

#### Enter our Watchdog powered job queue

The basic idea of the jobs queue is that we set up a background process to monitor a directory for changes. When a data set is ready to be loaded into the database a trigger file will be created in the directory. This triggers the watchdog to place a load job in a waiting queue

There are three key libraries that are used in our watchdog jobs queue:

* watchdog
* threading 
* Queue

I havn't used the [Python watchdog library](http://pythonhosted.org/watchdog/) before but it's pretty slick. It's a Python api layered over one of a number of watchdog tools depending on the OS you're using. All these tools detect changes in a directory structure and then perform a set action when a change is detected (for instance if a file is modified do X; if it is deleted do Y).

The threading and queue libraries provides a high level interface to manage threads. Threading is one approach to achieving parallelism and concurrency in Python (the other major one being multiprocessing). Given our jobs queue will be lightweight and memory non-intensive a thread is the perfect option. Queues operate as the conduits through which threads (or processes) communicate.

The code is in a git repo here with a simple example.
    
So lets walk through the code:

```python
 	if __name__ == '__main__':
	
	    # create queue
	    watchdog_queue = Queue()
	
	    # Set up a worker thread to process database load
	    worker = Thread(target=process_load_queue, args=(watchdog_queue,))
	    worker.setDaemon(True)
	    worker.start()
	
	    # setup watchdog to monitor directory for trigger files
	    args = sys.argv[1:]
	    patt = ["*.trigger"]
	    event_handler = SqlLoaderWatchdog(watchdog_queue, patterns=patt)
	    observer = Observer()
	    observer.schedule(event_handler, path=args[0] if args else '.')
	    observer.start()
	
	    try:
	        while True:
	            time.sleep(2)
	    except KeyboardInterrupt:
	        observer.stop()
	
	    observer.join()
```

 1. Lets start with the main bit, here we instantiate a Queue object, a thread object and run it in daemon mode and instantiate a watchdog. Lastly we place a second poll that waits for a `ctrl-C` kill command and if it recieves one shuts everything down gracefully.
 
	```python
	
		def process_load_queue(q):
		    '''This is the worker thread function. It is run as a daemon threads that only exit when
		       the main thread ends.
		
		       Args
		       ==========
		         q:  Queue() object
		    '''
		    while True:
		        if not q.empty():
		            event = q.get()
		            print "{0} -- Pulling {1} off the queue ...".format(datetime.datetime.utcnow().strftime("%Y/%m/%d %H:%M:%S"), event.dest_path)
		            log_path = os.path.join(os.environ["db_logging"], monetdb_load." + datetime.datetime.utcnow().strftime("%Y%m%d") + ".out")
		            with open(log_path, "a") as f:
		                f.write(datetime.datetime.utcnow().strftime("%Y/%m/%d %H:%M:%S") + "-- Processing {0}...\n".format(event.dest_path))
		            cmd = "cat {0} | while read command; do ${{command}}; done >> {1} 2>&1".format(event.dest_path, log_path)
		            subprocess.call(cmd, shell=True)
		            os.remove(event.dest_path)
		            with open(log_path, "a") as f:
		                f.write(datetime.datetime.utcnow().strftime("%Y/%m/%d %H:%M:%S") + "-- Finished processing {0}...\n".format(event.dest_path))
		            print "{0} -- Finished and sleeping ...".format(datetime.datetime.utcnow().strftime("%Y/%m/%d %H:%M:%S"))
		            time.sleep(10)
		            print "Awake!"
		        else:
		            time.sleep(1)
	
	```

 2. We also define a worker thread. The worker thread is intended to be a backgrounded process running indefinitely so we use an always true `while` statement. The worker thread then "polls" the queue every second, checking if any jobs have been placed on it.

	```python
	
		class SqlLoaderWatchdog(PatternMatchingEventHandler):
		    ''' Watches a nominated directory, when a trigger file is moved to take the ".trigger" extension
		        it proceeds to execute the commands within that trigger file.
		
		    Notes
		    ============
		    Intended to be run in the background
		    and pickup trigger files generated by the "${AI_MP}/intermediate_to_monetdb.mp" abinitio graph.
		    '''
		
		    def __init__(self, queue, patterns):
		        PatternMatchingEventHandler.__init__(self, patterns=patterns)
		        self.queue = queue
		
		    def process(self, event):
		        '''
		        event.event_type
		            'modified' | 'created' | 'moved' | 'deleted'
		        event.is_directory
		            True | False
		        event.src_path
		            path/to/observed/file
		        '''
		        self.queue.put(event)
		
		    def on_moved(self, event):
		        self.process(event)
		        
	```
 
 3. We also define a watchdog class, inheriting from the Watchdog `PatternMatchingEventHandler` class. In this code our watchdog will only be triggered if a file is moved to have a `.trigger` extension. Once triggered the watchdog places the event object on the queue, ready to be picked up by the worker thread
 
  
	
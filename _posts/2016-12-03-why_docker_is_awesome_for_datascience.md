---
layout: post
title:  "Why Docker is Awesome For Data Science"
date:   2016-10-21
categories: [docker, datascience, agile]
---

Docker has been a hot topic for quite some time now. I have been using Docker every day at work as a core component of my development, analysis and modelling workflow for around 7 months now and Dockerized applications act as a key vehicle of delivery for the Data Science department in which I work. I really like Docker and I thought I'd take an opportunity to step back and explain a couple of reasons why I love Docker and think it's a great tool for use in Data Science. 

## What is Docker?

There are lots of great resources out there on what Docker is and how to start using it so I want rehash any of that old stuff. The typical way to describe Docker is as a tool for building and deploying fully contained filesystems. The "fully contained filesystems" are called Docker containers and contain everything that is needed to run: code, runtime, system tools, system libraries – anything you can install on a server.

![You will love this whale](assets/posts/2016-12-03-why_docker_is_awesome_for_datascience/docker.png)

## Why is it Awesome for Data Science?

### 1. Puts the power of tool selection and configuration back into the hands of the users

For me this might be the biggest benefits, particularly in organisations. When you give developers; the people who are trying to solve a problem, the power to use the tools that suit them best to find the best solution you change the way people think about the problem, and methods of solving it. 

I see this as a massive culture shift from the traditional IT/tech procurement employed by a lot of companies. Open source tools and collaboration, elastic-computing resources (cloud), the ability to create custom work environments in an easy and shareable way all feed into giving users the power and flexibility to create the solutions that work best for a particular problem **AND** the ability to improve and change those solutions in an iterbable way when the problem changes.

I see Docker as embodying this empowerment of individuals to be as productive and successful as possible.

### 2. Escapes the conflicting libraries problem

You may experienced this problem before, you have found a great library `AmazeLib` you want to try out. Before you can install it you need to install a bunch of dependancies `dependA`, `dependB=2.03` and `dependC=1.02`. No problem you think, i'll just get those installed. But, disaster! Turns out that one of those dependancies is incompatible with another library you have installed. 

Docker to the rescuce. All you need to do is set up your fully encapsulated environment with just the libraries/packages needed for that analysis and you are good to go! 

There are alternative to solving this type of problem. For example, `conda` allows you to switch between environments with different python libraries installed. None offers the depth or flexibility to build the environment you want as Docker.


### 3. Makes it easier to share and replicate results and collaborate with others

Analysis only has value if you can share it with others. Say you have done some analysis and you wan to share it with another of your colleagues. Just pull the code off the repo right? Oh, wait first you need to install this list of packages and their depedancies. Are  you using `packageA=2.02.1`? Or `packageA=2.03.4`? How about `LibraryB`? Does the OS matter? All sorts of potential incompatibilities can arise when sharing bits of code.

There exists of course a way of managing libraies for different package managers (for example `requirements.txt` with `pip`) but these only go a small way to ensure a reproducible environment. 
	
Docker means you can configure and create an entire filesystem that is 100% replicable on any host machine with no effort. Just include the `DockerFile` describing the environment used in your work and anyone else can build and run it in the exact same way.
	
### 4. Reduces the time barrier involved in trying new tools

One of the most frustrating things about trying a new piece of OSS (or any software product) is getting it installed properly. Library mismatches or incompatibilities, envirornment configuration, compiler problems etc. all meant that often a significant amount of time and effort would need to be spent getting something to work before you could even begin to assess whether is was useful or not.

![http://xkcd.com/1742/](assets/posts/2016-12-03-why_docker_is_awesome_for_datascience/will_it_work.png)

Not anymore! Pick an open-source tool and chances are that someone will have created an image  that encapsulates a working version of that tool. It takes on the order of $$10^1-10^2$$ seconds to have a working environment to try that tool and see if it can help you do your job better. The time barrier involved in getting something to work before you can try it has almost completely evaporated.

### 5. If set up correctly offers a secure environment to experiment

A criticism that can leveled at giving individuals the power to configure environments is that it can be dangerous. Divesting power over tools individuals use in the environment they want has generally also meant divesting power over the security of those tools. This is where Dockers containerisation really stands out. If the Docker Engine is set up correctly (a topic for a whole new post) containers should act as isolated sandboxes. Nothing malicious inside them should be able to affect the host machine. Any potential damage is limited and the risk surrounding trying something new also drop proportionally.

### 6. Platform Agnostic

Building a working application in Docker means that it will work on what ever infrastructure is available. It is platform agnostic and will require minimal changes to run.
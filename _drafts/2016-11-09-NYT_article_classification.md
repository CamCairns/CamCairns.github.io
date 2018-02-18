---
layout: post
title:  "NYT Article Classification"
date:   2016-11-09
categories: [datascience, NYT, classification]
---

I built a simple NLP classifcation model using New York Times articles quite a while ago (late 2015 I believe) but I thought i'd put a little write-up here. You can find source code and jupyter notebooks in the [NYT_article_classication repo](https://github.com/CamCairns/NYT_article_classification). A version of the corpus generated in 2015 is [available to download here](http://camcairns-nyt-corpus.s3-website.eu-west-2.amazonaws.com/nyt_corpus.tar.gz).

### What are we modelling?

The NYT website publishes articles under lots of different categories (Arts, Politics, Sports, etc.). It would interesting to see how accurately we could classify these articles based on there text.

{: .center}
{%include image.html img="/assets/posts/2016-11-09-NYT_article_classification/nyt_article_classes.png" caption="NYT Article Categories"%}

Do to this we scrape 1010 articles from each of 5 article categories (so 5050 articles in total):

* Arts
* Business 
* Obituaries
* Sports
* World

The corpus is split into a train and test set and Bernoulli Naive Bayes classifier is trained on the training data and the holdout set is used to assess accuracy at the end.

### How did we get the data?

The data is pulled via the NYT API. 

#### The NYT Article Search API

First thing you have to do is signup and register as a developer. API Keys are assigned by API, so make sure you specify the [Article Search API](http://developer.nytimes.com/docs/read/article_search_api_v2).

Once you have recieved an API key set it as environmntal variable `nyt_api_key` in your `.bash_profile`:

    export nyt_api_key=<YOUR API KEY>

Even before you register, you can use the NYT's handy API Console to interactively test your queries: http://developer.nytimes.com/io-docs

The [Article Search API](http://developer.nytimes.com/docs/read/article_search_api_v2) is pretty flexible; you can call it with no parameters except for your `api-key` and it will return (presumably) a list of articles, in reverse chronological order, starting from Sept. 18, 1851. However, it only returns 10 articles per request. And it won't let you paginate beyond a `page` parameter of `100` (i.e. you can't go to `page` 100000 to retrieve the 1,000,000th oldest Times article). To put it another way, you can only paginate through a maximum of 10,000 results, so you'll have to facet your search.

A lot more information about the article can be pulled out, but for the classication project we only need the title and body text.


### What modelling approach did we take?

For this post we just apply a simple 'bag-of-words' and a Bernoulli Naive Bayes classifer. We perform a cross-validation parameter sweep over the smoothing parameter $\alpha$ parameter and pick the parameter that optimises mean-accuracy.

### What were the results?

Scoring on the hold out set 

{: .center}
{%include image.html
img="/assets/posts/2016-11-09-NYT_article_classification/confusion_matrix.png"
title="Confusion Matrix"
caption="Proportional Confusion Matrix"%}

{: .center}
{%include image.html
img="/assets/posts/2016-11-09-NYT_article_classification/std_of_confusion_matrix.png"
title="Standard Deviation of Confusion Matrix"
caption="Standard Deviation of Confusion Matrix"%}

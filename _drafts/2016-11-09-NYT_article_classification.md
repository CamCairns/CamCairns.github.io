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

For this post we just apply a simple 'bag-of-words' and a Bernoulli Naive Bayes classifer. We perform a cross-validation parameter sweep over the smoothing parameter $$\alpha$$ parameter and pick the parameter that optimises mean-accuracy.

#### Bayes Law

To understand the Naive Bayes algorithm we first need to understand Bayes Law. Bayes Law is easily derived and describes how a conditional probability $$p(A \vert B)$$ is related to $$p(B \vert A)$$

$$
p(A|B) = \frac{p(B|A)p(A)}{p(B)}
$$

assuming $$p(B) \ne 0$$. In the formula above:

* $$P(A \vert B)$$ (the likelihood of event $$A$$ occurring given $$B$$ is true) is known as the *posterior*. It is the degree of belief in $$A$$ having accounted for $$B$$.
* $$P(A)$$ is the *prior*, what is our initial understanding of the probability of $$A$$ absent any knowledge of $$B$$.
* $$\frac{P(A \vert B)}{P(B)}$$ is the *scaled likelihood*, and represents support obervation $$B$$ provides for $$A$$.

#### Use in text classification 

Ok, but how does this relate to text classification? Suppose the word "football" appears in an article, this probably adds to the likelihood that the article has been filed under the "Sports" desk, but by how much? Well we can use Bayes Law to tell us 

$$
p(\text{Sports} \vert \text{football}) = \frac{p(\text{football} \vert \text{Sports})p(\text{Sports})}{p(\text{football})}
$$

The RHS of the equation we can compute with enough pre-labeled data

* $$p(\text{football} \vert \text{Sports})$$ is just the probability that the word "football" appears in an article under the Sports desk. We can actually get an idea of this by running a `grep` on the NYT Sports corpus

		>> grep -r "football" Sports/ | wc -l
		208

	so 	$$p(\text{football} \vert \text{Sports}) = 208/1010 = 0.21$$ 
* $$p(\text{Sports})$$ is the probability that the article under the Sports desk. In our NYT corpus each of the 5 categories has an equal number of articles so $$p(\text{Sports}) = 0.2$$.
* $$p(\text{footbal})$$ is the probability that any article contains the word "football". Again, in our NYT corpus we can find this just by using `grep`

		>> grep -r "football" . | wc -l
		353

	so 	$$p(\text{football}) = 353/1010 = 0.35$$ 

All together

$$
p(\text{Sports} \vert \text{football}) = \frac{0.21 \cdot 0.25}{0.35} = 0.12
$$

Is that right?

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

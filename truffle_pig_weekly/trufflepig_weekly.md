# *TrufflePig* at Your Service

Steemit can be a tough place for minnows. Due to the sheer amount of new posts that are published by the minute, it is incredibly hard to stand out from the crowd. Often even nice, well-researched, and well-crafted posts of minnows get buried in the noise because they do not benefit from a lot of influential followers that could upvote their quality posts. Hence, their contributions are getting lost long before one or the other whale could notice them and turn them into trending topics.

However, this user based curation also has its merits, of course. You can become fortunate and your nice posts get traction and the recognition they deserve. Maybe there is a way to support the Steemit content curators such that high quality content does not go unnoticed anymore? There is! In fact, I am a bot that tries to achieve this by using Artificial Intelligence, especially Natural Language Processing and Machine Learning.

My name is *TrufflePig*. I was created and am being maintained by @smcaterpillar. I search for quality content that got less rewards than it deserves. I call these posts truffles, publish a daily top list, and upvote them.

In this weekly series of posts I want to do two things: First, give you an overview about my inner workings, so you can get an idea how I select and reward content. Secondly, I want to peak into my training data with you and show you what insights I draw from all the posts published on this platform. If you have read one of my previous weekly posts before, you can happily skip the first part and directly scroll to the new stuff about analyzing my most recent training data.

# My Inner Workings

I try to learn how high quality content looks like by researching publications and their corresponding payouts of the past. My working hypothesis is that the Steemit community can be trusted with their judgment; I follow here the idea of [*proof of brain*](https://steem.io/steem-bluepaper.pdf). So whatever post was given a high payout is assumed to be high quality content -- and crap doesn't really make it to the top.

Well, I know that there are some whale wars going on and there may be some exceptions to this rule, but I try to filter those cases or just treat them as noise in my dataset. Yet, I also assume that the Steemit community may miss some high quality posts from time to time. So there are potentially good posts out there that were not rewarded enough!

My basic idea is to use well paid posts of the past as training examples to teach a part of me, a Machine Learning Regressor (MLR), how high quality Steemit content looks like. In turn, my trained MLR can be used to identify posts of high quality that were missed by the curation community and did receive much less payment than deserved. I call this posts *truffles*.

The general idea of my inner workings are the following:

1. I train a Machine Learning regressor (MLR) using Steemit posts as inputs and the corresponding Steem Dollar (SBD) rewards and votes as outputs.

2. Accordingly, the MLR learns to predict potential payouts for new, beforehand unseen Steemit posts.

3. Next, I can compare the predicted payout with the actual payouts of recent Steemit posts. If the Machine Learning model predicts a huge reward, but the post was merely paid at all, I classify this contribution as an overlooked truffle and list it in a daily top list to drive attention to it.

### Feature Encoding, Machine Learning, and Digging for Truffles

Usually the most difficult and involved part of engineering a Machine Learning application is the proper design of features. How am I going to represent the Steemit posts so they can be understood by my Machine Learning regressor?

It is important that I use features that represent the content and quality of a post. I do not want to use author specific features such as the number of followers or past author payouts. Although these are very predictive features of future payouts, these do not help me to identify overlooked and buried truffles.

I used some features that encode the style of the posts, such as number of paragraphs, average words per sentence, or spelling mistakes. Clearly, posts with many spelling errors are usually not high-quality content and are, to my mind, a pain to read. Moreover, I included readability scores like the [Flesch-Kincaid index](https://en.wikipedia.org/wiki/Flesch%E2%80%93Kincaid_readability_tests) and syllable distributions to quantify how easy and nice a post is to read.

Still, the question remains, how do I encode the content of a post? How to represent the topic someone chose and the story an author told? The most simple encoding that is quite often used is the so called ['term frequency inverse document frequency'](https://en.wikipedia.org/wiki/Tf%E2%80%93idf) (tf-idf). This technique basically encodes each document, so in my case Steemit posts, by the particular words that are present and weighs them by their (heuristically) normalized frequency of occurrence. However, this encoding produces vectors of enormous length with one entry for each unique word in all documents. Hence, most entries in these vectors are zeroes anyway because each document contains only a small subset of all potential words. For instance, if there are 150,000 different unique words in all our Steemit posts, each post will be represented by a vector of length 150,000 with almost all entries set to zero. Even if we filter and ignore very common words such as `the` or `a` we could easily end up with vectors having 30,000 or more dimensions.

Such high dimensional input is usually not very useful for Machine Learning. I rather want a much lower dimensionality than the number of training documents to effectively cover my data space. Accordingly, I need to reduce the dimensionality of my Steemit post representation. A widely used method is [Latent Semantic Analysis](https://en.wikipedia.org/wiki/Latent_semantic_analysis) (LSA), often also called Latent Semantic Indexing (LSI). LSI compression of the feature space is achieved by applying a Singular Value Decomposition (SVD) on top of the previously described word frequency encoding.

After a bit of experimentation I chose an LSA projection with 128 dimensions. To be precise, I not only compute the LSA on all the words in posts, but on all consecutive pairs of words, also called bigrams. In combination with the aforementioned style and readablity features, each post is, therefore, encoded as a vector with about 150 entries.

For training, I read all posts between 7 and 17 days of age. These posts are first filtered and subsequently encoded. Usually, this leaves a training set of about 70,000 contributions. Too short posts, way too long ones, non-English, whale war posts, posts flagged by @cheetah, or posts with too many spelling errors are removed from the training set. The resulting matrix of about 70,000 by 150 entries is used as the input to a multi-output [Random Forest regressor from scikit learn](http://scikit-learn.org/stable/modules/generated/sklearn.ensemble.RandomForestRegressor.html). The target values are the reward in SBD as well as the total number of votes a post received.

After the training, scheduled once a week, my Machine Learning regressor is used on a daily basis on recent posts between 2 and 26 hours old to predicted the expected reward and votes. Posts with a high expected reward but a low real payout are classified as truffles and mentioned in a daily top list. I slightly adjust the ranking to promote less popular topics and punish posts with very popular tags like #steemit or #cryptocurrency. Still this doesn't mean, that posts about these topics won't show up in the top-list (in fact they do quite often), but they have it a bit harder than others.

A more detailed explanation together with a performance evaluation of the setup can also be found [in this post](https://steemit.com/steemit/@smcaterpillar/trufflepig-introducing-the-artificial-intelligence-for-content-curation-and-minnow-support). If you are interested in the technology stack I use, take a look at [my creator's application on utopian](https://utopian.io/utopian-io/@smcaterpillar/trufflepig-a-bot-based-on-natural-language-processing-and-machine-learning-to-support-content-curators-and-minnows). Oh, and did I mention that I am open source? No? Well, I am, you can find my blueprints in [my creator's github profile](https://github.com/SmokinCaterpillar/TrufflePig).

# Let's dig into my very recent Training Data and my Learnings!

Let's see what Steemit has to offer and if we can already draw some inferences from my training data before doing some complex Machine Learning!.

So this week I scraped posts between {start_date} and {end_date}. After filtering the contributions (as mentioned above, because they are too short or not in English, etc.) my training data this week comprises of {total_posts} posts that received {total_votes} votes leading to a total payout of {total_payout} SBD. Wow, this is a lot!

How are these payouts distributed among the posts? Well, on average a post received {average_payout} SBD. However, this number is quite misleading because the distribution of payouts is heavily skewed. In fact, the median payout is **only {median_payout} SBD**! Moreover, {no_earnings_percent} % of posts are paid less than 1 SBD! Even if we look at posts earning more than 1 Steem Dollar, the distribution remains heavily skewed, with most people earning a little and a few earning a lot. Below you can see an example distribution of payouts for posts earning more than 1 SBD and the corresponding vote distribution (this is the distribution from my first post because I do not want to re-upload this image every week, but trust me, it does not change much over time).

![earnings](https://raw.githubusercontent.com/SmokinCaterpillar/TrufflePig/feature/weekly_status/img/distribution.png)

Next time you envy other peoples' payouts of several hundred bucks and your post only got a few, remember that you are already lucky if making more than 1 Dollar! Hopefully, I can help to distribute payouts more evenly and help reward good content.

While we are speaking of the rhich kids of Steemit. Who has earned the most money with their posts? Below is a top ten list of the high rollers in my dataset.

{top10_earners}

Let's continue with top lists. What are the most favorite tags, and how much did they earn in total?

{top10_tags}

Ever wondered which words are used the most?

{top10_words}

To be fair, I actually do not care about these words. They occur so frequently that they carry no information whatsoever about whether your post deserves a reward or not. I only care about words that occur in 10% or less of the training data, as these really help me distinguish between posts. Next, let's take a look at on which features I really base my decisions on.

### Feature Importances

Fortunately, my random forest regressor allows us to take a look at the importance of the features I use to evaluate posts. I summarizes my 150 or so features into three categories: Spelling errors, readability features, and content. Spelling errors are rather self explanatory and readability features comprise of things like ratios of long syllable to short syllable words, variance in sentence length, or ratio of punctuation to text. By content I mean the importance of the LSA projection that encodes the subject matter of your post.

The importance is shown in percent, the higher the importance, the more likely the feature is able to distinguish between low and high payout. In technical terms, the higher the importance the higher up are the features used in the decision trees of the forest to split the training data.

So this time the spelling errors have an importance of {spelling_percen} % in comparison to style with {style_percent} %. Yet, the biggest and most important part is the actual content you post about, with all LSA topics together accumulating to {topic_percen} %.

You are wondering what these 128 topics of mine are? I give you the first twenty as an example below. Each topic is described by it's most important words with a large positive or negative contribution.

{topics}

So now I'll use these insights I have gained to dig for truffles. Watch out for my daily top lists!

## You can Help and Contribute

By checking, upvoting, and resteeming the found truffles of my daily top lists, you help minnows and promote good content on Steemit. By upvoting and resteeming this weekly data insight, you help covering the server costs and finance further development and improvement of my humble self.

**NEW**: You may further show your support for me and all the found truffles by [**following my curation trail**](https://steemauto.com/dash.php?trail=trufflepig&i=1) on SteemAuto!

## Delegate and Invest in the Bot

If you feel generous, you can delegate Steem Power to me and boost my daily upvotes on the truffle posts. In return, I will provide you with a *small* compensation for your trust in me and your locked Steem Power. **Half of my daily SBD income will be paid out to all my delegators** proportional to their Steem Power share. Payouts will start 3 days after your delegation.

Click on one of the following links to delegate **[1]({sp1}), [5]({sp5}), [10]({sp10}), [50]({sp50}), [100]({sp100}), [500]({sp500}), [1000]({sp1000}),** or even **[5000 Steem Power]({sp5000})**. Thank You!

Cheers,

{truffle_image}

*`TrufflePig`*


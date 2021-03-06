# Visualizing Your Reading Data

I'm going to give you a little bit of a spoiler alert: I've read the equivalent of about 14 books this past year. Now, I'm not a cover-to-cover novel reading person -- I consume most of my content in the form of articles and tutorials. So while I'm feverishly reading all the time, I never have a sense of how much I'm actually reading. After all, it's not like I'm exactly keeping track of how many articles I'm reading. 

But I could! One thing I didn't mention is that my reading flow is almost completely through [Pocket](). For those of you who don't know, Pocket is a convenient way to save content (whether that be in the form of articles, video, etc) for later use. This is especially important to me because it gives me an easy way of viewing content while I don't have internet service. In other words, when I'm on the NYC subway.

Since I always save what I read to Pocket, I can use their API to pull my archive data. With this data, we'll run through a simple analysis and wordcloud visualization. 

### Environment Setup

This guide was written in Python 3.6. If you haven't already, download Python and Pip. Next, you’ll need to install several packages that we’ll use throughout this tutorial:

```
pip3 install pocket==0.3.6
pip3 install wordcloud==1.3.1
```

We'll be using the Pocket API, so make an application and generate API keys [here](https://getpocket.com/developer/apps/new). 

Since we’ll be working with Python interactively, using the [Jupyter Notebook](http://jupyter.readthedocs.io/en/latest/install.html) is the best way to get the most out of this tutorial. Once you have your notebook up and running, you’re good to go!

### Getting Started

Assuming you've successfully generated your pocket API keys, we can call teh pocket client to begin. Your keys are associated with your account, so these are what will provide you with the data needed for this exercise. 


```python
from pocket import Pocket, PocketException
import json

p = Pocket(
consumer_key='your_consumer_key',
access_token='your_access_key'
)
```

If you look at the pocket documentation, you'll see that the `get()` method has a few parameters you can utilize. For the purposes of what we're trying to do, we'll set the parameters `since` and `state`. *Since* allows us to select a date from which to pull data from. And since we're trying to figure out how much we've read this year so far, we'll set `state` to `archive`. 


```python
# this retrieves all my readings since jan 1, 2017
lis = p.get(since=1483246800, state="archive")
```

If we print this get request, we'll see that it returned a tuple:


```python
print(type(lis))
```

    <class 'tuple'>


The second element of the tuple is just information about the get request, so it's not necessarily needed for this exercise. The first element, however, is a dictionary that contains information on the get request and, more importantly, the metadata for each read article. Within this dictionary, we actually only need the dictionary stored as a value to the "list" key. For simplicity, we'll set this dictionary to a variable:


```python
sub_list = lis[0]['list']
```

Before we continue on, let's take a look at what this json format is.

```
{'1781427846': {'item_id': '1781427846', 'resolved_id': '1781427846', 'given_url': 'http://gutsmagazine.ca/emotional-labour/', 'given_title': 'Three Thoughts on Emotional Labour – GUTS', 'favorite': '1', 'status': '1', 'time_added': '1498066507', 'time_updated': '1498404667', 'time_read': '1498404667', 'time_favorited': '1498404659', 'sort_id': 0, 'resolved_title': 'Three Thoughts on Emotional Labour', 'resolved_url': 'http://gutsmagazine.ca/emotional-labour/', 'excerpt': 'Recently a guy I went on a few dates with was going through a rough time. We were texting about it and I said “If there’s anything I can do to be supportive, please let me know.” He responded by asking me how he could accept support from me without exploiting my emotional labour.', 'is_article': '1', 'is_index': '0', 'has_video': '0', 'has_image': '0', 'word_count': '2494'}
```

Shown above is just one of the articles in the data. I'll highlight a few parts: The main key is (1781427846 above) the ID for each element. Within the dictionary that serves as the value to this key are a couple of keys that will be important for this exercise. First, we have `word_count` which will help us keep track of total word_count. Next, we have the `given_title`, which is the cleaned version of each article title.

Before we move on, let's review what we're going to accomplish the following two tasks:

- estimate about how many books did you read so far this year
- make a word cloud based on the titles of content read

Given this, we'll begin by starting a count for all the words read and a string that will consist of every single title in our pocket archives.


```python
words = 0
words2 = ""
```

Awesome! Now we're ready to get going with our analyses. As we iterate through the dictionary by IDs, we'll check to see if the `word_count` and `given_title` key words exist. If they do, we'll add the word count to the `words` parameter and concatenate the string to the `words2` string.


```python
for i in sub_list: 
    if 'word_count' in sub_list[str(i)].keys(): 
        words += int(sub_list[str(i)]['word_count']) 
    if 'given_title' in sub_list[str(i)].keys():
        words2 += " " + sub_list[str(i)]['given_title']
```

According to [Huffington Post](http://www.huffingtonpost.com/2012/03/09/book-length_n_1334636.html), the average book contains about 64,000 words. To estimate the number of books we've read this year so far, we'll take the total word count and divide it by 64,000. 


```python
print(words/64000)
```

    13.69678125


As I said before, according to my Pocket data, I've read about 13.7 books this year! 

### What are you reading? 

Python has a library `wordcloud` that provides functions to generate an image of our the frequent words in a given text. Using the string of every single title we've put together, we can use the `wordcloud` module to create our wordcloud visualization. 


```python
from wordcloud import WordCloud, STOPWORDS
import matplotlib.pyplot as plt
```

Notice we also imported `STOPWORDS` from the wordcloud module. These are to keep from visualizing words like "the", "and", "or" from appearing in the wordcloud. Words like these will clearly occur very frequently, but give no insight as to what topics we're reading about. So the set of stop words will be removed from the final wordcloud visualization: 


```python
stopwords = set(STOPWORDS)
```

Often times, the titles of articles contains the name of the publication. Since I read The New Yorker, Medium, and New York Times, I decided to just remove them. 


```python
nsw = ["medium", "new", "york", "times", "stop"] 
for i in nsw:
    stopwords.add(i)
```

This might be hard to believe, but now we can initialize the wordcloud object! This object is what represents the image we'll use `matplotlib` to output. 


```python
wordcloud = WordCloud(stopwords=stopwords, background_color="white")
```

As a last step for creating the wordcloud object, we fit the model to the string of all titles:


```python
wordcloud.generate(words2)
```




    <wordcloud.wordcloud.WordCloud at 0x10fdeed30>



And finally, we invoke `matplotlib` to display our image. For this example, we won't do any special customizes, but in case you're interested in how to go about doing this, check the documentation [here]().


```python
plt.imshow(wordcloud)
plt.axis("off")
plt.show()
```


![png](/_posts/output_28_0.png)


And as though it wasn’t glaringly obvious before, I read lots of articles and posts about Python, Data Science, and Machine Learning. Go figure.

In this tutorial, we used an API to analyze what our everyday content consumption looks like. For me, that meant highlighting that I spend a good portion of my type learning or improving upon topics that fall under the realm of data science. While this is my particular wordcloud visualization, anyone can complete the same task so long as you have a Pocket account!
Happy reading! 

If you liked what we did here, [follow me](https://twitter.com/lesleyclovesyou) (@lesleyclovesyou) on Twitter for more content, data science ramblings, and most importantly, retweets of super cute puppies.

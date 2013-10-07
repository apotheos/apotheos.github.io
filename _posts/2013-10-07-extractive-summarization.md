---
layout: post
title: "Extractive Summarization"
description: "Extractive Document Summarization with NLTK"
category: articles
tags: [csc, thesis, nlp]
comments: true
# permalink: "/path/to/permalink.html"
published: true
---

Chapter 4 Problem 32
====================

```
32. Develop a simple extractive summarization tool, that prints the sentences of a document which contain the highest
total word frequency. Use FreqDist() to count word frequencies, and use sum to sum the frequencies of the words
in each sentence. Rank the sentences according to their score. Finally, print the n highest-scoring sentences in
document order. Carefully review the design of your program, especially your approach to this double sorting.
Make sure the program is written as clearly as possible.
```

Code in [summarizer.py](https://github.com/apotheos/CSC499-NLP/blob/master/ch_4/exercises/summarizer.py).

Architecture
------------

Before I could start this problem, I wanted to think about how the process works algorithmically.

I know that I need to:

* find the frequency distribution of an entire article
* get the sentences for that article
* for each sentence, get the frequency of each word
* sum the list of frequencies and store it alongside the sentence
* be able to sort the results in various ways

Now that I know how to tackle the problem, I began architecting a solution. I created a class called
`Summary` and contained within it the list of setences, then created a `summarize()` function to start
the calculations.

I also noted that each sentence needs to have information stored along with it:

* its relative position in the article
* its "score"
* and of course the sentence itself

To rectify this problem, and organize the data in a clean fashion, I created a `ScoredSentence`
class that stores the sentence's position, score, and sentence itself. It also overrides the
`__cmp__` magic method allowing the sentences to be sorted by score. The `summarize` method
in `Summary` creates a list of sentences, calculates the scores, and then sorts and returns
the requested amount of data.

Results
-------

Initial results from this algorithm were promising, but could use some refining. When testing
against the Verge's review of new release movie *Gravity*, the results were as follows:

* From the encumbered , awkward way the astronauts maneuver in their suits , to the cozy interior of the International Space Station , the film is filled with iconic imagery that people have grown up with thanks to NASA and the nightly news . 
* Gravity isn ’ t so much a sci - fi movie as it is a survival film : two people against the elements , only as the film ’ s opening title card reminds us , it ’ s in the harshest environment possible .
* Much has been written about the movie ’ s mix of live - action and computer - generated imagery — the majority of sets , ships , and even costumes were created digitally — and while Gravity contains some of the most breathtaking visual effects work in recent years the focus isn ’ t on sheer spectacle .

### Stop Words ###

Although these results aren't bad, there's still some problems with it. Most noticably, these sentences are rife
with stop words. Since they are most common, they're making the score of the whole sentence higher. Removing them
requires changing the architecture a bit to store two copies of the text in the `Summary` object: the original
data and the preprocessed text. After making these adjustments, here's what happens when we remove the
stop words from the calculation:

* Gravity isn ’ t so much a sci - fi movie as it is a survival film : two people against the elements , only as the film ’ s opening title card reminds us , it ’ s in the harshest environment possible .
* It ’ s been a year of disappointing sci - fi , and while the fantastical worlds of Star Wars and Star Trek aren ’ t going anywhere , maybe everyone else will learn to put the brakes on their action - movie antics . 
* Much has been written about the movie ’ s mix of live - action and computer - generated imagery — the majority of sets , ships , and even costumes were created digitally — and while Gravity contains some of the most breathtaking visual effects work in recent years the focus isn ’ t on sheer spectacle .

### Stemming ###

These results have two of the same results, so maybe they were most representative of the article after all. However,
removing the stop words did help with the one entry. Another thing we could do to refine the results is to stem
everything. After using NKTK's `Snowball EnglishStemmer`, we get results that look like this:

* Gravity isn ’ t so much a sci - fi movie as it is a survival film : two people against the elements , only as the film ’ s opening title card reminds us , it ’ s in the harshest environment possible .
* It ’ s been a year of disappointing sci - fi , and while the fantastical worlds of Star Wars and Star Trek aren ’ t going anywhere , maybe everyone else will learn to put the brakes on their action - movie antics .
* Much has been written about the movie ’ s mix of live - action and computer - generated imagery — the majority of sets , ships , and even costumes were created digitally — and while Gravity contains some of the most breathtaking visual effects work in recent years the focus isn ’ t on sheer spectacle .

Unfortunately, nothing changes here. That's alright though: even though it didn't change the results, it still models
the problem better than the previous approach did.

### Managing Length ###

While implementing the algorithm I realized that longer sentences are more likely to be in the result, because the
scores for sentences are based on "size", not "density". In an attempt to correct this, I tried dividing the score
for each sentence by the number of words in the sentence. This makes sense in theory, but the result was not good:

* It's on familiarity.
* It's reality building.
* One with rules, physicality, and consequences.

Why did this happen? Well, now we're at the opposite end of the spectrum. Instead of having long, descriptive sentences,
we now have short concise ones. Although they're are a few very descriptive words in the results above, they're lacking
in one area: context. It seems like shorter sentences just don't have the context that we desire. They're more likely
to be fragments. Also, if it's a summary we want, we will get more information from longer, descriptive sentences.

It's possible there is a way to work something in that does limit sentence length, but not quite to this degree, but
it would require some more thought.

One other thing I attempted 

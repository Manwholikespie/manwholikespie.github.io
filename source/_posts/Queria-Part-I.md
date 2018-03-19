---
title: Queria Part I
date: 2017-02-09 01:57:09
tags: ["natural language processing","fuzzy set theory","python","web scraping"]
---
# Introduction
Queria originally started as nano, a question-answer AI. James and I worked on it over spring break in William's cabin in the wee hours when we got bored of eating quesadillas and watching anime. This was also the project that sparked the creation of Inculi.

The concept was pretty simple: ask the bot some questions, and have it find you the answers to those questions. Being able to actually make this happen was the hard part. We didn't really have access to a giant conversational database, nor the means to train any sufficiently powerful computer on this dataset. So, we figured a more feasible target would be answering questions on our homework.

How do we do that? Through google. More specifically, wikipedia and quizlet. More technically-- with fuzzy string matching algorithms and some neat set theory functions like the [Jaccard Index](https://en.wikipedia.org/wiki/Jaccard_index) and [Hamming distance](https://en.wikipedia.org/wiki/Hamming_distance).

For multiple choice questions, you input the question with a list of possible answers. The program then searches the question on google, and takes the top 20 page results (excluding pdfs, powerpoints, etc.). In those 20 pages, it tokenizes the core text into individual sentences, and then performs a token set ratio comparing the question to each sentence. If a paragraph has enough similiar sentences, it saves it. Later, once all 20 pages have been parsed, it counts up how many times each of your given possible-answers show up, and assigns points to each possibility based on that count.

For the free response option, it basically does the same sort of search, but only uses the top three results from quizlet. Then, instead of performing a token set ratio on the sentences of the page, it performs it on the cards to find the relevant cards. No possible answers are needed.

Now that you are all caught up, here are some of my latest thoughts regarding using wikipedia summaries to aid the quizlet data in free response questions...

## Article Reading Function

Sadly, the 'easy' part is over. Though, I am glad that I shall now be moving on to something more challenging and important in the grand scheme of this project– assuming it works. Anyways, this is the plan for what I am going to do next:

When researching the answer to a given question, there is of course the task of finding a relevant resource with the information you are looking for. I believe the google search module does a good job of leaving this task to the search giant. Now, the next part is of course to actually read the article and pull the relevant information from the text into your brain. In other words, I get to play around with string matching and processing.

I figure if I can pull apart my question into its basic words, and then possibly feed those words to the thesaurus package I created earlier, I can then feed that data into the fuzzy string matching module that I used in the quizlet portion of queria.

**Let’s Begin.**  
I was able to get the data from the pages by using the wikipedia module:  

```python
pageText = wikipedia.WikipediaPage(topic).content
```

Then, I broke the text into single sentences with nltk’s sentence tokenizer, storing them into a list:  

```python
pageSentences = nltk.sent_tokenize(pageText)
```

This ended up being fairly straightforward, allowing me to use fuzzywuzzy’s token set ratio to single-out sentences that have a ratio above 80:  

```python
if int(fuzz.token_set_ratio(inputQuestion.lower(), item.lower())) > 80:
	relevantSents.append(item)
```
Experimentally, the lowest-scoring sentence I have found that had relevant data received a score of 84.

TODO…
Currently, the data is somewhat out of order. Firstly, the page text that the wikipedia module returns is filled with chapter markers denoted by:  

>	=== HISTORY ===  
>		Blah blah George Washington. Much war.  
>	=== DEATH ====  
>		He’s dead. :.(  

Which can be nice for the purpose of organization and such, however when I tokenize it, it’s still left in there. I figure before I tokenize it, I can go through the unicode object with some regex to filter it out. We’ll see… regex can be difficult sometimes.

As for what I want to add:
- *DONE* I want to store the ‘relevant’ sentences alongside their fuzzy score so that I can construct the paragraph with the most helpful information at the top.
- I need to figure out what the "mult" files do. I believe they had something to do with importing questions, which reminds me that I need to finish that as well (or at least perfect it).
- I want to possibly add a webgui to view the answers to my questions. At the very least, have it construct an html file for me to view the answers in, as it is really difficult to read off of this terminal sometimes.
- Experiment with googling questions EXACTLY as they are written (with quotation marks) and prioritizing those pages in searches.


## 2/9/17
Greetings und achtung, my dear children, and welcome back to another rendition of queria.
I am starting to think again about putting these subpar(-ly?) organized functions into their respective files, but I am not quite sure yet, as last time that didn't work out so well.

I want to work on the answerFree() function a bit, mainly incorporating these two technologies:  
- [sumy](https://github.com/miso-belica/sumy)  
- [python-goose](https://github.com/grangier/python-goose)  



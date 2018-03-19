---
title: Follow the Leader Part III
date: 2016-10-05 01:28:24
tags: ["python","web scraping"]
---

## 10/3/16
Greetings, and welcome back to the FTL project after a long hiatus. It has come to my attention that there are quite a few things that could be improved upon, mainly efficiency and logging, for the sake of knowing what the damn hell is going on at a given time.

For starters— efficiency. Currently, we have bs4 going through tens of thousands of pages, grabbing user data one page at a time. Which would be great, if we were programmers in 18th century Kenya. We’re better than that. Therefore, gamescan.py will be overhauled using the beautiful aria2c engine, so that we can download our 16 beautiful pages at a time.

Let’s begin.

## 10/4/16
Good news, my dear children. While I didn’t want to tackle the overhaul of gamescan, I figured it would be easy enough to change games.py. Here are the results:  

Old games.py times:  
- 17.94 real         1.36 user         0.07 sys  
- 18.12 real         1.40 user         0.07 sys  
- 15.48 real         1.35 user         0.07 sys  

New games.py times:  
- 7.52 real         1.19 user         0.10 sys  
- 6.24 real         1.20 user         0.10 sys  
- 8.93 real         1.20 user         0.11 sys  

Pretty damn good improvement, imho.

Now, I am going to change gamescan as CAREFULLY as I possibly can.
I figure the best way to attack this file would be to first examine carefully what it does. Remember, you haven’t touched this file in months.

After collecting the list of games with getGames(), it sends the game name to gameScan().
Within gameScan(), there is the function pageCounter(), which visits the first page of the game, and extracts a certain paragraph tag that holds the amount of players. Because we know that statistically only the first 1/3rd have positive net worths, this number is divided by three and rounded accordingly.

```python
def gameScan(gameName):
	# does a bunch of things (an understatement), but I want to highlight the fact
	# that we are calling pageCounter here.
	pageCounter(gameName)

def pageCounter(gameName):
	while x < numberOfPeople:
		list.append('marketwatch.com/'+gameName+'/ranking?'+str(x))
		# Output: list of urls for the pages we need to scrape.
```

### Todo
- Create a list of the urls for the first page of the games we need to scan, and append this to a list.
- Go Through this list and use aria2c to download the pages.
- Create a function that will go through the files for these pages, find the number of people within the game, and create a list of proper urls for the correct number of pages that need to be indexed. It is possible that I may be able to avoid one more url request by having pageScan accept multiple input types. Whenever I am parsing the page for the number of players, I might be able to send over the file if the number of people < 10, as I wouldn’t need any pages past this. Most of these games aren’t any more than 2 pages, so that might cut time in half.
- Download these pages to be parsed by pageScan.  
- Delete pageCounter
- Rewrite pageScan to open files instead of urls.
- Make sure that we keep the url in there somehow, most likely as the title of the file. There are some regex functions that will need some of the data stored here.  

Let’s begin, my dear children.

## 10/5/16
Alright, so I was able to complete the majority of the changes that I needed to, however pageScan is a *little* difficult. The main problem with this is just how much data it pulls from the url, which cannot exactly be saved in the file title.

The easiest way to address this would be to keep the regular expression things the same and just rebuild the url when it first opens up the file for parsing. I figure the easiest way to format the file name would be to use commas as OS X allows this in their file system. I’m not too sure about windows, however, but idc.

Well, the TODO list has been extended. I finished converting pageScan() to use downloaded pages, however I forgot about the playtime() function. Seems that this is slowing the program down quite a bit, which is strange as it shouldn’t be adding more... Nevermind. WE’RE DOWNLOADING 10,000 DAMN PAGES FOR THESE USERS. Time to make some changes.

It looks like I am downloading the playtime of these users before even filtering them out by percent gains or net worth. So rather than downloading ~500 pages I am downloading 10k.
This is the time to sigh deeply. Time to delve deep into the documentation of pandas again so more optimizations can be made.
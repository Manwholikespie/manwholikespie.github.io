---
title: Follow the Leader - Part I
date: 2017-05-07 23:38:50
tags: ["python", "web scraping"]
---
# Follow the Leader
Welcome to the notes document for the project “follow-the-leader”. This document contains notes regarding brainstorming, observations, and problem-solving.

## Brainstorming
### TRANSACTIONS
**HOLDINGS**: I realized that currently, all of my holdings are stored in a dictionary that does not take into account multiple users. Granted, the program as a whole still doesn’t really account for multiple users. That’s okay, though. In order to combat this, I need to store transactions as being user-specific. This is the way I’m thinking of setting up the dictionaries: 
 
```python
Holdings =	{
	‘Dustin Lien 234858’ : {
			‘AAPL Buy’ : 674
			‘AAPL Short : 536
			‘HDP Buy’ : 246
			‘NDRO Buy’ : 643
			}
}
```

**RATIOS**: Incorporating the multiple-users thing should be fairly simple when it comes to the ratios. It should be as simple as inputting a (1/50) in-front of the amount to buy– right?

**LOSSES**: Question: If a user drops out of the top 50, most likely due to their losses– or if their game ends– what would be the easiest and safest way to extract them from the “top 50” pool? I want to minimize losses, of course. My current planning:
Use gamescan() to find the top players of the day
Cross-reference the new list with that of yesterday
Based off of the differences, create a list of users to add and users to drop.
Users to drop:
Look at the holdings.log file, and create a function that sells all of the stock for a given user.

## Hypothesis
There are a few ways that this could go, insofar as returns. Let’s assume the best-case average scenario. Users trade an average of 8 (7.9) times per day. Removing the outliers, they get about 4.46% in returns each day, giving 133.8% a month. Now, assuming we go with the cheating outliers, the average raises to 64.78– giving 1,943.4% in 30 days.
Debugging
### TRANSACTIONS
**REWORK**: So, as you may have imagined due to the fact that I still haven’t finished this damned program, there are a few problems with the current way I am doing transactions. Mainly, there are way too many sources of error in monitoring transactions as per their list of past transactions. The sources of error mainly stem from how many things have to go perfectly in order for everything to be “okay”. Here are a few things:
In order to be copied, a trade has to be made in the past 5 minutes. Because I can’t always guarantee that a trade will be made in the past 5 minutes, there will be discrepancies between how a player performs in the game, and how he performs under my tutelage.
I can only copy trades that are in the present. If a user already has a holding (let’s say he bought Tesla a few months ago and plans on holding onto it for a while), I will not be able to enjoy the returns until he sells it and buys it again.
Some people have scripted their purchases, and therefore if I am monitoring 50 users, it may be 2+ minutes or so before I get to scan a user again. If a user is makes over 10 trades in this 2 minute period (a very possible scenario), they will scroll of the page of the most recent transactions and I will not see them– leading to another discrepancy.

So what is my solution? I believe it is smarter to look at holdings rather than transaction history. Then, I can find the ∆ value between holdings at different time periods. Example:

```python
John’s Holdings @10:30AM = {
	‘AAPL Buy’ : 100,
	‘GOOG Short’ : 100,
	‘TSLA Buy’ : 100,
	‘NKE Short’ : 100
}

John’s Holdings @10:35AM = {
	‘GOOG Short’ : 200,
	‘TSLA Buy’ : 50,
	‘NKE Short’ : 100
}
```

### WHAT NEEDS TO HAPPEN:
Make a function that returns the difference between two dictionaries like the ones seen above.
Make a function that creates a dictionary based on current holdings
Input: inputUrl
Output: {'NKE Buy': 581, 'SONC Buy': 943}
Make a function that reads a pickle file to find the holdings of our last scan
Make a function that finds the difference between the two dictionaries, and returns those stocks that have changed and the amount to which they have changed.
If there has been a change in the holdings, make the necessary changes to our own holdings, and update the pickle file to respect the report of his latest holdings.
Else if the dictionaries are the same, leave the pickle file and start scanning another user (or just sleep and prepare to scan again in a little while).

### RATIOS
**REWORK**:
Yay! This weekend’s changes have been completely implemented, meaning that I can now copy the transactions of a given user. There are a few problems– more with the system than the program– that need to be addressed, however. I will get to those later. As for now, there is another important thing that needs to be done: implementing ratios.

I plan on changing this a lot, meaning that may need to import these settings from a yaml file or something of the sort. I say yaml because I want a dictionary-type file that is easy for the user to edit and understand. Until I can add these “settings”, there are two main things that need to be done:

Given a file with a list of user-holding-URLs, I need to be able to count the amount of users that we care about. I also need to be able to of course read the urls.
“amountOfUsers” is important, as it shall be required for the primary ratio:
(1 / amountOfUsers )
This formula is used in the formula:
amountToBuy = (1 / amountOfUsers ) * ((myNetWorth/hisNetworth) * inputAmount))
However, this variable should be as a global when the program is first initiated, as I don’t want to read the file everytime I make a trade.




###RANKING
**REWORK**:
Alright, so it seems to be that what was done yesterday is working well now. However, there are a few other changes that need to be made if I am to get that “99.9%” efficiency that I was talking about earlier. One of those things is that I have to make some changes to the ranking algorithms so that the players, are of course, better. Here is a list of things that have to be done…

For players that have the same name, only keep the one that has the higher mmScore. This is to prevent people like Tom Riddle from taking 14 of the top 50 places in my ranking system
Remove duplicate names from the top 250 before cutting it down to 50.
There should be some stackoverflow answers on how to do this inside the pandas dataframe. Probably a combination of the list(), and set() functions would also work on the list of names. I may have to put it in an xrange() loop in order for it to remember the placement of the name among the other users in the ranking system.
As of now, there are players in the system that either 1) hardly make any trades or profit at all, or 2) hardly ever have any holdings on their person.
This needs to be fixed probably in the 250 —> 50 merge, as I will not be able to look at the performance of 500 players in a short amount of time. This will most likely have to be a new function that is able to deduce a crap-ton of data from the performanceURL alone… And I thought I was almost done…
Most of the work in this section will be fairly simple boolean operands, in which I can see if the user had any holdings on a particular day, and their percent returns for that day. What I am mainly looking for is if they made any trades in a given week… I don’t think it really stores this data (at least not in the performance section), though, so I might have to just count up the amount of times in a given week on their transactions page.
Set up the system that finds the difference between the “top 50” list for two given days (today and tomorrow– because remember, we’re scanning at ~5:00PM), and returns a list of users that need to be added, and a list of users that need to have all of their holdings sold.

----10/3/16
Greetings, and welcome back to the FTL project after a long hiatus. It has come to my attention that there are quite a few things that could be improved upon, mainly efficiency and logging, for the sake of knowing what the damn hell is going on at a given time.

For starters— efficiency. Currently, we have bs4 going through tens of thousands of pages, grabbing user data one page at a time. Which would be great, if we were programmers in 18th century Kenya. We’re better than that. Therefore, gamescan.py will be overhauled using the beautiful aria2c engine, so that we can download our 16 beautiful pages at a time.

Let’s begin.

----10/4/16
Good news, my dear children. While I didn’t want to tackle the overhaul of gamescan, I figured it would be easy enough to change games.py. Here are the results:

Games.py times:
       17.94 real         1.36 user         0.07 sys
       18.12 real         1.40 user         0.07 sys
       15.48 real         1.35 user         0.07 sys

New Games.py times:
        7.52 real         1.19 user         0.10 sys
        6.24 real         1.20 user         0.10 sys
        8.93 real         1.20 user         0.11 sys

Pretty damn good improvement, imho.

Now, I am going to change gamescan as CAREFULLY as I possibly can.
I figure the best way to attack this file would be to first examine carefully what it does. Remember, you haven’t touched this file in months.

After collecting the list of games with getGames(), it sends the game name to gameScan().
Within gameScan(), there is the function pageCounter(), which 

gameScan():
Calls pageCounter()
pageCounter(gameName):
visits the first page of the game, and extracts a certain paragraph tag that holds the amount of players. Because we know that statistically only the first ⅓ have positive net worths, this number is divided by three and rounded accordingly.
while x < numberOfPeople: list.append(marketwatch.com/<gameName>/ranking?<x>)
Output: list of urls for the pages we need to scrape.

TODO list for the updated version:
Create a list of the urls for the first page of the games we need to scan, and append this to a list.
Go Through this list and use aria2c to download the pages.
Create a function that will go through the files for these pages, find the number of people within the game, and create a list of proper urls for the correct number of pages that need to be indexed.
It is possible that I may be able to avoid one more url request by having pageScan accept multiple input types. Whenever I am parsing the page for the number of players, I might be able to send over the file if the number of people < 10, as I wouldn’t need any pages past this. Most of these games aren’t any more than 2 pages, so that might cut time in half.
Download these pages to be parsed by pageScan.
Delete pageCounter
Rewrite pageScan to open files instead of urls.
Make sure that we keep the url in there somehow, most likely as the title of the file. There are some regex functions that will need some of the data stored here.

Let’s begin, my dear children.

----10/5/16
Alright, so I was able to complete the majority of the changes that I needed to, however pageScan is a *little* difficult. The main problem with this is just how much data it pulls from the url, which cannot exactly be saved in the file title.

The easiest way to address this would be to keep the regular expression things the same and just rebuild the url when it first opens up the file for parsing. I figure the easiest way to format the file name would be to use commas as OS X allows this in their file system. I’m not too sure about windows, however, but idc.

Well, the TODO list has been extended. I finished converting pageScan() to use downloaded pages, however I forgot about the playtime() function. Seems that this is slowing the program down quite a bit, which is strange as it shouldn’t be adding more — fuck. Nevermind. WE’RE DOWNLOADING 10,000 DAMN PAGES FOR THESE USERS. Time to make some changes.

It looks like I am downloading the playtime of these users before even filtering them out by percent gains or net worth. So rather than downloading ~500 pages I am downloading 10k.
Well, fuck. Time to delve deep into the documentation of pandas again so that I can insert this POS code somewhere else.


11/15/16
As the last time I worked on this was over a month ago, I can’t exactly say that I remember how far I got with converting playTime() to working with downloads, however it has come to my attention that regardless of how quickly these things download, the main problem that is stagnating progress for this application is that I have to carefully schedule times when I can debug it. This is because by the time I decide to work on this, I first have to wait 45 minutes for all of the player data to download (in order to rank them).

Therefore, it would be best if I could set up some sort of crontab on my server at home to rank the players ~30mins after the market closes. This way, I will have a players.csv file ready for me when I want to work on it. Just FYI, though, I might want to hold off actually setting up the crontab until I can set aside some time for me to actually work on this. I don’t want to have marketwatch blacklist my IP before I can work on this.

This should be a fairly simple feat (I really just need to come up with some sort of naming system that will save the scandate as the filename, so until then— I’ll make some more notes on how I could convert optimize playTime() / convert it to a download function.

It should be simple enough to cut the time down by making playTime() calculate after we cut it from 3900 players to 1000.

11/17/16
Alright, so I added some more stuff to clean out the duplicate users. All I need now is to iterate through the master dataFrame and remove the users that do not have the higher percentReturns… However, I do have one problem with doing this… A lot of this depends on ranking based off of the total mmScore for a user. However, if I am removing the lower duplicate users based just off of their totalPercentReturns alone, I could be missing out on some better players. Maybe they switched accounts to another game, and are starting to perform better on there… I wouldn’t be able to know until they overtake the performance of their other player… We shall see what happens, though. I still think it is necessary for me to make this change, as I don’t want that thing with Tom Riddle to happen again where he takes up 30% of the top 50 player rankings.

In order to remove these faulty rows, I just need to find the right way to delete rows in the pandas documentation… so that is the next step.


11/22/16
Alright so I’m now working on automating the execution of gamescan() so that I don’t have to do it manually every day. It is important that it has a fresh csv to work with. Here is the general process it should run every single day:

Find today’s date and weekday.
today = str(datetime.datetime.now()).rsplit(" ",1)[0]
today = '2016-11-22'
Keep going backwards in time until it finds the latest existing csv file.
pastDay = str(datetime.datetime.now()-datetime.timedelta(days=1)).rsplit(" ",1)[0]
daysAgo = 1
while os.path.isfile(pastDay+'.csv') == False
daysAgo+=1
pastDay = str(datetime.datetime.now()-datetime.timedelta(days=daysAgo)).rsplit(" ",1)[0]


If one already exists for today, exit.
If today is monday, it should still check for ones made on the weekends— however:
if it is creating one, it should check to see if one was already made Friday… nothing has changed since then. If one wasn’t made on Friday, make one for today.

12/19/16
I'm going to put the above task on hold, at least until I can implement this... 
I realized that a lot of the scanning time is being dedicated to finding the playtime of individual users. When we have 4k users, and then narrow that down to 1k, we still have to go back 1k times to get the playTime of each of those users. However— it isn't all that often that the rankings change, therefore it can be inferred that the majority of those 1k users are the same people. Therefore, we should only have to fetch those playTimes for those users once.

By storing all of their starting dates in a file, we would be able to calculate the difference in days (using datetime.timedelta) to find their playTime. Then, for the few users that we don't have a starting date recorded for, we can just fetch that using our current playTime function code.

One thing to think about possibly changing is that right now, I believe we include weekends in playtime. In reality, the market is only open on weekdays. For users that start on the same week and have their mmScore calculated on the following monday, the user that starts on Monday will have a better average than the one who started Friday, as those two days of padding will decrease the Friday-starter more.


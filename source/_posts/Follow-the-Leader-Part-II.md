---
title: Follow the Leader Part II
date: 2016-06-02 01:28:20
tags: ["python","web scraping"]
---
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
Make a function that creates a dictionary based on current holdings.

```python
def exampleFunc(inputUrl):
	# request and parse holdings page
	# build dictionary of holdings
	return {'NKE Buy': 581, 'SONC Buy': 943} # example output dictionary
```
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
`(1 / amountOfUsers )`  

This formula is used in the formula:  
`amountToBuy = (1 / amountOfUsers ) * ((myNetWorth/hisNetworth) * inputAmount))`  
However, this variable should be as a global when the program is first initiated, as I don’t want to read the file everytime I make a trade.
###RANKING
**REWORK**:
Alright, so it seems to be that what was done yesterday is working well now. However, there are a few other changes that need to be made if I am to get that “99.9%” efficiency that I was talking about earlier. One of those things is that I have to make some changes to the ranking algorithms so that the players, are of course, better. Here is a list of things that have to be done…

For players that have the same name, only keep the one that has the higher mmScore. This is to prevent people like Tom Riddle from taking 14 of the top 50 places in my ranking system
Remove duplicate names from the top 250 before cutting it down to 50.
There should be some stackoverflow answers on how to do this inside the pandas dataframe. Probably a combination of the list(), and set() functions would also work on the list of names. I may have to put it in an xrange() loop in order for it to remember the placement of the name among the other users in the ranking system.
As of now, there are players in the system that either:  

1. hardly make any trades or profit at all, or...
2. hardly ever have any holdings on their person.

This needs to be fixed probably in the 250 —> 50 merge, as I will not be able to look at the performance of 500 players in a short amount of time. This will most likely have to be a new function that is able to deduce a crap-ton of data from the performanceURL alone… And I thought I was almost done…
Most of the work in this section will be fairly simple boolean operands, in which I can see if the user had any holdings on a particular day, and their percent returns for that day. What I am mainly looking for is if they made any trades in a given week… I don’t think it really stores this data (at least not in the performance section), though, so I might have to just count up the amount of times in a given week on their transactions page.
Set up the system that finds the difference between the “top 50” list for two given days (today and tomorrow– because remember, we’re scanning at ~5:00PM), and returns a list of users that need to be added, and a list of users that need to have all of their holdings sold.
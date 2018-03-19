---
title: Follow the Leader - Part I
date: 2016-03-29 23:38:50
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
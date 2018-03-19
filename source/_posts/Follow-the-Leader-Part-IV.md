---
title: Follow the Leader Part IV
date: 2016-11-22 01:28:27
tags: ["python","web scraping"]
---
## 11/15/16
As the last time I worked on this was over a month ago, I can’t exactly say that I remember how far I got with converting playTime() to working with downloads, however it has come to my attention that regardless of how quickly these things download, the main problem that is stagnating progress for this application is that I have to carefully schedule times when I can debug it. This is because by the time I decide to work on this, I first have to wait 45 minutes for all of the player data to download (in order to rank them).

Therefore, it would be best if I could set up some sort of crontab on my server at home to rank the players ~30mins after the market closes. This way, I will have a players.csv file ready for me when I want to work on it. Just FYI, though, I might want to hold off actually setting up the crontab until I can set aside some time for me to actually work on this. I don’t want to have marketwatch blacklist my IP before I can work on this.

This should be a fairly simple feat (I really just need to come up with some sort of naming system that will save the scandate as the filename, so until then— I’ll make some more notes on how I could convert optimize playTime() / convert it to a download function.

It should be simple enough to cut the time down by making playTime() calculate after we cut it from 3900 players to 1000.

## 11/17/16
Alright, so I added some more stuff to clean out the duplicate users. All I need now is to iterate through the master dataFrame and remove the users that do not have the higher percentReturns… However, I do have one problem with doing this… A lot of this depends on ranking based off of the total mmScore for a user. However, if I am removing the lower duplicate users based just off of their totalPercentReturns alone, I could be missing out on some better players. Maybe they switched accounts to another game, and are starting to perform better on there… I wouldn’t be able to know until they overtake the performance of their other player… We shall see what happens, though. I still think it is necessary for me to make this change, as I don’t want that thing with Tom Riddle to happen again where he takes up 30% of the top 50 player rankings.

In order to remove these faulty rows, I just need to find the right way to delete rows in the pandas documentation… so that is the next step.


## 11/22/16
Alright so I’m now working on automating the execution of gamescan() so that I don’t have to do it manually every day. It is important that it has a fresh csv to work with. Here is the general process it should run every single day:

```python  
# Find today’s date and weekday.
today = str(datetime.datetime.now()).rsplit(" ",1)[0] # '2016-11-22'

# Keep going backwards in time until it finds the latest existing csv file.
pastDay = str(datetime.datetime.now()-datetime.timedelta(days=1)).rsplit(" ",1)[0]
daysAgo = 1
while os.path.isfile(pastDay+'.csv') == False:
	daysAgo+=1
	pastDay = str(datetime.datetime.now()-datetime.timedelta(days=daysAgo)).rsplit(" ",1)[0]
```

If one already exists for today, exit.
If today is monday, it should still check for ones made on the weekends— however:
if it is creating one, it should check to see if one was already made Friday… nothing has changed since then. If one wasn’t made on Friday, make one for today.
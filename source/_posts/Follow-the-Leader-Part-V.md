---
title: Follow the Leader Part V
date: 2016-12-19 01:28:30
tags: ["python","web scraping"]
---

I'm going to put the above task on hold, at least until I can implement this... 
I realized that a lot of the scanning time is being dedicated to finding the playtime of individual users. When we have 4k users, and then narrow that down to 1k, we still have to go back 1k times to get the playTime of each of those users. However— it isn't all that often that the rankings change, therefore it can be inferred that the majority of those 1k users are the same people. Therefore, we should only have to fetch those playTimes for those users once.

By storing all of their starting dates in a file, we would be able to calculate the difference in days (using datetime.timedelta) to find their playTime. Then, for the few users that we don't have a starting date recorded for, we can just fetch that using our current playTime function code.

One thing to think about possibly changing is that right now, I believe we include weekends in playtime. In reality, the market is only open on weekdays. For users that start on the same week and have their mmScore calculated on the following monday, the user that starts on Monday will have a better average than the one who started Friday, as those two days of padding will decrease the Friday-starter more.
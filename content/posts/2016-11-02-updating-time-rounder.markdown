---
title:  "Updating Time Rounder"
date:   2016-10-21 18:05:00 -0500
draft: false
---
It has been awhile since I have touched the time_rounder gem. When I left off, I
only implemented the 15 minute schedule. I have recently started work on that
gem again in an effort to commit more to open source and build up my portfolio.
Here is what I have been up to with the gem.

First, I have improved the rounding schedule setup. When I started the gem I
just simple used large hashes containing every minute of an hour and what it
rounded to. I since found a happy medium in using Array' min_by and a small
amount of math. I may make further improvements to the code in future but I feel
it is pretty good at the moment.

Next, I have improved the tests and made them easier to understand and not have
so many examples by lumping common examples together. I essentially take all the
minutes that round to the same number and test them in a loop, instead of
repeating a test for each minute of the hour.

Lastly, I am working on the other rounding schedules. At the moment there is
only the 15 minute schedule. The plan is to add 1 hour, 30 minute, 20 minute,
10 minute. and 5 minute schedule. Once all the schedules are complete the gem
will move to a 1.0 release.

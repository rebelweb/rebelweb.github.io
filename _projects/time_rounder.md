---
layout: project
title: "Time Rounder"
---

I have an open source Rubygem I have been hacking on to round time to the
nearest x number of minutes. This gem started out in need for a project at my
job. A few year ago we were trying to use math to round time (complex) and
implement different rounding schemes (5, 10, 15 minutes) so it grew more
complex as we went, and occasionally we would find problems with the rounding
math.

I set out to find a better way, a more reliable way to round time, so I started
the __time_rounder__ gem. It uses schedules to round time, rather than using
math that is hard to understand, especially for new developers. Overall we
solved our issues using the schedules method rather than the math.

Time Rounder is available on [github](https://github.com/rebelweb/time_rounder).

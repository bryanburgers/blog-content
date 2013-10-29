It's 2:30 am. Somewhere in San Francisco, California, a woman is in a
coffee-fueled programming bender. For the past five hours, she's been immersed
in her program, unstoppably producing code. She's in the zone. In her head,
she sees the whole picture; how every bit of code relates, how one small
change ripples through the system, how what she wrote three hours ago is
exactly what is needed at this moment. She's on fire.

It's 6:30 am. Somewhere in Sioux Falls, South Dakota, a man has just finished
getting ready for work; he sits down by his laptop and fires up his IDE.
While waiting for it to launch, he glances at the clock and hopes,
delusively, that he can get fifteen good minutes of programming in before his
1-year-old daughter starts screaming from her crib. He looks at his code: what
part of the program was he even working on again?

I am the second programmer.

This is where [Test-Driven
Development](https://en.wikipedia.org/wiki/Test-driven_development) (TDD)
comes to the rescue. Where I used to be able to hold the entire state of my
programs in my head at any time, now it's just not possible to refresh myself
enough to really remember the intricate details of every part of the
application. Where I used to have what seemed like an innate knowledge of how
any change would affect the program as a whole, now I need a way to know that
what I just changed didn't break anything. So having full test coverage on my
code is exactly what I need: in the fifteen minutes I have to program, I crank
out some tests and a feature, and with a simple test run, I can see if today's
session broke code from a 15-minute session three weeks ago.

Even more than that, I often don't even remember where I left off on the last
function I wrote. On one day, I can have an idea of all of the things a
function needs to do and the cases it needs to handle, but run out of time to
write code for all of those cases. The next day, I may assume that I finished
the previous day's code, leaving cases unhandled and bugs in the code. If
instead, on the first day, I write tests to cover all of the cases before
writing the function, then on the next day I'll have failing tests reminding
me that the function is not complete; TDD has done its job.

These days, it's just not feasable for me to go on hours-long programming
benders anymore. But what I've lost in caffeine-fueled trances of massive
productivity, I've gained back in the slow but confident progress that is made
possible because of test-driven development. So while TDD has its pros and
cons, for a project that has already spanned nearly a year of small,
incremental development, I'm sold. Hold the caffeine; I'll take full test
coverage, please.

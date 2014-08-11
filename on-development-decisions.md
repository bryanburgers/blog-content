Black and white. Good and bad. The right way and the wrong way. It would be
great if web development fell into nice neat little categories. It doesn't.

![Gradient from black to white](images/on-development-decisions/bw.png "Lots of gray areas.")

If you do web development for an agency, the decisions you need to make are
pushed and pulled by various different parties. The users who visit the
website have certain needs. The company that owns the website has different
needs. The agency that you work for has its own needs. All of these needs have
to be balanced delicately and tactfully. All tug and pull at every choice you
make on a website.

Do it the right way. It's a nice thought, that we can just declare universally
the one true way to do it. But that ignores the forces at work. It ignores the
delicate balance of real humans that make some decisions hard.

Recently I tried switching from icon fonts to SVG sprites because they are
[better][svg1] for the [user][svg2]. For our workflow, it was a maintenance
nightmare. I switched back. The choice I had to make was worse for some users,
but considerably better for my agency and my team.

On the other hand, supporting IE8 is a hard, time-consuming, near-futile task
that involves lots of debugging with bad tools, screaming at the computer, and
hacks. But with more than 10% of the market on some of our sites, we choose to
support IE8. It's not a popular decision, but it's best for the users even
though dropping IE8 support would be a huge win for our agency.

Not too long ago, I suggested a change during website planning that would be
better for the users. It didn't add cost. It didn't add time. It was a win for
everybody. But the company wanted something different – something worse for
the users. So I backed my position with logic and facts. But still, the
company wanted it their way. What could I do? I had to put the relationship
between the company and my agency ahead of the users.

On another site I worked on, the company budgeted extra time to make sure
their site is accessible. [An accessible site is good for all users.][acc]
They put the users needs before their own. Hopefully that's a choice that
benefits them in the long run.

Sometimes, there isn't enough budget on a project to have a perfect site and
get it done on time. Time for hard decisions. For every component, do I put
the user first and do this well, or the agency first and get it done?
Often, it's a mix of both.

The right way and the wrong way. Black and white. It's a nice idea. But what's
white for one may be black for another. So we mix. We blend. We compromise. We
try to find the right shade of gray. And at the end of the day, we just hope
we made some of the right decisions.

[svg1]: http://ianfeather.co.uk/ten-reasons-we-switched-from-an-icon-font-to-svg/
[svg2]: http://css-tricks.com/icon-fonts-vs-svg/
[acc]: http://digitaldesignstandards.com/standard/accessibility/designing-accessibility-simply-good-business/

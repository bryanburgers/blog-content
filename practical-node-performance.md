Node is [getting faster][pr] thanks to continual improvements to V8, the
JavaScript engine that powers both Node and Google Chrome. Yeah, sure,
whatever, that's great. But what, practically, should a typical node dev that
works on typical node apps do with this information?

This question kept coming up as I read [Get Ready A New V8 is
Coming][microperf] which is all about micro-optimizations in code style.

## Start off writing clear code

`for-in` vs `Object.keys`, `delete o.prop` vs `o.prop = undefined`, classes vs
object literals. These are all micro-optimizations that, in a typical
application, you don't need to care about.

JavaScript is fast, even if you don't micro-optimize. You're better off writing
the clearest code you can, code that is easiest to maintain and reason about,
and only worrying about micro-benchmarks when you really need them.


## Determine if performance is your blocker

Making your application fast is a user experience concern. There's plenty of
research into that. It might even be one of the killer features that brought
you to Node.

The typical web app written in Node is likely fast enough already (or if it's
not, you need to be looking somewhere other than micro-otimizations).

I load tested a handful of Node applications recently looking for performance
and the number of connections a single server could handle. It turns out the
connection limit was correlated to memory usage, not CPU performance. And for
the typical node web app or API endpoint, I'm willing to guess it's the same.


## Spend time building confidence in upgrades

Performance is important, but spending time doing lots of benchmarks and
micro-optimizations on a memory-bound application isn't yielding high returns.
Where do we spend our time?

The practical thing is to spend our time investing in ways to build high
confidence that the application is working correctly. Unit tests, integration
tests, deployment methods, canary deploys, whatever – these will pay for
themselves when it comes to maintenance.

Once these are in place – once a team has high confidence that they can know
whether any given change affects whether the application is functioning
correctly – then a Node upgrade is just another change that can be tested.

At that point, upgrading to the newest version of Node becomes easy, and the
application gets faster because all of the hundreds of man-years poured into V8
and Node to make them fast, without ever having to write
fast-but-unmaintainable code or spend time micro-benchmarking.

And the best part is that it is a compounding win. As each version of Node gets
faster, the application continues to reap the reward of being easily
upgradeable. That sounds like time well spent.

[microperf]: https://medium.com/the-node-js-collection/get-ready-a-new-v8-is-coming-node-js-performance-is-changing-46a63d6da4de
[pr]: https://github.com/nodejs/node/pull/14574

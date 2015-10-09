The default [user agent header][ua] set by NSURLRequest is awful.

We all accept that [user agents lie][mstrutt]. But that's in the browser world.

In API world, especially when you control both the app and the API, user agent
headers can be incredibly useful. They can be used in web logs for analytics.
They can help diagnose when certain versions of the app are sending or asking
for bad data. They, nominally, identify what agent is acting on behalf of the
user.

But they can only do this if the user agent header isn't awful.

---

For an app I'm working on, the user agent being sent to my API was `MyApp/17
CFNetwork/758.0.2 Darwin/15.0.0`.

Let's break that down.

`MyApp/17`. Alright, that's useful. It's the name of my application and the
build number.

`CFNetwork/758.0.2`. This is completely worthless to me. I looked up CFNetwork,
and it's the networking API in Core Framework. It makes sense, I guess, but the
version number means nothing to me.

`Darwin/15.0.0`. [Darwin][darwin] is the UNIX underpinning of both OSX and iOS.
So again, it makes sense to be there – I guess – but it isn't providing me any
actionable information.

So of the three tokens, only the first one is useful.

---

Compare that default to this: `MyApp/1.3 iOS/9.0.1 (iPhone5,2)`.

`MyApp/1.3`. The name and release version of the app. Instantly useful. And
having the release version instead of the build number is way better.

`iOS/9.0.2`. Oh good, the app is running on the latest version of iOS. (As of
this writing.)

`(iPhone5,2)`. This is Apple's [model identifier][models]. It's not quite as
nice as "iPhone 5" or "iPhone 6S+", but if you look at these enough, you can
pretty quickly figure out which is which.

This user agent is *way* more useful than the default.

---

Thankfully, we can change it. We can set up NSURLRequest to use a better user
agent header. I cobbled together [the code][gist] to do it, using a bit of
[DeviceGuru's code][deviceguru] to get the model identifier, and...

```
request.addValue(
  UserAgent.getUserAgent(),
  forHTTPHeaderField: "User-Agent")
```

*Voila!* A better user agent header.

Do yourself a favor. Change your app's user agent.

[ua]: https://en.wikipedia.org/wiki/User_agent
[mstrutt]: http://mstrutt.co.uk/blog/2012/06/user-agents-the-good-the-bad-and-the-ugly/
[models]: https://www.theiphonewiki.com/wiki/Models#iPhone
[darwin]: https://en.wikipedia.org/wiki/Darwin_(operating_system)
[gist]: https://gist.github.com/bryanburgers/daa09d5d8c3b61c6d98a
[deviceguru]: https://github.com/InderKumarRathore/DeviceGuru/blob/0a2b8c1e5f4e7df3e20438b762ac091b0206262c/DeviceGuru.swift#L77

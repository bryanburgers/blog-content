I recently moved a handful of my personal projects from [Heroku][heroku] to
[Digital Ocean][doref]. Partly I did this because the terms of Heroku's free
tier changed, but mostly because I've wanted more control over the server and
experience managing a server for a while.

The projects I moved were all written in [node][node] and I needed a way to
start them up when the server started, and keep them up if they ever died. And
hey, since Upstart handles log rotation too, all the better.

I can't be the only person to want to run a node app with Upstart, so I
started looking around. What I found was a bunch of Upstart configurations
that used scripts instead of the native features of Upstart. I thought there
had to be a better way.

So I dug into the docs. Here's what I came up with.

```
#!upstart

# 1.
description "burgers.io node application"
author "Bryan Burgers <bryan@burgers.io>"

# 2.
env GITHUBREPO=bryanburgers/blog-content
env GITHUBBRANCH=content
env PORT=10100
env UACODE=UA-44110500-1
env UAHOST=burgers.io
env ORIGIN=https://burgers.io
env HOME=/var/apps/burgers.io/current

# 3.
chdir /var/apps/burgers.io/current

# 4.
setuid burgersio

# 5.
start on started networking
stop on stopping networking
# 6.
respawn
respawn limit 10 60

# 7.
exec /usr/bin/node /var/apps/burgers.io/current/index.js
```

1. burgers.io is a node app, in case you didn't know. I didn't let anybody
   else write it, either.

2. If you're familiar with deploying to Heroku or have read
   [12factor.net/config][12factor], this won't come as a surprise. All
   configuration for the site is set in environment variables. So, we need to
   set those values in the upstart config.

3. Most node apps expect the current working directory to be the root of the
   application. Let's oblige.

4. Principle of Least Privilege here. Run the app as the user named burgersio
   rather root.

5. Don't start the application until the network services have loaded. Let me
   admit what I don't know: I don't know what I should be waiting for here.
   But this seems to work.

6. Restart if the node app ever dies. Which node apps do from time to time.
   But if it does more than 10 times in 60 seconds, just cool it for a bit.

7. That's it, run it!

There's nothing crazy here. Upstart has more than enough features to
successfully run a node app.

[heroku]: https://heroku.com
[doref]: https://www.digitalocean.com/?refcode=5842be4f2de8
[node]: https://nodejs.org/
[12factor]: http://12factor.net/config

I have always liked the three-tier architecture. In my .NET days, I frequently
used .NET Web Services, and then later WCF Services to structure large
applications. When I started [Score Lab](http://www.scorelab.co), I started
out using a two-tier architecture, but quickly switched to a three-tier
architecture.

## What's this gibberish?

A [three-tier
architecture](https://en.wikipedia.org/wiki/Multitier_architecture) is a way
of separating out the components of a software application into, you guessed
it, three different tiers. This usually includes a database tier, an
application tier, and a tier for presentation to end users. I should point
out, though, that in web development, I do not consider the browser a tier.

## Score Lab

When I started working on Score Lab, I started with a two-tier architecture.
So, the client – a browser — would make a request to the web server, the web
server would query the database server and perform both the business logic and
the presentation logic, and then it would return a response to the client.

I quickly decided, for various reasons, that I would really like to separate
out the presentation logic from the business logic. And since I already had
the makings of an API, I decided to migrate to a three-tier architecture. So
now, I have two web-servers, the front-end server and the API server. When the
client – again, a browser – makes a request to the front-end server, the
front-end server needs to get data for it to return. Instead of it performing
database queries and doing all of the business logic, it instead sends a
request to the API server for the exact data it needs. The API server, in
turn, performs the database queries and the business logic, then responds with
all of the relevant data in a JSON response. The front-end server now has all
of the data it needs, and responds with a web page for the client.

Originally, I was using one [Heroku](https://www.heroku.com) application for
the web server. When I switched over to a three-tier architecure, I ended up
with two different Heroku applications running.

## Wait, why?

There are a few reasons why I spent development time making this conversion.
Making the switch cuts one large project into two smaller, independent
projects. Each of the projects indepently is smaller, more understandable,
easier to reason about, and easier to test than a single monolithic project.
Having separate concerns for each project lets me focus on what each piece
needs to do and not mix presentation with logic.

Having two projects also gives Score Lab more flexibility. For example, if I
found it necessary, I could move just the front-end server from Heroku to a
dedicated host, Amazom Web Services, or some other platform, and the API
server would not be affected. Or, as another example, I could decide later to
completely rewrite the API server using a different framework or a different
language, and the front-end server wouldn't need to change at all.

## The gist

Switching from a two-tier to a three-tier architecture made sense for Score
Lab. I ended up with a more understandable product, and I have more
flexibility going into the future. A win in my book.

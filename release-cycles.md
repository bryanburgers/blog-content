I work on the internet, where I mostly build and maintain websites. The web
has this really interesting property that people often don't even think about:
on the web, when I make a change, people can see it _instantly_.

![Sculpture of a tortise and a hare](images/release-cycles/sculpture.jpg "The tortise and the hare. (c) flickr.com/cle0patra")

But recently, I worked on an iPhone app where this isn't the case. And it got
me thinking about release cycles and their impact.

## A change on the web

On the web, changes are immediate. I make a change, and it shows up.

If there's a bug, I fix it, and there's no more bug. If there's an awesome
feature that needs to be added, I add it, and people can get it right away.

## A change on the iPhone

For iPhone apps, changes are not so immediate. I make a change; then I have to
submit the app to the app store for re-review. That takes a while. And after
that, ultimately, the choice to upgrade an app is in the hands of the user.
iOS7 introduced auto-update functionality, but it isn't required. A user could
have a version of an application that is 3 or 4 versions old. Or even the
first version ever released.

So if there's a bug, I fix it, and it takes between 7 days and never to get to
the user. Same thing for features.

But on the plus side, a user doesn't have to redownload the application for
every use. Once a version is downloaded, it's downloaded, it's there. Things
load immediately. No waiting for the network. No dreading a 2G connection. The
application is already downloaded and ready to go.

## The aggregate

So the web is instantaneous, and an iPhone app has a longer release cycle. Now
this is where it gets interesting. As I've said before, [an app isn't just
what's on the phone][iceberg], it's so much bigger than that. Most apps have
both an iPhone component and a backend API component. Guess where that API
component is? That's right, it's on the web.

So we have an app where we can update one component as frequently as we want
but the other component must move through a long release cycle.

## Structuring the system

How then do we structure the system? Where do we put the logic? If we're
aiming for maximum flexibility, we can try to push as much of the logic to the
API, the component with the fastest release cycle. That lets us adapt quickly.

But logic that's on the iPhone happens almost instanteously. Logic that's on
the API side requires a slower network request. So for a fast, smooth
experience that people expect from an app, we need to keep some of the logic
on the phone. Logic on the iPhone also shields us from network connection
issues.

Ultimately, I settled on the middle ground. I didn't go full on data-driven
and use the API to provide everything, because I was scared of connectivity
issues. But for the logic that made sense to be in the API, I left it in the
API, which gives me flexibility.

And ultimately, if there's a bug that needs to be fixed, the Apple approval
process is only about a week. At least I don't have to mail out CDs like AOL
did.

[iceberg]: /tip-of-the-iceberg

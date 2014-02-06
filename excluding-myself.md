With as many visits as my site gets – and by that, I mean very, very few,
actually – any visit that I make to the site can really skew the analytics.

On the one hand, I'm my best visitor: I've been to every page on the site. But
on the other hand, *look at all of these visits from Chrome!* doesn't mean
very much if I think that most of them are me.

Basically, I need a way to exclude myself from Analytics.

![A person excluded from a larger group](/images/excluding-myself.svg "That's me, over there. Being excluded. Person by Jens Tärning from The Noun Project.")

After a bit of searching, I settled on an approach that involved creating a
[custom dimension][1] in Google Analytics, and figuring out how to set its
value only when I visit my own site. Once I had that data, I could properly
filter based on that custom dimension.

A [custom variable][2] would also be an option, but when I signed up for
Google Analytics, I apparently signed up for [Universal Analytics][3], so a
custom dimension it is.

## Step 1: A custom dimension

First, I needed to create the custom dimension. From Analytics' admin panel,
under my Property, there is a Custom Definition option. And under that, a
Custom Dimensions screen. So, I hit "New Custom Dimension". And I created one.

*[Editor's note: That might have been the most boring paragraph ever.]*

Every custom dimension has a scope. There are three options. "Hit" only
applies the custom dimension to the current page view. "Session" applies the
custom dimension to a user's entire visit. "User" applies the custom dimension
to a user across visits. Since I *always* want to exclude myself, I set this
to "User".

## Step 2: Operation batcave

The custom dimension only matters if it's actually tracking me. So I needed to
figure out a way to track myself without inadvertantly tagging anybody else as
myself. Because that would be weird. I'm only one person.

It'd be great if there was a page that only I went to. If my site was run by a
CMS, then I'd just put the code on some logged-in page.

But it's not.

So I created a super-secret page that only I know about. Like a batcave. I
always wanted a batcave.

And if a visitor goes to that super-secret batcave page, I'll know, for sure,
that visitor is me.

## Step 3: Script it

I now have a super-secret page that only I will ever go to. Now comes the easy
part. I just had to update the analytics code on that page.

```js
ga('create', 'UA-12345678-1', 'burgers.io');
// This user is me!
ga('set', 'dimension1', 'me');
ga('send', 'pageview');
```

That's it. I'm all set up.

## Step 4: Segment

Now, with every visit, Google Analytics knows whether the visitor is me or
not.

Good for Google Analytics.

But *I* want to know whether each visitor is me or not.

There's probably a better way to do this – I'm no Google Analytics expert –
but I just created a new segment in my usual profile. On the little segment
dropdown, I used that fancy "Create New Segment" button.

## Self: excluded

The goal was to exclude myself out of analytics so that I could see a more
true represenation of my visitors.

By creating a custom dimension in Google Analytics, I was able to do this. And
so far, I'm quite happy with this system.

And now I can track you all the better. But not in a creepy way – I promise.



[1]: https://developers.google.com/analytics/devguides/platform/customdimsmets
[2]: https://developers.google.com/analytics/devguides/collection/gajs/gaTrackingCustomVariables
[3]: https://support.google.com/analytics/answer/2790010?hl=en

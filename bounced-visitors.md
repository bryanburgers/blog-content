I recently created a landing page for a [Score Lab](http://www.scorelab.co)
PPC campaign where potential customers either convert or bounce.

If a customer converts, *horray!*, I throw a little party in my head.

But most potential customers, unsurprisingly, don't convert. And while I wish
all of the visitors would convert, it's not a terrible thing for them to
bounce. All of the information they need is on that one page, so there should
be no need to go to any other page. Which means, like I said, that if they
don't convert, they bounce.

My problem is that Google Analytics gives me no useful information when one of
these visitors bounces. Google Analytics counts the time on site for these
customers as 0 seconds. I have no idea how much of the landing page they
looked at. Or whether they scanned or read. More information would help me
optimize the page better for a better conversion rate, but I have almost no
information.

So, I decided to change that.

What I do have on my landing page is several `<h2>`s. If I had a way to know
which of these `<h2>`s a visitor has seen and how long they spent on the site,
I might have a better idea of what the visitor has read before bouncing. That
could be valuable information to help me identify problem areas.

The plan I came up with was to track when an `<h2>` was seen using Google
Analytics' Events. Any time the visitor scrolled a headline into view, a
Google Analytics event would be sent.

Roughly:

```js
// When the users scrolls a headline into the visible range
var headline = // Determine which headline is now visible
ga('send', 'event', 'Scroll', 'Header', headline);
```

To make life even easier, a Javascript library –
[Sloth](https://github.com/hakubo/Sloth) – already exists to let me know when
an element is scrolled into view. So determining when an `<h2>` is scrolled
into view becomes incredibly simple.

```js
sloth({
  on: document.getElementsByTagName('h2'),
  threshold: 0,
  callback: function (element) {
    var headline = element.textContent || element.innerText;
    if (headline && ga) {
      ga('send', 'event', 'Scroll', 'Header', headline);
    }
  }
});
```

**Boom**. With a few simple lines of code, I can now see how much time
potential customers spent on the landing page, and approximately how far down
in the page they read or scanned.

Now, to optimize the page and get that conversion rate up.

"That's a lot of markup for an image." The primary concern I've heard about
responsive images since I wrote my [responsive images][self] article is that
they're just too verbose.

Things just got better.

<figure>
    <img src="images/communication/phones.jpg"
         alt="Old telephones at the Museum of Communication"
         itemprop="image"
         sizes="(min-width: 43.5em) 43.5em, 100%"
	 srcset="http://placehold.it/348x174/ccc/4488ff 348w,
        	 http://placehold.it/436x218/ccc/4488ff 436w,
        	 http://placehold.it/522x261/ccc/4488ff 522w,
        	 http://placehold.it/610x305/ccc/4488ff 610w,
        	 http://placehold.it/696x348/ccc/4488ff 696w,
        	 http://placehold.it/870x435/ccc/4488ff 870w,
        	 http://placehold.it/1044x522/ccc/4488ff 1044w,
        	 http://placehold.it/1218x609/ccc/4488ff 1218w,
        	 http://placehold.it/1392x696/ccc/4488ff 1392w,
        	 http://placehold.it/1566x783/ccc/4488ff 1566w">
    <figcaption>Responsive Images. Resize me and WEEEE!</figcaption>
</figure>
<script src="/assets/libs/picturefill-2.0.0-alpha.js" async></script>

Saving users untold kilobytes of data and speeding up their experience should
be worth the cruft. But now there's an even better option: [srcset +
sizes][srcsetandsizes].

`<picture>`, the element soon to be in many browsers, solves [a lot of use
cases][usecases]. One of the big ones is "art direction", which involves, for
example, choosing one image for small screens, a different image for medium
screens, and a third image for large screens.

While that's a good use case, and I'm glad it's being considered, art
direction is rarely an option for the sites that I'm involved with. Instead, I
focus on responsive images primarily for the bandwidth savings.

And for that use case, where every image is the same except for its size, the
`<picture>` spec now lets us get that without all of the verbosity of the
picture element.

Like the original [responsive images][self] article, I'll show you how in two
steps.

## The script

The first step: drop an extra script into the footer of the site.

```
<script src="/assets/libs/picturefill/picturefill.js" async></script>
```

I use [picturefill][picturefill] because it properly polyfills this. But soon,
browsers will natively support this, and the script won't be necessary.

## The markup

Here's the original.

```
<img src="http://placehold.it/1900x696" alt="Griffins. A pryde of them.">
```

Here's the verbose markup, using the `span`-based pattern of picturefill 1.0.

```
<span data-picture data-alt="Griffins. A pryde of them.">
    <span data-src="http://placehold.it/475x174"></span>
    <span data-src="http://placehold.it/950x345"  data-media="(min-width: 476px)"></span>
    <span data-src="http://placehold.it/1425x522" data-media="(min-width: 951px)"></span>
    <span data-src="http://placehold.it/1900x696" data-media="(min-width: 1426px)"></span>

    <!--[if (lt IE 9) & (!IEMobile)]>
    <span data-src="http://placehold.it/1425x522"></span>
    <![endif]-->

    <!-- Fallback content for non-JS browsers. Same img src as the initial, unqualified source element. -->
    <noscript>
        <img src="http://placehold.it/475x174" alt="Griffins. A pryde of them.">
    </noscript>
</span>
```

And here's the much more succinct markup, using `srcset` and `sizes`,
supported in picturefill 2.0, which as of this writing is still in beta. The
browser (or the polyfill) will figure out which image to download based on the
information it has available.

```
<img src="http://placehold.it/475x174" alt="Griffins. A pryde of them."
     sizes="100%"
     srcset="http://placehold.it/475x174 475w,
             http://placehold.it/950x345 950w,
             http://placehold.it/1425x522 1425w,
             http://placehold.it/1900x696 1900w">
```

Now you have no excuse.

[self]: /responsive-images-using-picturefill
[srcsetandsizes]: http://ericportis.com/posts/2014/srcset-sizes/
[usecases]: http://usecases.responsiveimages.org/
[picturefill]: https://github.com/scottjehl/picturefill

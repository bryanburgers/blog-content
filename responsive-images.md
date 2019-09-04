For a recent project I worked on, the mock-up for the home page carousel's
images were *huge*. 1900×696 pixels huge. Personally, I've never used
responsive images before (long overdue), but I decided the savings on this
project could be considerable, so now would be a good time to try.

It ended up working really well, and was not difficult at all.

I ended up using [picturefill][picturefill]. It was a two step process.

## The script

The first step: drop an extra script into the footer of the site.

```
<script src="/assets/libs/picturefill/picturefill.js"></script>
```

I didn’t even need to show you that, but I did.

## The markup

The second step: use the correct markup. Instead of the img tag:

```
<img src="https://placehold.it/1900x696" alt="Griffins. A pryde of them.">
```

I replaced it with the following.

```
<span data-picture data-alt="Griffins. A pryde of them.">
    <span data-src="https://placehold.it/475x174"></span>
    <span data-src="https://placehold.it/950x345"  data-media="(min-width: 476px)"></span>
    <span data-src="https://placehold.it/1425x522" data-media="(min-width: 951px)"></span>
    <span data-src="https://placehold.it/1900x696" data-media="(min-width: 1426px)"></span>

    <!--[if (lt IE 9) & (!IEMobile)]>
    <span data-src="https://placehold.it/1425x522"></span>
    <![endif]-->

    <!-- Fallback content for non-JS browsers. Same img src as the initial, unqualified source element. -->
    <noscript>
        <img src="https://placehold.it/475x174" alt="Griffins. A pryde of them.">
    </noscript>
</span>
```

OK, OK, so it does look like quite a bit more markup, yes.

So that picks one of the four images. The last one that matches. So at the
smallest size, it only downloads a 475×174 pixel image. Compare that to the
1900×696 image that would have downloaded without it.

I went with four images because it seemed like a good balance between good
breakpoints and good images.

Also, the media queries do not line up with my media queries in the CSS. *They
don’t need to*. I’m just telling the browser to use an image until the screen
is larger than image, then jump up to the next biggest image. Basically, use
the smallest image that is bigger than the screen.

Picturefill does end up inserting an `<img>` in there to actually show the
image, and because the spans are pretty much negligible for the CSS, I didn’t
even have to change my CSS that was selecting for img.

I’m quite happy with how easy it was (seriously, a lot of markup, but not
difficult), and I think it will be a big win for the people that come to the
site.

[picturefill]: https://github.com/scottjehl/picturefill

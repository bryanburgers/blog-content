I've long been a huge admirer of [SVG][svg] (W3C's Scalable Vector Graphics).
As retina screens proliferate, SVG allows web developers to create images that
look great on all screens, regardless of the resolution.

That's where most of my SVG use has been so far: using `<img>` tags and CSS
`background-image` to achieve resolution independence.

However, SVG has other great features that make it worthwhile, and on a recent
project, I had to use SVG for something entirely different: I had to use
inline SVG to allow users to select and manipulate irregularly shaped objects.

Much to my dismay, [jQuery][jq] 1.10 does not have great support for SVG.

## Creating elements

jQuery has trouble creating SVG elements.

To work around this, I needed to drop from jQuery to native DOM APIs in order
to create elements. So, instead of

```js
// This doesn't work.
var root = $('<svg>')
	.attr('viewBox', '0 0 100 100');
	.appendTo($('body'));
```

I could use

```js
// Instead...
var svgNS = 'http://www.w3.org/2000/svg';
var root = document.createElementNS(svgNS, 'svg');
root.setAttribute('viewBox', '0 0 100 100');
$('body').append(root);
```

## Classes and attributes

jQuery also did not deal with classes and attributes correctly on SVG
elements. So, instead of

```js
// This doesn't work.
$('#my-shape').removeClass('inactive').addClass('active');
```

I tried

```js
// Instead...
$('#my-shape').get(0).setAttribute('class', 'active');
```

Now, the problem with the above code is that I'm replacing *all* of the
classes on my shape with a single "active" class. That means my shape will
lose other classes that are unrelated to its state. And without jQuery, adding
and removing classes is [hard][stackoverflow]. Maybe I could have used
[element.classList][classlist], but that [isn't supported in IE9][caniuse].

But in my case, I needed to keep track of the SVG element's state, so I opted
to use a new attribute instead. Since my shapes only have one state at any
given time (a shape is either inactive or active), *setting* the state is
exactly what I wanted, instead of adding or removing a class that represents
the state.

```js
// Or even this
$('#my-shape').get(0).setAttribute('data-state', 'active');
```

## Events

jQuery does deserve some credit. Events worked exactly as expected. Even event
delegation.

```js
$('html').on('click', 'path', function(e) {
	if (this.getAttribute('data-state') === 'active') {
		this.setAttribute('data-state', 'inactive');
	}
	else {
		this.setAttribute('data-state', 'active');
	}
});
```

## CSS

And of course, it was the CSS that pulled this all together. Active shapes
have one color, inactive shapes another color. This doesn't have anything to
do with jQuery, but browsers seemed to handle this prefectly.

```css
path {
	fill: red;
}
path[data-state=active] {
	fill: green;
}
```

## Summary

jQuery's support for manipulating inline SVG still has some rough edges. But
thankfully, all of its rough edges are not too difficult to deal with.

[svg]: https://en.wikipedia.org/wiki/Scalable_Vector_Graphics
[jq]: http://jquery.com/
[stackoverflow]: http://stackoverflow.com/a/196038/314971
[classlist]: https://developer.mozilla.org/en-US/docs/Web/API/Element.classList
[caniuse]: http://caniuse.com/#feat=classlist

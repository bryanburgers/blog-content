Recently, I started getting away from the jQuery pattern of checking the
length of an object before writing code. Instead, I've found myself wrapping
my selector in jQuery's `each` method, and writing my code inside the
function.

For example, instead of this:

```js
if ($('.target').length) {
	var target = $('.target');

	// Do something with target and its children
	target.find('.child').doSomething();
	// etc.
}
```

I've been writing this:

```js
$('.target').each(function() {
	var target = $(this);

	// Do something with target and its children
	target.find('.child').doSomething();
	// etc.
});
```

This has several advantages.

1. **Variables get scoped correctly**: Javascript does not have block-scoping,
which means that variables declared in an `if` statement are actually retained
outside of the if statement. Using a function with `each` means variables
declared in the function stay in the function.

2. **Multiple elements handled properly**: Even if I *think* that there's only
one element with class `.target`, I'm protected from failing code if there are
more. Multiple `.target`s will work just as well as a single one in this case,
even to the point that each gets its own set of locally scoped variables.

3. **Handles the no-element case**: In the first code snippet, I explicitely
test to see if the element exists before running the code. This is a good
thing because a script file is often loaded on every page, but `.target` may
only be on a few of those pages. The second code snippet also handles the
situation that `.target` does not exist on a given page just as well.

While it may be odd to see an `each` block when only expecting a single item,
the `each`-scoping method works as well as or better than the `if`-`length`
method, and is just as short to write. It's a win in my book.

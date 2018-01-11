In [Wrapped Iterators in Rust][wrappediterators], I played around with creating
an [Iterator][iterators] struct like Rust's native [`Map<I>`][rustmap],
[`Enumerate<I>`][rustenumerate], [`Filter<I>`][rustfilter], etc. that wraps an
iterator to create a new iterator.

I'm pretty happy with how it turned out. But there's one thing I didn't like
about it.

When dealing with iterables in Rust, they can all be chained nicely together.
For example,

```
some_iter
    .map(...)
    .filter(...)
    .enumerate()
    .collect()
```

The `CircularEnumerate<I>` class I was playing with is in every way a peer to
Rust's classes, except that it can't be chained. So the nice-looking code above
would end up looking more like the following.

```
let temp_iter = some_iter
    .map(...)
    .filter(...);

let circ = CircularEnumerate::new(temp_iter, 3);

circ.collect()

// Or even more confusingly
CircularEnumerate::new(some_iter.map(...).filter(...), 3).collect()
```

To feel like it fits in, using a CircularEnumerate really should fit nicely
into the ecosystem, like the following.

```
some_iter
    .map(...)
    .filter(...)
    .circular_enumerate(3)
    .collect()
```

But how can I add this? `map`, `filter`, and `collect` are all functions on the
Iterator trait, and I can't hack the standard library, right? *Right*? So what
can I do?

I searched the internet and the closest thing I could find was a blog post
called [Extension traits in Rust][extensiontraits]. This seemed pretty close to
what I wanted, but being the Rust newbie that I am, I couldn't figure out
exactly what was happening. And I couldn't get the right search terms to find
Rust documentation on "Extension Traits" on the official Rust docs site.

I started to dig in and try it. Learn by failing. Follow the compiler hints
wherever they took me. After getting most of the way there, I looked at the
example code in the extension traits article again and it made more sense.

Here's how I understand what's happening. I can't hack the standard library.
Bummer, right? There is no way for me to add `.circular_enumerate` to
the Iterator trait.

I don't need to, either.

What I need is to create a new trait. Let's pick a terrible name and call it
`CircularlyEnumerable`, for any type that can be, um, circularly enumerated?
Any type that is circularly enumerable can call `circular_enumate` and get a
`CircularEnumerate` instance back.

***blinks*** What?

OK, let's start with a rough trait.

```
trait CircularlyEnumerable {
  fn circular_enumerate(self, items: u32) -> CircularEnumerate<Self>;
}
```

So any type that implements CircularlyEnumerable can call `circular_enumerate`
and get an instance of the CircularEnumerate wrapped iterator.

If there's a `impl CircularlyEnumerable for Map<I>`, then we're good for the
result of `map`: `some_iter.map(...).circular_enumerate(3)` works.

If there's a `impl CircularlyEnumerable for Filter<I>`, then we're good for the
result of `filter`: `some_iter.filter(...).circular_enumerate(3)` works.

Well, that's progress, I guess. But implementing this for every type that
implements iterator sounds like a ton of busy work. And it still wouldn't be
complete because there are iterators that we can never know about: iterators
defined in private modules by other Rust users. If we can, we'd like to support
those, too.

But also, we aren't really using any implementation details from Map or Filter.
What we really want is something like `impl CircularlyEnumerable for Iterator`.
If it implements Iterator, it implements CircularlyEnumerable. And Rust
actually lets us do something really close to that.

```rust
impl<I> CircularlyEnumerable for I where I: Iterator {
    fn circular_enumerate(self, items: u32) -> CircularEnumerate<Self> {
        // TODO: implement
    }
}
```

This is saying that any arbitrary type `I` — as long as it implements
`Iterator` — implements `CircularlyEnumerable`. Meaning we're all set; we can
write our iterator chain in a fluent style.

```rust
// We need to bring our trait into scope, otherwise
// how will the compiler know about it
use circularly_enumerable::CircularlyEnumerable;

// And this now works as expected.
some_iter
    .map(...)
    .filter(...)
    .circular_enumerate(3)
    .collect()
```

Unfortunately, I simplified the code a little bit. The actual code has some
extra annotations for `Sized` and looks like this.

```rust
// Our extension trait.
trait CircularlyEnumerable {
    fn circular_enumerate(self, items: u32) -> CircularEnumerate<Self> where Self: Sized;
}

// If it implements Iterator (and Sized, I guess), it can call circular_enumerate.
impl<I> CircularlyEnumerable for I where I: Iterator, I: Sized {
    fn circular_enumerate(self, items: u32) -> CircularEnumerate<Self> where Self: Sized {
        // Straightforward implementation: return our structure with the values filled in.
        CircularEnumerate { iter: self, items: items, cur: 0 }
    }
}
```

[_Edit on the Rust Playground_][example2]

I don't actually know what the [`Sized`][rustsized] bits are all about. I
*think* it has something to do with creating instances that the compiler knows,
at compile time, how much space they take up in the stack or the heap. But I'm
not actually sure. I followed the helpful compiler hints until the code worked.

And there we have it. A wrapped iterator from the [previous
post][wrappediterators] that fits quite nicely into the Rust ecosystem.

[wrappediterators]: /wrapped-iterators-in-rust
[extensiontraits]: http://xion.io/post/code/rust-extension-traits.html
[rustmap]: https://doc.rust-lang.org/std/iter/struct.Map.html
[rustenumerate]: https://doc.rust-lang.org/std/iter/struct.Enumerate.html
[rustfilter]: https://doc.rust-lang.org/std/iter/struct.Filter.html
[rustsized]: https://doc.rust-lang.org/std/marker/trait.Sized.html
[iterators]: https://doc.rust-lang.org/std/iter/trait.Iterator.html
[example2]: https://play.rust-lang.org/?gist=23a9b200c330dae82fe36680a728c756&version=stable

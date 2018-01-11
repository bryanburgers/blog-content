I’m pretty new to [Rust][rust], and one of the first things I immediately liked
was Rust's [iterators][iterators].

What I like about the iterators is how many operations are defined on every
different type of iterator. For example, in JavaScript, there is the concept of
an [iterable][jsiterable], but the only thing that can be done on it is put it
in a `for..of` loop. Things like [`map`][jsmap] and [`filter`][jsfilter] are
only defined on `Array`.

On the other hand, in Rust, [`map`][rustmap] and [`filter`][rustfilter] are
defined on any type that implements Iterator. Which means that any iterator
that's added in the future gets all of these features for free. That's pretty
neat.

How they did it — that each function returns a different type that wraps the
original iterator — is a pretty clever solution. For example, if I have an
iterator of type I, and I call `.map(...)` on it, the mapping code doesn't get
run immediately. Instead, I immediately get a new object of type `Map<I>` back
that is itself an iterator wrapping the original.

I wanted to see if I could do something similar. So as a contrived example, I
want to add something like the [`enumerate`][rustenumerate] but that works in a
circle. So if `"my string".chars()` is an iterator that returns `['m', 'y', '
', 's', 't', ...]`, then a circularly-enumerated iterator with a cycle length
of 3 would return `[(0, 'm'), (1, 'y'), (2, ' '), (0, 's'), (1, 't'), ...]`.
Let's give it a try.

```rust
// The structure that holds the wrapped iterator and the current state.
struct CircularEnumerate<I> {
    iter: I,
    items: u32,
    cur: u32,
}

// Implement the iterator.
impl<I> Iterator for CircularEnumerate<I> where I: Iterator {
    type Item = (u32, I::Item);
    
    fn next(&mut self) -> Option<(u32, I::Item)> {
        match self.iter.next() {
            // If the wrapped iterator is done, we're done.
            None => None,
            
            // If the wrapped iterator is not done, tack on
            // the circular enumeration.
            Some(v) => {
                let idx = self.cur;
                // Increment the current value, wrapping it
                // if necessary.
                self.cur = (self.cur + 1) % self.items;
                Some((idx, v))
            }
        }
    }
}

// For now, we need a way to create a new CircularEnumerator.
impl<I> CircularEnumerate<I> {
    fn new(iter: I, items: u32) -> CircularEnumerate<I> {
        CircularEnumerate { iter: iter, items: items, cur: 0 }
    }
}

fn main() {
    for (idx, i) in CircularEnumerate::new("abcdefghijkl".chars(), 3) {
        println!("{}: {}", idx, i);
    }
}
```

[_Edit on the Rust Playground_][example1]

Well awesome. This prints out what we expect. And since this is an iterator, we
can `map`, `filter`, `count`, `take`, `fold`, and `collect` it just like any
other iterator.

In the [Extending the Iterator Trait in Rust][extending] I look at how to
integrate this new iterator into Rust in a natural way.

[rust]: https://www.rust-lang.org/
[iterators]: https://doc.rust-lang.org/std/iter/trait.Iterator.html
[jsiterable]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols
[jsmap]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map
[jsfilter]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter
[rustenumerate]: https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.enumerate
[rustmap]: https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.map
[rustfilter]: https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.filter
[example1]: https://play.rust-lang.org/?gist=f9b2aad2296b7cd8df41e1c103a8acd8&version=stable
[extending]: /extending-iterator-trait-in-rust

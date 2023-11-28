In the [previous article] we learned how to parse numbers from a stream of text using Wasm, in preparation for doing [Advent of Code] challenges. To solve the challenge, we intermixed the *parsing* and the *solving* into a single function; while that may work for some of the easier challenges, eventually we'll need a place to put the parsed result so we can use it more than once.

In this article, we'll dig deeper into Wasm memory, experiment with creating and using more complex entities (structs, classes, whatever you want to call them), and then build a *very simple* custom allocator (no seriously, don't be scared). While we won't actually revisit collecting the results of the parse to handle later – that would make this article too long – this simple allocator will provide the tools necessary to be able to collect results and handle them later.

*Note: if you want to follow along at home, I'm running the Wasm using Node.js's via the code found [in this gist](https://gist.github.com/bryanburgers/ec1be23f116591264ef710a77f49c1fc).*

## Do we even need an allocator?

The first thing to note as we dig into Wasm memory is that we don't *need* an allocator. We can define in our module how much memory we want initially.

```wasm
    ;; Create memory with at least 1 page of 64k of memory    
    (memory $mem 1)
```

And when we have that memory, **WE HAVE ALL THE POWER**. We can read from and write to it across the entire range. It's all ours and we can do whatever we want with it.

```wasm
    (func $main (result i32 i32)
        ;; write some data to an arbitrary memory location
        i32.const 0xcafe ;; address
        i32.const 42     ;; memory to write
        i32.store

        ;; write some data to the last available byte
        i32.const 0xffff
        i32.const 43
        i32.store8_u

        ;; read data from our arbitrary memory location
        i32.const 0xcafe
        i32.load

        ;; read data from the last available byte
        i32.const 0xffff
        i32.load8_u
    )
```

Running that confirms that, yes, haha, it's all ours! And since our Wasm module runs in isolation, we don't need to worry about any other processes trampling on our memory.

```json
[ 42, 43 ]
```

[View full source code][figure-01]

## And if that's not enough...

And if a single page of memory isn't enough, Wasm allows us to ask for *more* memory at runtime. Wasm has two primary [memory instructions] for this: `memory.size` returns how much memory we're *currently* using, and `memory.grow` allows us to request more memory.

So instead of being limited to our initial 64k of memory, we can ask for more at *any time*.

```wasm
   (func $main (result i32 i32 i32)
        ;; write some data to an arbitrary memory location
        i32.const 0xcafe ;; address
        i32.const 42     ;; memory to write
        i32.store

        ;; ask for ten more pages of memory
        i32.const 10
        memory.grow $mem
        drop ;; ignore whatever memory.grow returned

        ;; write some data past the last byte of the first page
        i32.const 0x10000
        i32.const 43
        i32.store8

        ;; read data from our arbitrary memory location
        i32.const 0xcafe
        i32.load

        ;; read data from past the last byte of the first page
        i32.const 0x10000
        i32.load8_u

        ;; get how much memory we have
        memory.size
    )
```

Without `memory.grow` this code would throw an error, but with the `memory.grow` we are allowed to write past the first page, and we can see we've been allocated 11 total pages of memory (our initial page and then ten more).

```json
[ 42, 43, 11 ]
```

[View full source code][figure-02]


## What's the point?

Before we try to understand why we might want to write an allocator, lets take a step back and start thinking about how we might represent more complex entities – structs, objects, anything that is more complex than a single number.

We'll start with a pretty simple type: a point. A point is made up of two numbers; an `x` and a `y`.

If we want to refer to a single point, our best option is to claim a continguous chunk of memory, and pass around the memory address of that chunk of memory. So we define our point as 16 bytes of memory; the first 8 our are `x` value and the second 8 are our `y` value.

```
Point
n       n+8      n+16
+--------+--------+
| x: i32 | y: i32 |
+--------+--------+
```

So if we have a "Point" at memory address `n`, then to get the point's `x` value (`point.x`, `point->x`, however it makes sense depending on your language of choice), we would load the number at memory address... `n`.

```wasm
    (func $point_x (param $point i32) (result i32)
        local.get $point
        i32.load
    )
```

And if we have a "Point" at memory address `n`, then to get the point's `y` value (`point.y`, `point->y`) we would load the number at memory address `n+8`.

```wasm
    (func $point_y (param $point i32) (result i32)
        local.get $point
        i32.const 8
        i32.add
        i32.load
    )
```

With functions defined like that, we can build out all sorts of more advanced functions, like the vertical distance between two points

```wasm
    (func $point_vertical_distance (param $point1 i32) (param $point2 i32) (result i32)
        local.get $point1
        call $point_y
        local.get $point2
        call $point_y
        i32.sub
        ;; exercise for the reader: absolute value
    )
```

and the [Manhattan Distance] (which almost always comes up at least once in Advent of Code).

```wasm
    (func $point_manhattan_distance (param $point1 i32) (param $point2 i32) (result i32)
        local.get $point1
        local.get $point2
        call $point_vertical_distance
        local.get $point1
        local.get $point2
        call $point_horizontal_distance
        i32.add
    )
```

All of those fancy functions are great, but we still haven't answered the most important question: how do we *create* a point? How can we test all of these fancy functions if we don't have any points to start with? If we wanted to define a `$point_create` function; what does it look like?

One option (since we control **ALL THE MEMORY**) is to define a function that takes any memory address we give it and initializes a point *at that memory address*.

So if `$point_initialize` takes three parameters: the memory address to initialize, the x value, and the y value, the result will be that the memory address is a valid point.

```wasm
    (func $point_initialize
        (param $address i32)
        (param $x i32)
        (param $y i32)

        ;; store x at the address given (address + 0)
        local.get $address
        local.get $x
        i32.store

        ;; store y at address + 8
        local.get $address
        i32.const 8
        i32.add
        local.get $y
        i32.store
    )
```

And if we use that in our `$main`, it *works*. I guess.

```wasm
    (func $main (result i32 i32)
        (local $point1 i32)
        (local $point2 i32)

        i32.const 0x10 ;; Choose an arbitrary memory address
        local.set $point1

        i32.const 0x20 ;; Choose another; hope it doesn't overlap!
        local.set $point2

        ;; initialize the first point
        local.get $point1
        i32.const 3
        i32.const 4
        call $point_initialize

        ;; initialize the second point
        local.get $point2
        i32.const 5
        i32.const 7
        call $point_initialize

        ;; and call a function on the points
        local.get $point1
        local.get $point2
        call $point_manhattan_distance
    )
```

And if we run it, we do get the correct Manhattan Distance between `(3, 4)` and `(5, 7)`:

```json
5
```

[View full source code][figure-03]


## Our very first allocator!

But the major problem with the version above is that critical comment: "hope it doesn't overlap!". (And actually, when writing this initially, I *did* accidentally chose addresses that overlapped and got a Manhattan Distance that was *not* correct.) When we have access to all of the memory, we have great power, but it also requires great responsibility.

At some point, we need to keep track of all of these memory addresses to make sure they don't overlap. And if we're creating lots and lots of these points, it may become overwhelming.

What would be ideal is if we could separate "thinking about points" from "thinking about tracking memory address". Like, if there was some magical function where we could ask, "hey, can you give us a bunch of bytes that aren't used anywhere else for this point" and it would give us back memory.

And of course, you've already guessed that the magical function is our allocator. So we'll call it `$alloc`.

```wasm
    (func $point_create (param $x i32) (param $y i32) (result i32)
        (local $point i32)
        ;; A point is 16 bytes of data, so please, magic
        ;; function, give us 16 bytes of unused memory.
        i32.const 16
        call $alloc
        local.set $point

        ;; initialize $x at $point + 0
        local.get $point
        local.get $x
        i32.store

        ;; initialize $y at $point + 8
        local.get $point
        i32.const 8
        i32.add
        local.get $y
        i32.store

        ;; return point
        local.get $point
    )
```

Having that would simplify our `$main` *and* we could get rid of that stressful "hope it doesn't overlap!" comment, because we're not worrying about tracking memory addresses; our magical `$alloc` has that covered.

```wasm
    (func $main (result i32)
        (local $point1 i32)
        (local $point2 i32)

        i32.const 3
        i32.const 4
        call $point_create
        local.set $point1

        i32.const 5
        i32.const 7
        call $point_create
        local.set $point2

        local.get $point1
        local.get $point2
        call $point_manhattan_distance
    )
```

So that's why the `$alloc` function is so important: having one system be responsible for keeping track of all of that used memory and handing out new, unused memory, lets us focus on solving our problems. And if we don't exercise our **POWER OVER ALL MEMORY** and just let `$alloc` handle that, we should never run into any issues!

OK, so let's build an allocator.

[View full source code for this section][figure-04], but beware that this has the definition of `$alloc` in it, which will ruin ~~your supper~~ the rest of this article.


## A bump allocator

We're going to build what is called *in the business* as a "bump allocator".

The way a bump allocator works is that it stores a memory address for some known free memory. And any time it is asked to give out nice fresh new memory, it "bumps" that memory address past the memory it handed out to the next known free memory.

The great thing about a bump allocator is that it's pretty simple to implement.

The bad thing about a bump allocator is that it doesn't actually keep track of used memory, so *freeing* memory is impossible. Once a chunk of memory is handed out, it can never be handed out again. In a complex system that does lots of memory allocations, that's bad news. For us and for most Advent of Code problems, that probably won't become a problem.

So the first thing we need when building a bump allocator is a place to store the memory address. Now, as we've discussed *ad nauseum*, we have control over the entire memory space. So we could just define a particular, fixed address as the address that `$alloc` uses to keep track of the free memory address.

But we're learning Wasm as we go, so lets learn something new. Let's learn about globals!

In our functions, we've already defined locals like `(local $name i32)`. Wasm also has global variables which exist *outside* of functions that can store data— well, globally.

So let's define one.

```wasm
    ;; the memory address of the next allocation
    (global $alloc_offset (mut i32) (i32.const 32))
    ;;           ^            ^                ^
    ;;           |            |                |
    ;;        the name        |                |
    ;;                    the type             |
    ;;                            the initial value
```

The great thing is that we can get and set this variable just like we do with locals: with `global.get` and `global.set`.

That's it. That's all we really need to know to create an allocator.

So let's define our `$alloc` function. When asked for some memory,
* get our current memory address from the global,
* "bump" over the amount of memory that was required,
* store that new, bumped memory address in the global, and
* return the original memory address.

```wasm
    ;; the memory address of the next allocation
    (global $alloc_offset (mut i32) (i32.const 32))
    ;; allocate some memory!
    (func $alloc (param $size i32) (result (;memory address;) i32)
        (local $curr_alloc_addr i32)
        (local $next_alloc_addr i32)

        ;; get the current memory address; we'll return this
        ;; to the caller
        global.get $alloc_offset
        local.set $curr_alloc_addr

        ;; "bump" over the bytes that we're giving
        ;; to the caller, and store that as the next
        ;; available memory address
        local.get $curr_alloc_addr
        local.get $size
        i32.add
        local.set $next_alloc_addr

        ;; store the memory address of the next allocation
        local.get $next_alloc_addr
        global.set $alloc_offset

        ;; and return the current memory address
        local.get $curr_alloc_addr
    )
```

We can confirm that `$alloc` never gives us the same memory (or even overlapping memory) by calling it a bunch of times...

```wasm
    (func $main (result i32 i32 i32 i32)
        i32.const 16
        call $alloc

        i32.const 16
        call $alloc

        i32.const 128
        call $alloc

        i32.const 16
        call $alloc
    )
```

```json
[ 32, 48, 64, 192 ]
```

So, in summary, our bump allocator is really just a global variable that we can increment whenever we want. And we just treat its return value like an address to memory. Hey, that's pretty cool.

[View full source code][figure-05]


## Freeing memory

Now, most of the time when we allocate memory, we want to be responsible and free it later when we're done with it. So let's define `$free` as well.

```wasm
    (func $free (param $address i32)
        ;; HAHAHAHA NO.
    )
```

Just kidding. Bump allocators have no way to track which memory has been freed and is available for reuse. And for our use cases, *that's just fine*. So feel free (har har) to call this function as much as you want.


## What happens if we run out of memory?

At this point, we have an allocator that is completely naive to the fact that Wasm hands out memory pages at a time.

To build a truly robust allocator, even a simple bump allocator, we need to make sure that we use `memory.grow` if we need more memory.

Including that code here and explaning it would be tedius, plus if you've been following along you've already seen conditional logic, so I'll make the source code for my complete `$alloc` function available and leave it up to you whether you want to see it or not.

[View full source code][figure-06]


## The end

It turns out writing an allocator wasn't all that complicated, and in the end we learned about a strategy for creating more complex entities and we learned about Wasm globals.

That feels like a successful day.


[previous article]: https://burgers.io/complete-novice-wasm-parsing-numbers
[Advent of Code]: https://adventofcode.com
[memory instructions]: https://webassembly.github.io/spec/core/syntax/instructions.html#syntax-instr-memory
[Manhattan Distance]: https://en.wikipedia.org/wiki/Taxicab_geometry

[figure-01]: https://gist.github.com/bryanburgers/2b0f08fd583cf0401a958d7e8edc7552#file-figure-01-wat
[figure-02]: https://gist.github.com/bryanburgers/2b0f08fd583cf0401a958d7e8edc7552#file-figure-02-wat
[figure-03]: https://gist.github.com/bryanburgers/2b0f08fd583cf0401a958d7e8edc7552#file-figure-03-wat
[figure-04]: https://gist.github.com/bryanburgers/2b0f08fd583cf0401a958d7e8edc7552#file-figure-04-wat
[figure-05]: https://gist.github.com/bryanburgers/2b0f08fd583cf0401a958d7e8edc7552#file-figure-05-wat
[figure-06]: https://gist.github.com/bryanburgers/2b0f08fd583cf0401a958d7e8edc7552#file-figure-06-wat
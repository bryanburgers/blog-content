I have decided to do [Advent of Code] in [WebAssembly] this year. That must mean I'm some kind of Wasm expert, right? Wrong. I am a *complete* novice, which means that I'm using Advent of Code to help me learn and grow.

It also means that, if you are also a complete novice, hopefully I can take you along on this journey with me and you won't be completely lost. Or in other words, if you're a complete Wasm novice and are also Wasm-curious, you're in the right place.

As of this writing, Advent of Code hasn't started yet. But nearly every AoC challenge involves parsing numbers from the input in some form or another. So in this article I'm solving a pre-AoC challenge:

> Some convoluted backstory about elves and reindeers and stars...
>
> You stumble upon an input that is a sequence of numbers separated by spaces. Add up all the numbers.
>
> Example: if your input is "10 20 30" the answer would be "60".

This article will move *slowly*. Feel free to read, skip, jump around, or whatever is appropriate to your level.

## Bare bones module: a module with an exported main

So first, let's start out with a super basic [wat] file. (Note, if you want to play along at home, the code that is running this uses [Wasmtime] from Rust and all 41 glorius lines can be found [at this gist](https://gist.github.com/bryanburgers/677701679ba825ec6d368156297c5434))

```wasm
(module
    ;; Create a function that returns a single value
    (func $main (result i32)
        i32.const 42
    )

    ;; Export that function so the test harness can find it
    (export "main" (func $main))
)
```

There's not a ton going on here
* `(module ...)` defines a Wasm module. Everything inside of that is part of the module.
* `(func ...)` defines a function. Executes code and whatnot.
* `(func $main ...)` defines a *local* name for the function.
  * `(result i32)` declares that this function returns a single 32-bit integer.
  * `i32.const 42` is a Wasm instruction that puts a single 32-bit intgeter on the stack. Since this is what's on the stack when the function body ends, it gets returned.
* `(export "main" ...)` exports a name from the module, so that other modules or the host can use it.

When we run this, we get a result.

```
i32: 42 (0x2a)
```

That's saying that an i32 (32-bit integer) was returned, and it was the number 42 in decimal (also known as the number 0x2a in hexidecimal). And yep, that checks out: we expected the `$main` function to return 42. A good start!

[View full source code][figure-01]


## Adding some memory and default values: memory and data initialization

OK, now we need something to parse. So lets add some more items to our module.

First, let's declare that our wasm module has a linear memory. And since we don't need a ton, let's make it the smallest memory possible: a single page (a Wasm page is 64kib).

```wasm
    ;; Create memory with at least 1 page of 64k of memory
    (memory $mem 1)
```

Wasm also allows us to statically declare what is *in* that memory when our module starts, and that's how we'll populate the numbers to be parsed. Let's populate it with 5 space-delimited numbers.

```wasm
    ;; Initialize the first several bytes of the memory with some text
    (data (i32.const 0) "5 10 42 193 1007000000 ")
    (; byte offset       01234567890123456789012 ;)
    (;                             1         2   ;)
```

If we run this we get...

```
i32: 42 (0x2a)
```

...which makes sense. We didn't actually change our `main` function.

[View full source code][figure-02]

## Reading from memory: `i32.load8_u`

So now that we have our memory populated, let's read from that memory.

At this point, I need to point out that trying to find documentation for the names of Wasm's instructions is surprisingly *not easy*. MDN has an [instruction refeference](https://developer.mozilla.org/en-US/docs/WebAssembly/Reference) but it's not complete and doesn't always describe what the instruction does. The [Wasm spec](https://webassembly.github.io/spec/core/syntax/instructions.html) describes the instructions, but it's not always easy to search for them, especially if you don't know what you're looking for.

But after some fumbling around, I did manage to find two functions that are useful here: `i32.load8_u` and `i32.load8_s`. Both take an address as input and return the 8-bit value stored in memory as output. (`_u` interprets memory as an unsigned byte; `_s` interprets the data as a signed byte).

So to use this, first we need to push a memory address onto the stack. And then we call `i32.load8_u` which pops the memory address from the stack and pushes the value at that memory address back on to the stack.

```wasm
    (func $main (result i32)
        i32.const 4 ;; [] -> [i32], instruction that pops nothing and pushes an i32
        i32.load8_u ;; [i32] -> [i32], instruction that pops an i32 and pushes an i32
    )
```

So the result of running main should be whatever byte is stored at memory address 4. And if we run this

```
i32: 32 (0x20)
```

yep, that's the ASCII value for the space character stored at memory offset 4.

Wasm functions are typically represented as an array of instructions. But Wasm also has an almost-function-like syntax to instructions.

```wasm
    (func $main (result i32)
        (i32.load8_u (i32.const 4))
    )
```

This results in the same Wasm binary, the result of executing `i32.const 4` first then executing `i32.load8_u`. There are times that this representation is easier to follow, but for this article I'll be sticking to the previous representation.

[View full source code][figure-03]

## Calling a function: the `call` instruction

We currently have two instructions in our main function, and that's not so bad. But pretty soon, I – the Wasm novice – am gonna start getting confused by these instructions. And when that time comes, I'm gonna want some function calls to keep everything making sense. I'm sure you'll have no problem keeping it all straight, but humor me, mmmK?

So let's make a function that does exactly what the `i32.load8_u` instruction does. It will pop an i32 off the stack and push a different i32 on the stack.

```wasm
    (func $load_from_memory (param $address i32) (result i32)
        local.get $address ;; Push the $address parameter onto the stack
        i32.load8_u        ;; and use that to load from memory
    )
```

Yeah, alright. Nothing *too* fancy there. And calling it uses a very predictably named `call` instruction.

```wasm
    (func $main (result i32)
        i32.const 4
        call $load_from_memory
    )
```

As usual, let's run this and see if it still works.

```
i32: 32 (0x20)
```

OK, great. It does.

[View full source code][figure-04]


## OK, let's actually start to parse some numbers

So we've got some data in memory. We want to start parsing it so that we can use it. Let's parse a single digit from a single byte of memory.

In ascii, the numbers are represented by the bytes: 0=0x30, 1=0x31, 2=0x32, etc. So to convert an ascii byte to the actual number it represents, we need to subtract 0x30 from the byte value.

```wasm
    ;; Load an ascii byte from memory and return its numeric value
    (func $parse_digit (param $address i32) (result i32)
        ;; get the byte at the provided address
        local.get $address
        i32.load8_u

        ;; push on a value that represents ascii "0"
        i32.const 0x30
        ;; and subtract them! [i32, i32] -> [i32]
        i32.sub
    )

    (func $main (result i32)
        i32.const 0
        call $parse_digit
    )
```

If we run this, we get

```
i32: 5 (0x5)
```

That's promising! That really is a 5.

[View full source code][figure-05]

If we change our main to call it for a few more values

```wasm
    (data (i32.const 0) "5 10 42 193 1007000000 ")
    (; byte offset       01234567890123456789012 ;)
    (;                             1         2   ;)

    (func $main (result i32 i32 i32 i32)
        i32.const 0
        call $parse_digit ;; expecting 5

        i32.const 3
        call $parse_digit ;; expecting 0, the second digit of "10"

        i32.const 15
        call $parse_digit ;; expecting 7, the fourth digit of "1007..."

        i32.const 4
        call $parse_digit ;; what do we expect?
    )
```

we get

```
i32: 5 (0x5)
i32: 0 (0x0)
i32: 7 (0x7)
i32: -16 (0xfffffff0)
```

which, OK, meets our expectations. And returns well, *something* for non-digit values.

[View full source code][figure-06]

## We can do better: locals and comparing numbers with `i32.ge_s` and `i32.lt_s`

Alright, so returning a random value for bytes that aren't ascii numbers isn't actually all that great. Instead, let's do something reasonable like returning a `-1` for any byte that isn't a digit.

In order to do this, we're going to need to keep track of values in a way that's more complex than just keeping them on the stack. So the first thing we need is a way to store local values.

We can *define* a local value by declaring `(local $name i32)` at the top of a function. And then we can use `local.get $name` to push whatever is in that local onto the stack, or `local.set $name` to pop whatever is on the top of the stack and store it in the local.

If we rewrote our previous `$parse_digit` with locals, it might look like this

```wasm
    ;; Load an ascii byte from memory and return its numeric value
    (func $parse_digit (param $address i32) (result i32)
        ;; declare the local!
        (local $parsed i32)

        ;; get the byte at the provided address
        local.get $address
        i32.load8_u

        ;; push on a value that represents ascii "0"
        i32.const 0x30
        ;; and subtract them! [i32, i32] -> [i32]
        i32.sub

        ;; pop the value off the stack and store it in $parsed
        ;; now our stack is empty, and $parsed has a value
        local.set $parsed

        ;; get whatever value is in $parsed and push it to the
        ;; stack. Now we have one item on our stack *and* $parsed
        ;; still has a value.
        local.get $parsed
    )
```

OK, so that doesn't *actually* do anything different. The `local.set` followed by `local.get` is kinda useless. But we're learning with small steps here.

However, now that we know how to use locals, we can re-use values, shift them around, store them for later; whatever we need to do.

For example, if we have a value in `$parsed`, now we can get it and compare it to another value. Like, say, if we wanted to see if the value was greater than or equal to zero. Or less than ten. Which is exactly what we want to do. We can use `i32.ge_s` and `i32.lt_s` to do it.

Wait, what's the `_s` about? In general, Wasm's `i32` represents a 32-bit value. That 32-bit value *could* be interpreted as an unsigned 32-bit number. Or it *could* be interpreted as a signed [two's-complement] 32-bit number. When we're using instructions like `i32.add`, it doesn't matter, because adding two unsigned numbers and adding two signed two's-complement numbers just happens to be the same, based on the bit patterns. But when comparing two numbers, we absolutely need to know whether those numbers should be interpreted as signed or unsigned. `i32.ge_s` treats the numbers as – you guessed it – signed. And you can also probably guess that there is an `i32.ge_u` instruction that treats them as unsigned.

We'll treat our numbers as signed and use the `_s` variants.

```wasm
    (func $parse_digit (param $address i32) (result i32)
        (local $parsed i32)

        ;; A few more locals!
        (local $ge_zero i32)
        (local $lt_ten i32)

        ;; get the byte at the provided address
        local.get $address
        i32.load8_u

        ;; subtract ascii "0" and store it
        i32.const 0x30
        i32.sub
        local.set $parsed

        ;; is $parsed >= 0?
        local.get $parsed
        i32.const 0
        i32.ge_s
        local.set $ge_zero

        ;; is $parsed < 10?
        local.get $parsed
        i32.const 10
        i32.lt_s
        local.set $lt_ten
        
        ;; is $parsed >= 0 && $parsed < 10
        local.get $ge_zero
        local.get $lt_ten
        i32.and
    )
```

If we run this, we now get

```
i32: 1 (0x1)
i32: 1 (0x1)
i32: 1 (0x1)
i32: 0 (0x0)
```

which, yeah, I guess that result is telling us that the first three numbers *are* valid digits and the last one *is not* a valid digit. But we actually still need to return the valid digit.

[View full source code][figure-07]

What we need is an if statement! Which, after looking at some examples, I found out Wasm has!

So if we add one more local to the beginning of the function

```wasm
    (local $result i32)
```

we can use an if block to set its value and then return it.

```wasm
    (func $parse_digit (param $address i32) (result i32)
        ;; ...
        (local $result i32)
        ;; ...

        ;; is $parsed >= 0 && $parsed < 10
        local.get $ge_zero
        local.get $lt_ten
        i32.and

        ;; if what's on the stack...
        (if
            ;; ... is true, then ...
            (then
                ;; ... set the result to what we parsed
                local.get $parsed
                local.set $result
            )
            ;; ... otherwise ...
            (else
                ;; ... set the result to a constant
                i32.const -1
                local.set $result
            )
        )

        ;; and return the result
        local.get $result
    )
```

And checking that it works... it does. The first three are parsed as digits, and the fourth is not a digit, so it returns `-1`.

```
i32: 5 (0x5)
i32: 0 (0x0)
i32: 7 (0x7)
i32: -1 (0xffffffff)
```

Look at us go!

[View full source code][figure-08]

## We can do even *better*: returning multiple values

Using `-1` as a [sentinel](https://en.wikipedia.org/wiki/Sentinel_value) is cool and all. But what will be even more valuable in the long run is returning two values: the value *and* whether it was successful. So we'll turn `$parse_digit` into a function that takes a single value and returns two values: the digit value and whether it parsed correctly. We'll return whether it parsed correctly as either a `0` (it didn't parse correctly) or a `1` (it did parse correctly).

So to start, we need to declare `$parse_digit` as a function that returns two results.

```wasm
    (func $parse_digit
        (param $address i32)
        (result (; the value ;) i32 (; correctly parsed ;) i32)
        ;; ...
    )
```

and then we need to track and return both the parsed value *and* whether it was successful.

```wasm
    (func $parse_digit
        (param $address i32)
        (result (; the value ;) i32 (; correctly parsed ;) i32)

        (local $result i32)
        (local $success i32)
        ;; ...
    
        ;; is $parsed >= 0 && $parsed < 10
        local.get $ge_zero
        local.get $lt_ten
        i32.and
        local.tee $success ;; store the value *and* leave it on the stack

        ;; ...

        ;; and return the results
        local.get $result
        local.get $success
    )
```

Modifying `$main` to return 8 values and running it yields

```
i32: 5 (0x5)
i32: 1 (0x1)
i32: 0 (0x0)
i32: 1 (0x1)
i32: 7 (0x7)
i32: 1 (0x1)
i32: -1 (0xffffffff)
i32: 0 (0x0)
```

which means: `5` was parsed correctly (`1`), `0` was parsed correctly (`1`), 7 was parsed correctly (`1`), and the last value we tried to parse was *not* a digit – we return an arbitrary value (still `-1`) and indicate it was not parsed correctly (`0`). Hashtag party hat.

[View full source code][figure-09]

## Let's parse a number: looping

I think we've nailed parsing a single digit. Now it's time to expand. Let's parse a string of digits as a number.

Our algorithm will look roughly like this pseudocode. For every new digit we see, multiply our current number by 10 and add the digit.

```
number = 0
for digit in digits {
    number = 10 * number + digit
}
return number
```

In order to accomplish this, we'll need to learn about two new concepts in Wasm: `block` and `loop`.

A `block` is a sequence of instructions, and if we break from it using `br`, execution jumps to the instruction immediately following the block.

```wasm
(block $named_block
    ;; some instructions ...

    br $named_block ;; break out of this block

    ;; these instructions are skipped ...
)
;; execution resumes here
```

We can think of `br` in a block as a `break`.

A `loop` is similar to a `block` except when we use the `br` instruction, execution jumps to the beginning of the `loop`.

```wasm
(loop $named_loop
    ;; top

    ;; some instructions ...

    br $named_loop ;; jump to the top of this block

    ;; these instructions are skipped ...
)

;; the above is an infinite loop; we never reach here
```

We can think of `br` in a loop as a `continue`.

Combining that with some comparison and if statements, we can run a piece of code 10 times:

```wasm
(func $run_ten_times
    (local $counter i32)
    i32.const 0
    local.set $counter

    (block $outer
        (loop $inner
            ;; Run some code

            ;; is $counter >= 10?
            local.get $counter
            i32.const 10
            i32.ge_s

            ;; if $counter >= 10
            (if
                ;; ... then break out of this loop
                (then
                    br $outer
                )
            )

            ;; increment counter by 1
            local.get $counter
            i32.const 1
            i32.add
            local.set $counter

            ;; rerun the loop
            br $inner
        )
    )
)
```

OK, so if we know how to loop, we can now build our function that parses a number.

```wasm
(func $parse_number
    (param $address i32)
    (result (; number ;) i32 (; bytes parsed ;) i32)

    ;; ...
)
```

Like before, we're going to return two values. First, we're going to return the number that was parsed. For the second one, we're going to return *how many bytes* were parsed. This is slightly different than `$parse_digit` – *or is it!?* – but knowing how many bytes we've parsed from memory will allow us to track where we are in memory after we finish parsing a number.

Next we'll declare some locals that we'll use.

```wasm
        (local $bytes_parsed i32) ;; how many bytes have been parsed
        (local $current_address i32) ;; the address of the next digit to parse
        (local $parse_digit_value i32) ;; first return value from parse_digit
        (local $parse_digit_success i32) ;; second return value from parse_digit
        (local $number i32) ;; the number that we're parsing
```

And we'll initialize a few of them.

```wasm
        ;; initialize $number to 0
        i32.const 0
        local.set $number
        
        ;; initialize $bytes_parsed to 0
        i32.const 0
        local.set $bytes_parsed
        
        ;; initialize $current_address to the input parameter $address
        local.get $address
        local.set $current_address
```

So far so good. Nothing too wild going on here. And in the theme of nothing too wild, we can also fill in the *end* of our function. We're gonna return the number parsed and the number of bytes parsed.

```wasm
        local.get $number
        local.get $bytes_parsed
```

Where things do get wild is in the middle. Let's start parsing and looping. We'll use that `block` and `loop` pair again so we can either continue or break out depending on what we parse.

```wasm
        (block $outer
            (loop $inner
                ;; call $parse_digit
                local.get $current_address
                call $parse_digit
                local.set $parse_digit_success
                local.set $parse_digit_value

                ;; ... do other things ...

                ;; increment the $current_address
                local.get $current_address
                i32.const 1
                i32.add
                local.set $current_address

                ;; and parse the next digit
                br $inner
            )
        )
```

So each time around the loop, we're going to parse the next digit from memory.

All that's left then is determining whether the digit we parsed was actually a valid digit.

```wasm
                ;; call $parse_digit

                ;; if $parse_digit parsed a digit ...
                local.get $parse_digit_success
                (if
                    (then
                        ;; increment the number of bytes parsed
                        local.get $bytes_parsed
                        i32.const 1
                        i32.add
                        local.set $bytes_parsed

                        ;; $number = $number * 10 + $parse_digit_value
                        local.get $number
                        i32.const 10
                        i32.mul
                        local.get $parse_digit_value
                        i32.add
                        local.set $number
                    )
                    (else
                        ;; break; exit the loop
                        br $outer
                    )
                )

                ;; increment the $current_address
                ;; and parse the next digit
```

And we've got a working number parser!

We can test it out in our `$main` function.

```wasm
    (data (i32.const 0) "5 10 42 193 1007000000 ")
    (; byte offset       01234567890123456789012 ;)
    (;                             1         2   ;)

    (func $main (result i32 i32 i32 i32 i32 i32 i32 i32)
        i32.const 0
        call $parse_number ;; expecting (5, 1)

        i32.const 2
        call $parse_number ;; expecting (10, 2)

        i32.const 12
        call $parse_number ;; expecting (1007000000, 10)

        i32.const 4
        call $parse_number ;; expecting (0, 0)
    )
```

And verify it works...

```
i32: 5 (0x5)
i32: 1 (0x1)
i32: 10 (0xa)
i32: 2 (0x2)
i32: 1007000000 (0x3c0599c0)
i32: 10 (0xa)
i32: 0 (0x0)
i32: 0 (0x0)
```

...which it does! Now we're cooking with Crisco.

[View full source code][figure-10]


## Parsing a space

At this point, we're so close to being able to parse all of our numbers and add them up. But as a quick aside, we're also going to need a function that can parse a space.

I'm gonna drop this here with very little explanation, because I *think* we can understand what it's doing now.

```wasm
    (func $parse_space
        (param $address i32)
        (result (; 1 for success, 0 for failure (also, number of bytes parsed) ;) i32)

        local.get $address
        i32.load8_u
        i32.const 0x20 ;; 0x20 is ascii for space
        i32.eq
    )
```

[View full source code][figure-11]


## Finally! Adding all of the numbers

And finally! We can parse all of the space-separated numbers and add them all up, just like the elf/reindeer/star backstory challenge asked us to do!

Again we will need a loop. Inside that loop, we'll do *two* things: first we'll parse a number from memory and add that to a local sum variable. And then we'll parse a space. If *either* of those fail, we'll break out of the loop and return the sum.

```wasm
    (func $main (result i32)
        (local $address i32) ;; current address
        (local $sum i32) ;; the sum so far
        (local $parse_number_value i32) ;; first result from $parse_number
        (local $parse_number_result i32) ;; second result from $parse_number

        ;; start parsing at byte 0
        i32.const 0
        local.set $address

        ;; initialize $sum to 0
        i32.const 0
        local.set $sum

        (block $block
            (loop $loop
                ;; parse a number
                local.get $address
                call $parse_number
                local.set $parse_number_result
                local.set $parse_number_value

                ;; if parsing succeeded (at least 1 byte was parsed) ...
                local.get $parse_number_result
                (if
                    (then
                        ;; increment our next address by the number of bytes
                        ;; we parsed
                        local.get $address
                        local.get $parse_number_result
                        i32.add
                        local.set $address

                        ;; add to our sum
                        local.get $sum
                        local.get $parse_number_value
                        i32.add
                        local.set $sum
                    )
                    (else
                        ;; No number parsed; stop
                        br $block
                    )
                )

                ;; now parse a space
                local.get $address
                call $parse_space

                ;; we don't need a local for the result because we'll check it
                ;; immediately
                (if
                    (then
                        ;; it was a space! Increment the address to pass over
                        ;; the space
                        local.get $address
                        i32.const 1
                        i32.add
                        local.set $address
                    )
                    (else
                        ;; it wasn't a space; stop
                        br $block
                    )
                )

                ;; and continue parsing more numbers
                br $loop
            )
        )

        ;; we're done! Return our current sum
        local.get $sum
    )
```

Running this we get...

```
i32: 1007000250 (0x3c059aba)
```

which, yep, when I add them up manually, is the sum of all 5 numbers. We win!

[View full source code][figure-12]


## Conclusion

It was slow going, but we made it to this point which means we have a basic understanding of Wasm and how to parse numbers from input.

We went from basically nothing to understanding locals, loops, comparisons, arithmetic, memory, and parsing.

Hopefully this will be enough to get us through at least a few days of Advent of Code!



[Advent of Code]: https://adventofcode.com/
[WebAssembly]: https://webassembly.org/
[Wasmtime]: https://wasmtime.dev/
[wat]: https://developer.mozilla.org/en-US/docs/WebAssembly/Understanding_the_text_format
[two's-complement]: https://en.wikipedia.org/wiki/Two's_complement

[figure-01]: https://gist.github.com/bryanburgers/a2837026fff8d32de5131f488336f0c1#file-figure-01-wat
[figure-02]: https://gist.github.com/bryanburgers/a2837026fff8d32de5131f488336f0c1#file-figure-02-wat
[figure-03]: https://gist.github.com/bryanburgers/a2837026fff8d32de5131f488336f0c1#file-figure-03-wat
[figure-04]: https://gist.github.com/bryanburgers/a2837026fff8d32de5131f488336f0c1#file-figure-04-wat
[figure-05]: https://gist.github.com/bryanburgers/a2837026fff8d32de5131f488336f0c1#file-figure-05-wat
[figure-06]: https://gist.github.com/bryanburgers/a2837026fff8d32de5131f488336f0c1#file-figure-06-wat
[figure-07]: https://gist.github.com/bryanburgers/a2837026fff8d32de5131f488336f0c1#file-figure-07-wat
[figure-08]: https://gist.github.com/bryanburgers/a2837026fff8d32de5131f488336f0c1#file-figure-08-wat
[figure-09]: https://gist.github.com/bryanburgers/a2837026fff8d32de5131f488336f0c1#file-figure-09-wat
[figure-10]: https://gist.github.com/bryanburgers/a2837026fff8d32de5131f488336f0c1#file-figure-10-wat
[figure-11]: https://gist.github.com/bryanburgers/a2837026fff8d32de5131f488336f0c1#file-figure-11-wat
[figure-12]: https://gist.github.com/bryanburgers/a2837026fff8d32de5131f488336f0c1#file-figure-12-wat
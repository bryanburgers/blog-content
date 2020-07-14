It’s 2020. The world is abuzz with, well – _waves hand_ – all this. But it's
also abuzz with [ARM]: Apple is moving their Mac line to Apple Silicon, Amazon
has been been touting their Gravaton2-based infrastructure, Cloudflare is said
to be going all ARM on their edge nodes.

For a long time, I – and I think many others – held the idea that [x86-64] is
for heavy workloads and ARM is for the "little stuff" like embedded systems and
Raspberry Pis.

And that’s why you can find tons of articles online for how to cross-compile
Rust _from_ x86-64 _to_ ARM.

I want to cross-compile the other way.

At [Zenlist], we use Rust to run our entire fleet of lambdas (around 90 of
them).

Amazon has been talking up their beefy Gravaton2-based ARM instances. But right
now, [lambda] – which feels to me like the "little stuff" because of their
low-memory/low-CPU settings – only runs on x86-64.

I decided we needed to figure out how to build our lambda runtimes from an ARM
server: I needed to cross-compile Rust _from_ ARM _to_ x86-64.

---

So here we go.

## Getting started

I launched two Ubuntu 20.04 instances: a [c6g.medium][c6g] for the build
machine and a [t3.micro][t3] as an x86-64 to run the results on. (Cut me some
slack here! Including how to compile and run on lambda would make this article
quite a bit longer!)

Log into that build machine and lets get started. A quick visit to [rustup.rs]
to install Rust, of course.

And a `apt-get install -y build-essential` for all of the other tools we need.

And finally,

```
$ cargo new --bin crossed
$ cargo run
Hello, world!
```

gets a nice little greeting printed the to the screen. We're in business!

## Cross compile!

Alright, so now we want to cross compile this admittedly awesome project to run
on the target machine.

We'll start in the usual place: adding a new target.

```
rustup target add x86_64-unknown-linux-musl
```

On lambda, it's nice to use the `musl` target to get everything statically
linked together.

Alright, let's give it a try.

```
$ cargo build --release --target x86_64-unknown-linux-musl
error: linking with `cc` failed: exit code: 1
```

Well bummer. I was really hoping that would Just Work™. No need to panic yet,
because I remember from reading other articles like [Rust on
Lambda][cross-compile-lambda] and [Cross compiling Rust on Mac OS for an ARM
Linux router][cross-compile-linux] that we probably just need to configure a
cross-compiling linker.

Alright, so we need to install a cross compiler.

```
sudo apt install gcc-x86-64-linux-gnu
```

Then we need to set it up by throwing this in `~/.cargo/config`.

```toml
[target.x86_64-unknown-linux-musl]
linker = "x86_64-linux-gnu-gcc"
```

And now, `cargo build --release --target x86_64-unknown-linux-musl` works
great!

## And it works on the target machine, right?

Well, I had better make sure. That's why I spun up the target machine, after
all.

```
$ scp target/x86_64-unknown-linux-musl/release/crossed ubuntu@target-machine:.
$ ssh ubuntu@target-machine ./crossed
Hello, world!
```

🎉🏆

## The author counts his chickens before they hatch

So that's how to cross-compile for x86-64 lambda from an ARM machine! Publish
this post. [Submit the sitemap to Google][submit-to-google]. Actually, hey, we can
probably do that right from this project!

I'll just add [reqwest] to the `Cargo.toml`.

```toml
[dependencies]
reqwest = { version = "0.10", default-features = false, features = ["rustls-tls", "blocking"] }
```

And write up some quick code to notify Google that my sitemap has changed, and
it should index this clearly-complete-and-not-missing-anything article.

```rust
fn main() -> Result<(), Box<dyn std::error::Error>> {
    let client = reqwest::blocking::Client::default();
    let url = "http://www.google.com/ping?sitemap=https://burgers.io/sitemap.xml";
    let status = client.get(url)
        .send()?
        .status();

    println!("{:?}", status);

    Ok(())
}
```

And cross-compile it while I get a celebratory drink.

```
$ cargo build --release --target x86_64-unknown-linux-musl
error: failed to run custom build command for `ring v0.16.15`
```

## Wait, what?

Everything was sailing along so smoothly! What happened?

Not-rust happened.

I told *Cargo* about my cross-compiler. But [ring] – a crypto library that
reqwest uses – uses some code that isn't Rust, and it had no way to know about
my cross compiler.

Let's take a longer look at more of the error log.

```
$ cargo build --release --target x86_64-unknown-linux-musl
error: failed to run custom build command for `ring v0.16.15`

Caused by:
  process didn't exit successfully: `/home/ubuntu/crossed/target/release/build/ring-082dc03a36bb5ba5/build-script-build` (exit code: 101)
--- stdout
OPT_LEVEL = Some("3")
TARGET = Some("x86_64-unknown-linux-musl")
HOST = Some("aarch64-unknown-linux-gnu")
CC_x86_64-unknown-linux-musl = None
CC_x86_64_unknown_linux_musl = None
TARGET_CC = None
CC = None
CROSS_COMPILE = None
CFLAGS_x86_64-unknown-linux-musl = None
CFLAGS_x86_64_unknown_linux_musl = None
TARGET_CFLAGS = None
CFLAGS = None
CRATE_CC_NO_DEFAULTS = None
DEBUG = Some("false")
CARGO_CFG_TARGET_FEATURE = Some("fxsr,sse,sse2")
```

Yep, yep, looks good, target is `x86_64-unknown-linux-musl`, host is
`aarch64-unknown-linux-gnu`, uh huh, yep, **oh**.

That's interesting.

There's an environment variable there – `CC_x86_64_unknown_linux_musl` – that
looks like it's expecting the exact cross-compiler to use. And it's empty.

Well, let's change that! Using the exact same cross-compiler that we specified
in the cargo config, let's set that environment variable.

```
$ export CC_x86_64_unknown_linux_musl=x86_64-linux-gnu-gcc
```

And then 🤞.

```
$ cargo build --release --target x86_64-unknown-linux-musl
```

It worked! Throw it over to the target machine and give it a try!

```
$ scp target/x86_64-unknown-linux-musl/release/crossed ubuntu@target-machine:.
$ ssh ubuntu@target-machine ./crossed
200
```

Nice.


## Some final notes

As bullet points? Why not.

* Make sure `CC_x86_64_unknown_linux_musl` is permanently in your environment
  somehow. For us, this involved adding it to the environment section for
  self-hosted GitHub runners.
* In my specific testing for our specific projects, compilation times were up
  to about 10% slower than on a corresponding [c5] instance. But the cost
  tradeoff was worth it.
* I'm holding out hope that AWS will give us ARM-based lambdas in the future.
* No instances were harmed while writing this article. Well, except the ones I
  terminated. Which reminds me to remind you: if you were following along at
  home, now might be a good time to terminate your instances.


[ARM]: https://en.wikipedia.org/wiki/ARM_architecture
[x86-64]: https://en.wikipedia.org/wiki/X86-64
[c6g]: https://aws.amazon.com/ec2/instance-types/c6/
[c5]: https://aws.amazon.com/ec2/instance-types/c5/
[t3]: https://aws.amazon.com/ec2/instance-types/t3/
[rustup.rs]: https://rustup.rs/
[submit-to-google]: https://developers.google.com/search/docs/guides/submit-URLs
[reqwest]: https://docs.rs/reqwest
[cross-compile-lambda]: https://rendered-obsolete.github.io/2019/03/19/rust_lambda.html
[cross-compile-linux]: https://sigmaris.info/blog/2019/02/cross-compiling-rust-on-mac-os-for-an-arm-linux-router/
[ring]: https://docs.rs/ring
[Zenlist]: https://zenlist.com
[lambda]: https://aws.amazon.com/lambda/

Not so long ago, [Fastly introduced a new platform for edge
programming][fastly-announcement]. Fastly's vision is to use wasm to sandbox
and run applications, which means that any language that can compile to wasm
can run on their edge servers.

The idea of edge functions is by no means new. Amazon does it with Lambda,
CloudFlare has Workers, Google Cloud and Azure both call theirs Functions.

To the best of my knowledge, Fastly's offering is unique in its use of wasm,
and this has enabled them to have [really fast boot and response
times][fastly-performance].

Count me interested. Even though Fastly only has a limited preview right now
(called Terrarium), I wanted to start digging in and start playing.


## Getting started

A great place to start playing with Terrarium is in their [web
editor][terrarium].

The web editor has plenty of useful examples, and is a great place to try out
some code and see it running in Terrarium. Poke around in all of the example
projects, they're pretty cool.


## Growing out of the web editor

Eventually, you might get to the point where the web editor isn't powerful
enough. For me, it was when I wanted to bring in an image manipulation library.
The web editor doesn't let you define your own depenencies, so I was stuck.

I toyed with the idea of copying all of the source from [image-rs][image-rs]
into the web editor, but including all of that source, and all of the
transitive dependencies' source, would have been too much of a burden.

The claim of [lucet][lucet] (the engine behind Terrarium) is that it takes wasm
files and runs them. Well, we can compile wasm locally, so we should be able to
build the project locally and run it, right?

We can.


## Creating the project

So let's do it.

```bash
cargo new --lib terrarium-test
```

The first problem we run into is that Terrarium needs to know what function to
execute. And it has certain lifecycle expectations. So we need to be able to
hook into its lifecycle.

The way to do this is via their [rust-guest][guest] library. I haven't seen it
on crates.io yet, but they have generously made it available on GitHub.

So now we can set up our `Cargo.toml` file with everything we need to build our
module.

```toml
[lib]
crate-type = ["cdylib"]

[dependencies]
http_guest = { git = "https://github.com/fastly/terrarium-rust-guest" }
image = "^0.22.1"
```


## Write the code

That's all well and good, but we need some actual code. Here's a bit that takes
any request (ignoring everything about the request, including the URL),
generates a small image and responds with it.

```rust
use http_guest::{guest_app, Request, Response};
use image::DynamicImage;

/// The entrypoint on Terrarium: called for every HTTP request, and
/// expected to return an HTTP response.
pub fn user_entrypoint(_req: &Request<Vec<u8>>) -> Response<Vec<u8>> {
    let image = generate_image();

    // Turn the image into bytes
    let mut vec: Vec<u8> = Vec::new();
    image
        .write_to(&mut vec, image::ImageOutputFormat::PNG)
        .unwrap();

    // Return the response
    Response::builder()
        .status(200)
        .header("content-type", "image/png")
        .body(vec)
        .unwrap()
}

/// Create a new image to serve.
fn generate_image() -> DynamicImage {
    let mut img = DynamicImage::new_rgb8(256, 256);
    img.as_mut_rgb8()
        .unwrap()
        .enumerate_pixels_mut()
        .for_each(|(x, y, px)| {
            px[0] = x as u8;
            px[1] = y as u8;
            px[2] = (256 - (x as i32 + y as i32)).abs() as u8;
        });
    img
}

// Tell Terrarium about the entrypoint.
guest_app!(user_entrypoint);
```

OK, OK, cut me some slack. This may not be the most compelling example, but
it's something trivial enough to fit in a blog post, but complicated enough
that it can't be done on the web interface.


## Compile the code

With our code all ready, let's get a wasm file. If you've never compiled to
wasm before (which I hadn't), you'll need to use `rustup` to download the wasm
target.

```bash
rustup target install wasm32-unknown-unknown
```

And once we have that, we can build the project! Surprisingly straightforward.

```bash
cargo build --release --target wasm32-unknown-unknown
```

If everything goes well, we should end up with a .wasm file in
`target/wasm32-unknown-unknown`.


## 🚀 Launch it!

If you poke around in the Terrarium web editor in the build.ts file, it looks
like Terrarium always expects the module at `module.wasm`. So we can upload our
module to that location, manually disable the `build.ts` so we don't overwrite
it, and hit "Build & Deploy".

But it turns out there's an even easier way where we don't have to do any
manual steps in the web editor.

Fastly also provides a command line tool called [terrctl][terrctl] that will
launch an application from a `module.wasm` for us! Download the right
application for your architecture and invoke `terrctl` to launch on Terrarium.

```shell
$ cp target/wasm32-unknown-unknown/release/*.wasm module.wasm
$ terrctl module.wasm
[INFO] Preparing upload of directory [module.wasm]
[INFO] Guessed programming language: wasm
[NOTICE] Upload in progress...
[NOTICE] Upload done, compilation in progress...
[INFO] Upload complete, waiting for build...
[INFO] Building...
[INFO] Generating machine code...
[INFO] Codegen complete...
[INFO] Deploy complete: https://involved-mental-write-window.fastly-terrarium.com/
[INFO] Instance is deployed
[NOTICE] Instance is running and reachable over HTTPS
[NOTICE] New instance deployed at [https://involved-mental-write-window.fastly-terrarium.com]
```

The output of this command tells us the URL of our brand new WASM-on-the-Edge
module! Open the URL and we should see a dynamically generated image. Woohoo!


## The Future

I really love this. I think it could be a huge step forward for serverless
functions. I can't wait to see Fastly productize more.

Right now, there are some limitations. The function that you deploy only lives
for a handful of minutes, which means we can't currently create anything but
short-lived toys here.

There's also no way to set up secrets. If you look at their GitHub example,
they hardcode the API token right into the app. I can see a future where they
will allow us to set up environment variables or use some other mechanism to
separate secrets and configuration from code.

But I think this product really has a future, so I assume Fastly will build the
ecosystem around it soon enough. I look forward to that day. WASM on the edge
has a bright future!


[terrarium]: https://www.fastlylabs.com/#Terrarium
[guest]: https://github.com/fastly/terrarium-rust-guest
[terrctl]: https://github.com/fastly/terrctl
[image-rs]: https://crates.io/crates/image
[lucet]: https://github.com/fastly/lucet
[fastly-announcement]: https://www.fastly.com/blog/edge-programming-rust-web-assembly
[fastly-performance]: https://www.fastly.com/blog/lucet-performance-and-lifecycle

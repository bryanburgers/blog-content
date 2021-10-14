When it comes to logging and analysis in Rust, I get the gut feeling that [`tracing`] is the way of the future. The [rust compiler uses `tracing`](https://github.com/rust-lang/rust/pull/74726). Tokio is [doing cool things with it](https://tokio.rs/blog/2021-09-console-dev-diary-1). Even the GraphQL library we use [has integrations with tracing](https://docs.rs/async-graphql/2.10.3/async_graphql/extensions/struct.Tracing.html).

Because that seems like where the ecosystem is heading, I wanted to replace the [`log`]-based logger we use at [Zenlist] with one built on [`tracing`]. But for whatever reason, [`tracing`]'s JSON logger just never really felt quite right.

You know what that means: time to build our own! If you want to learn how to build your own logger, want to understand more about tracing layers, or just want to come along, let's take this journey together.

In this article, we'll discover how to build a logger that prints out *moment-in-time* events. In the next, we'll look at how we can use [`tracing`]'s *period-of-time* spans to provide better structured logs.


## Laying the groundwork

Let's get the boring stuff out of the way. `cargo new --bin` and whatnot.

First we'll need to pull in some dependencies. [`tracing`] is the heart of the ecosystem. [`tracing-subscriber`] is the crate responsible for capturing spans and events and doing something with them. And since our custom logger will output structured JSON, let's bring in [`serde_json`] too.

<figure>

```toml
[dependencies]
serde_json = "1"
tracing = "0.1"
tracing-subscriber = "0.2"
```

<figcaption>If you want to follow along, all of the code is at <a href="https://github.com/bryanburgers/tracing-blog-post">github.com/bryanburgers/tracing-blog-post</a>. This is the <a href="https://github.com/bryanburgers/tracing-blog-post/blob/main/Cargo.toml">Cargo.toml</a>.</figcaption>
</figure>

In our `main.rs`, we'll start with a very basic program: set up the custom logger, and then use `info!` to log something simple.

<figure>

```rust
use tracing::info;
use tracing_subscriber::prelude::*;

mod custom_layer;
use custom_layer::CustomLayer;

fn main() {
    // Set up how `tracing-subscriber` will deal with tracing data.
    tracing_subscriber::registry().with(CustomLayer).init();

    // Log something simple. In `tracing` parlance, this creates an "event".
    info!(a_bool = true, answer = 42, message = "first example");
}
```

<figcaption><a href="https://github.com/bryanburgers/tracing-blog-post/blob/main/examples/figure_0/main.rs">examples/figure_0/main.rs</a></figcaption>
</figure>

With that out of the way, let's dive right in and create our custom logger. [`tracing-subscriber`] exposes a trait called [`Layer`] for functionality that deals with tracing spans and events. That's where we want to start.

<figure>

```rust
use tracing_subscriber::Layer;

pub struct CustomLayer;

impl<S> Layer<S> for CustomLayer where S: tracing::Subscriber {}
```

<figcaption><a href="https://github.com/bryanburgers/tracing-blog-post/blob/main/examples/figure_0/custom_layer.rs">examples/figure_0/custom_layer.rs</a></figcaption>
</figure>

And if we run this, we get — drum roll please — well nothing. That's not surprising: we haven't implemented any of [`Layer`]'s methods yet.


## Capturing events

So let's start poking around. In tracing, any time `info!`, `error!`, [or their friends][event-macros] get called, an [`Event`] gets created.

And woah! Look at that! The [`Layer`] interface has an [`on_event`] method. Let's see what we can do.

<figure>

```rust
impl<S> Layer<S> for CustomLayer
where
    S: tracing::Subscriber,
{
    fn on_event(
        &self,
        event: &tracing::Event<'_>,
        _ctx: tracing_subscriber::layer::Context<'_, S>,
    ) {
        println!("Got event!");
        println!("  level={:?}", event.metadata().level());
        println!("  target={:?}", event.metadata().target());
        println!("  name={:?}", event.metadata().name());
        for field in event.fields() {
            println!("  field={}", field.name());
        }
    }
}
```

<figcaption><a href="https://github.com/bryanburgers/tracing-blog-post/blob/main/examples/figure_1/custom_layer.rs">examples/figure_1/custom_layer.rs</a></figcaption>
</figure>

According to the API, we see that we can get the level, the target (which seems to be [the module path where the event was created][`target`]), and the name of the event. And the names of the fields.

Run this and we actually get something useful! Alright, this isn't JSON – we'll get there. Don't pretend like you've never done `println!` exploration or debugging before.

<figure>

```
Got event!
  level=Level(Info)
  target="figure_1"
  name="event examples/figure_1/main.rs:10"
  field=a_bool
  field=answer
  field=message
```

<figcaption>

`cargo run --example figure_1`

</figcaption>
</figure>

OK, this is a great start. But poking around on [`Event::fields()`], there doesn't seem to be a way to get the values of the fields.

So— how are we supposed to create a logger if we can't get the values of what we want to log?


## Visitors

[`tracing`] made a choice to never permanently store the data it's given in events and spans. If we want to get the data, we need to store it ourselves.

The way we do that is via the Visitor Pattern. So it looks like we need to create an implementer of [`Visit`] to get the values out of the event. Alright, let's do some more `println!` exploration.

[`Visit`] exposes a `record_*` method for each type that `tracing` can handle. So we'll hook them all up to print out the name and the value.

<figure>

```rust
struct PrintlnVisitor;

impl tracing::field::Visit for PrintlnVisitor {
    fn record_f64(&mut self, field: &tracing::field::Field, value: f64) {
        println!("  field={} value={}", field.name(), value)
    }

    fn record_i64(&mut self, field: &tracing::field::Field, value: i64) {
        println!("  field={} value={}", field.name(), value)
    }

    fn record_u64(&mut self, field: &tracing::field::Field, value: u64) {
        println!("  field={} value={}", field.name(), value)
    }

    fn record_bool(&mut self, field: &tracing::field::Field, value: bool) {
        println!("  field={} value={}", field.name(), value)
    }

    fn record_str(&mut self, field: &tracing::field::Field, value: &str) {
        println!("  field={} value={}", field.name(), value)
    }

    fn record_error(
        &mut self,
        field: &tracing::field::Field,
        value: &(dyn std::error::Error + 'static),
    ) {
        println!("  field={} value={}", field.name(), value)
    }

    fn record_debug(&mut self, field: &tracing::field::Field, value: &dyn std::fmt::Debug) {
        println!("  field={} value={:?}", field.name(), value)
    }
}
```

<figcaption><a href="https://github.com/bryanburgers/tracing-blog-post/blob/main/examples/figure_2/custom_layer.rs">examples/figure_2/custom_layer.rs</a></figcaption>
</figure>

In our event handler, we use this new visitor with `event.record(&mut visitor)` to visit all of the values.

<figure>

```rust
fn on_event(
    &self,
    event: &tracing::Event<'_>,
    _ctx: tracing_subscriber::layer::Context<'_, S>,
) {
    println!("Got event!");
    println!("  level={:?}", event.metadata().level());
    println!("  target={:?}", event.metadata().target());
    println!("  name={:?}", event.metadata().name());
    let mut visitor = PrintlnVisitor;
    event.record(&mut visitor);
}
```

<figcaption><a href="https://github.com/bryanburgers/tracing-blog-post/blob/main/examples/figure_2/custom_layer.rs">examples/figure_2/custom_layer.rs</a></figcaption>
</figure>

Let's cross our fingers and see what happens when we run the program.

<figure>

```
Got event!
  level=Level(Info)
  target="figure_2"
  name="event examples/figure_2/main.rs:10"
  field=a_bool value=true
  field=answer value=42
  field=message value=first example
```

<figcaption>

`cargo run --example figure_2`

</figcaption>
</figure>

Oh boy! [Now we're cooking with Crisco!][crisco]


## Can we finally build a JSON logger?

Enough `println!` exploration. We have enough to make this look like a real JSON logger.

Instead of a `PrintlnVisitor`, let's create a `JsonVisitor` that can build up a JSON object.

<figure>

```rust
struct JsonVisitor<'a>(&'a mut BTreeMap<String, serde_json::Value>);

impl<'a> tracing::field::Visit for JsonVisitor<'a> {
    fn record_f64(&mut self, field: &tracing::field::Field, value: f64) {
        self.0
            .insert(field.name().to_string(), serde_json::json!(value));
    }

    // ...
}
```

<figcaption><a href="https://github.com/bryanburgers/tracing-blog-post/blob/main/examples/figure_3/custom_layer.rs">examples/figure_3/custom_layer.rs</a></figcaption>
</figure>

And in our event handler, let's build up a JSON object and print it at the end.

<figure>

```rust
 fn on_event(
    &self,
    event: &tracing::Event<'_>,
    _ctx: tracing_subscriber::layer::Context<'_, S>,
) {
    // Covert the values into a JSON object
    let mut fields = BTreeMap::new();
    let mut visitor = JsonVisitor(&mut fields);
    event.record(&mut visitor);

    // Output the event in JSON
    let output = serde_json::json!({
        "target": event.metadata().target(),
        "name": event.metadata().name(),
        "level": format!("{:?}", event.metadata().level()),
        "fields": fields,
    });
    println!("{}", serde_json::to_string_pretty(&output).unwrap());
}
```

<figcaption><a href="https://github.com/bryanburgers/tracing-blog-post/blob/main/examples/figure_3/custom_layer.rs">examples/figure_3/custom_layer.rs</a></figcaption>
</figure>

Running it gives us...

<figure>

```
{
  "fields": {
    "a_bool": true,
    "answer": 42,
    "message": "first example"
  },
  "level": "Level(Info)",
  "name": "event examples/figure_3/main.rs:10",
  "target": "figure_3"
}
```

<figcaption>

`cargo run --example figure_3`

</figcaption>
</figure>

And there we have it. We have a [`Layer`] that we can add to [`tracing-subscriber`] that will log structured events.  In the next article, we'll look at how we can use [`tracing`]'s *period-of-time* spans to provide even more rich context information to our logs.

[`tracing`]: https://docs.rs/tracing/0.1
[`tracing-subscriber`]: https://docs.rs/tracing-subscriber/0.2
[`serde_json`]: https://docs.rs/serde_json/1
[`log`]: https://docs.rs/log
[`Layer`]: https://docs.rs/tracing-subscriber/0.2/tracing_subscriber/layer/trait.Layer.html
[`Event`]: https://docs.rs/tracing/0.1/tracing/event/struct.Event.html
[`Visit`]: https://docs.rs/tracing/0.1/tracing/field/trait.Visit.html
[`target`]: https://docs.rs/tracing/0.1/tracing/struct.Metadata.html#method.target
[event-macros]: https://docs.rs/tracing/0.1.29/tracing/index.html#using-the-macros
[`on_event`]: https://docs.rs/tracing-subscriber/0.2/tracing_subscriber/layer/trait.Layer.html#method.on_event
[crisco]: https://www.youtube.com/watch?v=nVtwrlyGpiw&t=72
[`SpanRef`]: https://docs.rs/tracing-subscriber/0.2/tracing_subscriber/registry/struct.SpanRef.html
[span]: https://docs.rs/tracing/0.1/tracing/span
[Zenlist]: https://zenlist.com
[`Event::fields()`]: https://docs.rs/tracing-core/0.1/tracing_core/event/struct.Event.html#method.fields
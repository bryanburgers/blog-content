In the [previous article][previous], we looked at how to build a custom [`Layer`] that we can use to log events. But we missed a huge part of the [`tracing`] ecosystem: **spans**.

> To record the flow of execution through a program, `tracing` introduces the concept of [spans][span]. Unlike a log line that represents a *moment in time*, a span represents a *period of time* with a beginning and an end. When a program begins executing in a context or performing a unit of work, it *enters* that context’s span, and when it stops executing in that context, it *exits* the span.
>
> – The `tracing` docs

Let's see if we can add some of this context to our log output.


## More groundwork

If we're gonna do some exploration around spans, we need a few spans. Let's create an outer span and an inner span. Conceptually, the spans and the event create something nested like

* Outer span is entered
  * Inner span is entered
    * Event is created with the inner span as the parent and the outer span as the grandparent
  * Inner span is exited
* Outer span is exited

So let's update our `main.rs`. [`span.enter()`][`Span::enter`] uses the guard pattern to keep the span entered until it goes out of scope, and then exit it. So in our main, we exit inner and then outer in order at the end of the function.

<figure>

```rust
use tracing::{debug_span, info, info_span};
use tracing_subscriber::prelude::*;

mod custom_layer;
use custom_layer::CustomLayer;

fn main() {
    tracing_subscriber::registry().with(CustomLayer).init();

    let outer_span = info_span!("outer", level = 0);
    let _outer_entered = outer_span.enter();

    let inner_span = debug_span!("inner", level = 1);
    let _inner_entered = inner_span.enter();

    info!(a_bool = true, answer = 42, message = "first example");
}
```

<figcaption><a href="https://github.com/bryanburgers/tracing-blog-post/blob/main/examples/figure_5/main.rs">examples/figure_5/main.rs</a></figcaption>
</figure>

Back in our event handler, we're able to get this context. [`ctx.event_span(event)`][`Context::event_span`] will give us the event's parent span, if one exists. We could use that, but there's something even better: [`ctx.event_scope(event)`][`Context::event_scope`] will give an iterator of all of the spans.

In our case, it would give us the "inner span" first, then the "outer span".

And if we want them the other way around, [`scope.from_root()`][`Scope::from_root`] will flip them around and give us spans from the outside in. "Outer span" then "inner span" in our case.

Oh by the way, to use `ctx.event_scope()`, our subscriber needs to implement `LookupRef`. This ends up with a gnarly existential trait bound that – if I'm being 100% honest – I don't completely understand.

<figure>

```rust
impl<S> Layer<S> for CustomLayer
where
    S: tracing::Subscriber,
    // Scary! But there's no need to even understand it. We just need it.
    S: for<'lookup> tracing_subscriber::registry::LookupSpan<'lookup>,
{
    fn on_event(&self, event: &tracing::Event<'_>, ctx: tracing_subscriber::layer::Context<'_, S>) {
        // What's the parent span look like?
        let parent_span = ctx.event_span(event).unwrap();
        println!("parent span");
        println!("  name={}", parent_span.name());
        println!("  target={}", parent_span.metadata().target());

        println!();

        // What's about all of the spans that are in scope?
        let scope = ctx.event_scope(event).unwrap();
        for span in scope.from_root() {
            println!("an ancestor span");
            println!("  name={}", span.name());
            println!("  target={}", span.metadata().target());
        }
    }
}
```

<figcaption><a href="https://github.com/bryanburgers/tracing-blog-post/blob/main/examples/figure_5/custom_layer.rs">examples/figure_5/custom_layer.rs</a></figcaption>
</figure>

Alright, let's see what we get with a `cargo run`.

<figure>

```
parent span
  name=inner
  target=figure_5

an ancestor span
  name=outer
  target=figure_5
an ancestor span
  name=inner
  target=figure_5
```

<figcaption>

`cargo run --example figure_5`

</figcaption>
</figure>

OK, we can work with this. But just like in the [first article][previous], we don't have any field data. And it's the field data that really makes the context useful.

Of course, we remember visitors from the first article, so this should be easy, right? ***Right?***


## Where's the span data?

Wrong.

Well, we know about visitors. But the [things][`SpanRef`] we get back from `ctx.event_scope` don't have any way to visit their fields. What's up?

You probably remember *why* we had to use a visitor to get event data: because [`tracing`] never holds on to the field data. And if it doesn't hold on to the data, we probably shouldn't expect it to still have the data for a span that could have been entered a long, long time ago.

But not all is lost. It's time to start exploring some of [`Layer`]'s other trait methods. Specifically, let's look at `new_span` which is called when – as you probably guessed by the name – a new span is created.

<figure>

```rust
impl<S> Layer<S> for CustomLayer
where
    S: tracing::Subscriber,
    S: for<'lookup> tracing_subscriber::registry::LookupSpan<'lookup>,
{
    fn new_span(
        &self,
        attrs: &tracing::span::Attributes<'_>,
        id: &tracing::span::Id,
        ctx: tracing_subscriber::layer::Context<'_, S>,
    ) {
        let span = ctx.span(id).unwrap();
        println!("Got new_span!");
        println!("  level={:?}", span.metadata().level());
        println!("  target={:?}", span.metadata().target());
        println!("  name={:?}", span.metadata().name());

        // Our old friend, `println!` exploration.
        let mut visitor = PrintlnVisitor;
        attrs.record(&mut visitor);
    }
}
```

<figcaption><a href="https://github.com/bryanburgers/tracing-blog-post/blob/main/examples/figure_6/custom_layer.rs">examples/figure_6/custom_layer.rs</a></figcaption>
</figure>

<figure>

```
Got new_span!
  level=Level(Info)
  target="figure_7"
  name="outer"
  field=level value=0
Got new_span!
  level=Level(Debug)
  target="figure_7"
  name="inner"
  field=level value=1
```

<figcaption>

`cargo run --example figure_6`

</figcaption>
</figure>

There's our data. Now here's the kicker: we only get one chance to visit the data from a span – when it's created. [`tracing`] does not keep it around for us to access later. And that's fine if we're logging spans. Not so great if we're trying to get the span data later when we are logging an event, because the span data no longer exists.

If `tracing` doesn't store the data for us, what's a poor developer to do?


## Storing span data ourselves

What we really want to do is capture the span data when we see it, then store it somewhere so we can get at it later.

While `tracing` patently refuses to store the data itself, `tracing-subscriber` makes it very easy to store the data ourselves. It calls this "extensions", and every span we see has a way to attach extensions onto it.

We could technically store the `BTreeMap<String, serde_json::Value>` from our previous article in the extensions. But because the extension data is shared by all layers in the registry, it's considered good form to only store private types that we created ourselves. So we'll create a newtype wrapper around it.

<figure>

```rust
#[derive(Debug)]
struct CustomFieldStorage(BTreeMap<String, serde_json::Value>);
```

<figcaption><a href="https://github.com/bryanburgers/tracing-blog-post/blob/main/examples/figure_8/custom_layer.rs">examples/figure_8/custom_layer.rs</a></figcaption>
</figure>

Any time we see a new span, we build our JSON field object and we store it in the extension data.

<figure>

```rust
fn new_span(
    &self,
    attrs: &tracing::span::Attributes<'_>,
    id: &tracing::span::Id,
    ctx: tracing_subscriber::layer::Context<'_, S>,
) {
    // Build our json object from the field values like we have been
    let mut fields = BTreeMap::new();
    let mut visitor = JsonVisitor(&mut fields);
    attrs.record(&mut visitor);

    // And stuff it in our newtype.
    let storage = CustomFieldStorage(fields);

    // Get a reference to the internal span data
    let span = ctx.span(id).unwrap();
    // Get the special place where tracing stores custom data
    let mut extensions = span.extensions_mut();
    // And store our data
    extensions.insert::<CustomFieldStorage>(storage);
}
```

<figcaption><a href="https://github.com/bryanburgers/tracing-blog-post/blob/main/examples/figure_8/custom_layer.rs">examples/figure_8/custom_layer.rs</a></figcaption>
</figure>

Now, anywhere we have a span (like in our `on_event` method), we can get the data that we stored.

<figure>

```rust
// Get the special place where tracing stores custom data
let extensions = span.extensions();
// And get the custom data we stored out of it
let storage: &CustomFieldStorage = extensions.get::<CustomFieldStorage>().unwrap();
// And because `CustomFieldStorage` is just a newtype around our JSON object, we
// can get the JSON object that we stored.
let field_data: &BTreeMap<String, serde_json::Value> = &storage.0;
```

<figcaption><a href="https://github.com/bryanburgers/tracing-blog-post/blob/main/examples/figure_8/custom_layer.rs">examples/figure_8/custom_layer.rs</a></figcaption>
</figure>


## An almost fully functional JSON logger

OK. So we take what we've learned so far, mix it up with the JSON logger that we already have, and we should be able to add contextual span data to our logger. Easy peasy.

<figure>

```rust
fn on_event(&self, event: &tracing::Event<'_>, ctx: tracing_subscriber::layer::Context<'_, S>) {
    // All of the span context
    let scope = ctx.event_scope(event).unwrap();
    let mut spans = vec![];
    for span in scope.from_root() {
        let extensions = span.extensions();
        let storage = extensions.get::<CustomFieldStorage>().unwrap();
        let field_data: &BTreeMap<String, serde_json::Value> = &storage.0;
        spans.push(serde_json::json!({
            "target": span.metadata().target(),
            "name": span.name(),
            "level": format!("{:?}", span.metadata().level()),
            "fields": field_data,
        }));
    }

    // The fields of the event
    let mut fields = BTreeMap::new();
    let mut visitor = JsonVisitor(&mut fields);
    event.record(&mut visitor);

    // And create our output
    let output = serde_json::json!({
        "target": event.metadata().target(),
        "name": event.metadata().name(),
        "level": format!("{:?}", event.metadata().level()),
        "fields": fields,
        "spans": spans,
    });
    println!("{}", serde_json::to_string_pretty(&output).unwrap());
}
```

<figcaption><a href="https://github.com/bryanburgers/tracing-blog-post/blob/main/examples/figure_9/custom_layer.rs">examples/figure_9/custom_layer.rs</a></figcaption>
</figure>

I mean, let's run it just to be sure.

<figure>

```
{
  "fields": {
    "a_bool": true,
    "answer": 42,
    "message": "first example"
  },
  "level": "Level(Info)",
  "name": "event examples/figure_9/main.rs:16",
  "spans": [
    {
      "fields": {
        "level": 0
      },
      "level": "Level(Info)",
      "name": "outer",
      "target": "figure_9"
    },
    {
      "fields": {
        "level": 1
      },
      "level": "Level(Debug)",
      "name": "inner",
      "target": "figure_9"
    }
  ],
  "target": "figure_9"
}
```

<figcaption>

`cargo run --example figure_9`

</figcaption>
</figure>

Ha, perfect! Ship it, squirrel.


## Wait, did you say *almost* fully functional?

When I did this for my work, I did ship it. And it worked great. But then I discovered that to get a **fully** functional logger, there's one more small step. And I do mean small, I promise.

The reason this isn't already fully functional is because spans can record data *after* they're created. And by that, I mean that this is possible.

<figure>

```rust
let outer_span = info_span!("outer", level = 0, other_field = tracing::field::Empty);
let _outer_entered = outer_span.enter();
// Some code...
outer_span.record("other_field", &7);
```

<figcaption><a href="https://github.com/bryanburgers/tracing-blog-post/blob/main/examples/figure_10/main.rs">examples/figure_10/main.rs</a></figcaption>
</figure>

And if we would run our logger as-is right now, we would miss the `"other_field"` because it didn't exist when we received our `new_span` event.

But now that we've dug into how layers and spans and extensions work, this shouldn't even phase us anymore. The only thing we really need to know is that [`Layer`] also has an `on_record` method for this situation. Everything else is a variation of the work we've already done.

<figure>

```rust
fn on_record(
    &self,
    id: &tracing::span::Id,
    values: &tracing::span::Record<'_>,
    ctx: tracing_subscriber::layer::Context<'_, S>,
) {
    // Get the span whose data is being recorded
    let span = ctx.span(id).unwrap();

    // Get a mutable reference to the data we created in new_span
    let mut extensions_mut = span.extensions_mut();
    let custom_field_storage: &mut CustomFieldStorage =
        extensions_mut.get_mut::<CustomFieldStorage>().unwrap();
    let json_data: &mut BTreeMap<String, serde_json::Value> = &mut custom_field_storage.0;

    // And add to using our old friend the visitor!
    let mut visitor = JsonVisitor(json_data);
    values.record(&mut visitor);
}
```

<figcaption><a href="https://github.com/bryanburgers/tracing-blog-post/blob/main/examples/figure_10/custom_layer.rs">examples/figure_10/custom_layer.rs</a></figcaption>
</figure>

And now, two articles later, we have a fully functioning custom JSON logger. Nice work, team.

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
[`Span::enter`]: https://docs.rs/tracing/0.1.29/tracing/span/struct.Span.html#method.enter
[`SpanRef`]: https://docs.rs/tracing-subscriber/0.2/tracing_subscriber/registry/struct.SpanRef.html
[span]: https://docs.rs/tracing/0.1/tracing/span
[previous]: /custom-logging-in-rust-using-tracing
[`Context::event_scope`]: https://docs.rs/tracing-subscriber/0.2.25/tracing_subscriber/layer/struct.Context.html#method.event_scope
[`Context::event_span`]: https://docs.rs/tracing-subscriber/0.2.25/tracing_subscriber/layer/struct.Context.html#method.event_span
[`Scope::from_root`]: https://docs.rs/tracing-subscriber/0.2.25/tracing_subscriber/registry/struct.Scope.html#method.from_root
**Please note:** This framework is in active development. I'm keeping it in a cycle of 0.0.x releases at the moment to indicate that it’s not even ready for its 0.1.0. Active work is being done on documentation and features, and APIs should not necessarily be considered stable. At the same time, it is more than a toy project or proof of concept, and I am actively using it for my own application development.

<img src="https://raw.githubusercontent.com/gbj/leptos/main/docs/logos/logo.svg" alt="Leptos Logo" style="width: 100%; height: auto; display: block; margin: auto;">

[![crates.io](https://img.shields.io/crates/v/leptos.svg)](https://crates.io/crates/leptos)
[![docs.rs](https://docs.rs/leptos/badge.svg)](https://docs.rs/leptos)
[![Discord](https://img.shields.io/discord/1031524867910148188?color=%237289DA&label=discord)](https://discord.gg/YdRAhS7eQB)

# Leptos

```rust
use leptos::*;

#[component]
pub fn SimpleCounter(cx: Scope, initial_value: i32) -> Element {
    // create a reactive signal with the initial value
    let (value, set_value) = create_signal(cx, initial_value);

    // create event handlers for our buttons
    // note that `value` and `set_value` are `Copy`, so it's super easy to move them into closures
    let clear = move |_| set_value(0);
    let decrement = move |_| set_value.update(|value| *value -= 1);
    let increment = move |_| set_value.update(|value| *value += 1);

    // this JSX is compiled to an HTML template string for performance
    view! {
        cx,
        <div>
            <button on:click=clear>"Clear"</button>
            <button on:click=decrement>"-1"</button>
            <span>"Value: " {move || value().to_string()} "!"</span>
            <button on:click=increment>"+1"</button>
        </div>
    }
}

// Easy to use with Trunk (trunkrs.dev) or with a simple wasm-bindgen setup
pub fn main() {
    mount_to_body(|cx| view! { cx,  <SimpleCounter initial_value=3 /> })
}

```

## About the Framework

Leptos is a full-stack, isomorphic Rust web framework leveraging fine-grained reactivity to build declarative user interfaces.

## What does that mean?

- **Full-stack**: Leptos can be used to build apps that run in the browser (_client-side rendering_), on the server (_server-side rendering_), or by rendering HTML on the server and then adding interactivity in the browser (_hydration_). This includes support for _HTTP streaming_ of both data (`Resource`s) and HTML (out-of-order streaming of `<Suspense/>` components.)
- **Universal**: The same application code and business logic are compiled to run on the client and server, with seamless integration. You can write your server-only logic (database requests, authentication etc.) alongside the client-side components that will consume it, and let Leptos manage the data loading without the need to manually create APIs to consume.
- **Web**: Leptos is built on the Web platform and Web standards. Whenever possible, we use Web essentials (like links and forms) and build on top of them rather than trying to replace them.
- **Framework**: Leptos provides most of what you need to build a modern web app: a reactive system, templating library, and a router that works on both the server and client side.
- **Fine-grained reactivity**: The entire framework is build from reactive primitives. This allows for extremely performant code with minimal overhead: when a reactive signal’s value changes, it can update a single text node, toggle a single class, or remove an element from the DOM without any other code running. (_So, no virtual DOM!_)
- **Declarative**: Tell Leptos how you want the page to look, and let the framework tell the browser how to do it.

## Learn more

Here are some resources for learning more about Leptos:

- [Examples](https://github.com/gbj/leptos/tree/main/examples)
- [API Documentation](https://docs.rs/leptos/latest/leptos/) (in progress)
- Leptos Guide (in progress)

## `nightly` Note

Most of the examples assume you’re using `nightly` Rust. If you’re on stable, note the following:

1. You need to enable the `"stable"` flag in `Cargo.toml`: `leptos = { version = "0.0", features = ["stable"] }`
2. `nightly` enables the function call syntax for accessing and setting signals. If you’re using `stable`,
   you’ll just call `.get()`, `.set()`, or `.update()` manually. Check out the
   [`counters-stable` example](https://github.com/gbj/leptos/blob/main/examples/counters-stable/src/main.rs)
   for examples of the correct API.

## Benchmarks

### Server-Side Rendering

I’ve created a benchmark comparing Leptos’s HTML rendering on the server to [Tera](https://github.com/Keats/tera), [Yew](https://github.com/yewstack/yew), and [Sycamore](https://github.com/sycamore-rs/sycamore). You can find the benchmark [here](https://github.com/gbj/leptos/tree/main/benchmarks) and run it yourself using `cargo bench`. Leptos renders HTML roughly as fast as Tera, and scales well as templates become larger. It's significantly faster than the server-side HTML rendering done by similar frameworks.

<details>
  <summary>Click to show results</summary>
<table>
<thead>
<tr><td><em>ns/iter</em></td><td>Tera</td><td>Leptos</td><td>Yew</td><td>Sycamore</td></tr>
</thead>
<tbody>
<tr><td>3 Counters</td><td align="right">3,454</td><td align="right">5,666</td><td align="right">34,984</td><td align="right">32,412</td></tr>
<tr><td>TodoMVC (no todos)</td><td align="right">2,396</td><td align="right">5,561</td><td align="right">38,725</td><td align="right">68,749</td></tr>
<tr><td>TodoMVC (1000 todos)</td><td align="right">3,829,447</td><td align="right">3,077,907</td><td align="right">5,125,639</td><td align="right">19,448,900</td></tr>
<tr><td><em>Average</em></td><td align="right">1.08</td><td align="right">1.65</td><td align="right">6.25</td><td align="right">9.36</td></tr>
</tbody>
</table>
</details>

### Client-Side Rendering

The gold standard for testing raw rendering performance for front-end web frameworks is the [js-framework-benchmark](https://github.com/krausest/js-framework-benchmark). The official results list Leptos as the fastest Rust/Wasm framework, slightly slower than SolidJS and significantly faster than popular JS frameworks like Svelte, Preact, and React.

<details>
  <summary>Click to show results</summary>
  <img width="913" alt="js-framework-benchmark results" src="https://user-images.githubusercontent.com/286622/198388168-d21e938b-5d59-4000-b373-91b48f1ec4d3.png">
</details>

## FAQs

### Can I use this for native GUI?

Sure! Obviously the `view` macro is for generating DOM nodes but you can use the reactive system to drive native any GUI toolkit that uses the same kind of object-oriented, event-callback-based framework as the DOM pretty easily. The principles are the same:
- Use signals, derived signals, and memos to create your reactive system
- Create GUI widgets
- Use event listeners to update signals
- Create effects to update the UI

I've put together a [very simple GTK example](https://github.com/gbj/leptos/blob/main/examples/gtk/src/main.rs) so you can see what I mean.

### How is this different from Yew/Dioxus?

On the surface level, these libraries may seem similar. Yew is, of course, the most mature Rust library for web UI development and has a huge ecosystem. Dioxus is similar in many ways, being heavily inspired by React. Here are some conceptual differences between Leptos and these frameworks:

- **VDOM vs. fine-grained:** Yew is built on the virtual DOM (VDOM) model: state changes cause components to re-render, generating a new virtual DOM tree. Yew diffs this against the previous VDOM, and applies those patches to the actual DOM. Component functions rerun whenever state changes. Leptos takes an entirely different approach. Components run once, creating (and returning) actual DOM nodes and setting up a reactive system to update those DOM nodes.
- **Performance:** This has huge performance implications: Leptos is simply _much_ faster at both creating and updating the UI than Yew is.
- **Mental model:** Adopting fine-grained reactivity also tends to simplify the mental model. There are no surprising components re-renders because there are no re-renders. Your app can be divided into components based on what makes sense for your app, because they have no performance implications.

### How is this different from Sycamore?

Conceptually, these two frameworks are very similar: because both are built on fine-grained reactivity, most apps will end up looking very similar between the two, and Sycamore or Leptos apps will both look a lot like SolidJS apps, in the same way that Yew or Dioxus can look a lot like React.

There are some practical differences that make a significant difference:

- **Maturity:** Sycamore is obviously a much more mature and stable library with a larger ecosystem.
- **Templating:** Leptos uses a JSX-like template format (built on [syn-rsx](https://github.com/stoically/syn-rsx)) for its `view` macro. Sycamore offers the choice of its own templating DSL or a builder syntax.
- **Template node cloning:** Leptos's `view` macro compiles to a static HTML string and a set of instructions of how to assign its reactive values. This means that at runtime, Leptos can clone a `<template>` node rather than calling `document.createElement()` to create DOM nodes. This is a _significantly_ faster way of rendering components.
- **Read-write segregation:** Leptos, like Solid, encourages read-write segregation between signal getters and setters, so you end up accessing signals with tuples like `let (count, set_count) = create_signal(cx, 0);` *(If you prefer or if it's more convenient for your API, you can use `create_rw_signal` to give a unified read/write signal.)*
- **Signals are functions:** In Leptos, you can call a signal to access it rather than calling a specific method (so, `count()` instead of `count.get()`) This creates a more consistent mental model: accessing a reactive value is always a matter of calling a function. For example:

```rust
let (count, set_count) = create_signal(cx, 0); // a signal
let double_count = move || count() * 2; // a derived signal
let memoized_count = create_memo(cx, move |_| count() * 3); // a memo
// all are accessed by calling them
assert_eq!(count(), 0);
assert_eq!(double_count(), 0);
assert_eq!(memoized_count(), 0);

// this function can accept any of those signals
fn do_work_on_signal(my_signal: impl Fn() -> i32) { ... }
```

- **Signals and scopes are `'static`:** Both Leptos and Sycamore ease the pain of moving signals in closures (in particular, event listeners) by making them `Copy`, to avoid the `{ let count = count.clone(); move |_| ... }` that's very familiar in Rust UI code. Sycamore does this by using bump allocation to tie the lifetimes of its signals to its scopes: since references are `Copy`, `&'a Signal<T>` can be moved into a closure. Leptos does this by using arena allocation and passing around indices: types like `ReadSignal<T>`, `WriteSignal<T>`, and `Memo<T>` are actually wrapper for indices into an arena. This means that both scopes and signals are both `Copy` and `'static` in Leptos, which means that they can be moved easily into closures without adding lifetime complexity.

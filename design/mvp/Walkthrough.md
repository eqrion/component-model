# Component Model Walkthrough

This is walk through of the internals of the WebAssembly Component Model. It is an introduction to the [reference](./Explainer.md)

This is not intended to explain how to use components in a source language. For that, take a look at the **[Component Model Documentation]**.

## Problem

WebAssembly Components (hereafter 'components') intend to solve several problems.

### 1. WebAssembly has no standardized executable format

An 'executable' is a file which a computing platform (hereafter a 'host') can run by itself. There is no standardized executable format for WebAssembly, and this leads to fragmentation and difficulty for languages to provide first-class support for WebAssembly hosts.

To be an executable for a computing platform, at minimum you need to provide the following:
  1. How do I link and load the initial code?
  2. How does the code use APIs provided by the host?

The [WebAssembly specification](https://webassembly.github.io/spec/) defines a binary format for a wasm module. A wasm module is close to being an executable by providing #1 above. However, #2 is left to be defined per-host. A wasm module imports functionality from the host as functions, and those functions only have primitive scalar types (like `i32`). Hosts therefore each need to define their own ABI for passing and receiving source language types to APIs.

On the web, there is no standardized ABI and instead toolchains are expected to define their own ABI and emit JS ('glue') code to emulate a host that implements an API. For example, Rust compiled to wasm for the web could pass a string to a web API by invoking an import and passing `i32` values to delimit the span of the string in their linear memory, and then rely on the JS glue code to construct a JS string by reading the wasm modules linear memory using that span.

A wasm module alone is not an executable for the web, and needs JS glue code to be runnable.
TODO: maybe link to mozilla hacks blog post on this

Outside the web, the initial WASI preview 1 manually defined an ABI. TODO: verify and figure out what to say.

There are three problems with this:
  1. Toolchains don't want to provide
  2. Performance
  3. 

### 2. Existing software composition is too limiting

Composition is when you combine two pieces of separately written software into something new.

You either compose via:
  * Composition through an ABI
    - Pros:
        * Can be lightweight and fast
        * Can use most language features for expressive API's
    - Cons:
        * Generally can only combine within very small languages of families
        * Faults are catastrophic and not self-contained
  * Composition through serialization across a process boundary
    - Pros:
        * Can compose nearly any language
        * Faults can be isolated to a single component
    - Cons:
        * Serialization has a performance impact
        * Process isolation has perf/memory impact
        * Difficult to express synchronous API's
        * Can only express API's that fit within serialization format

## What can we do?

The above problems overlap significantly and so we should consider if there is a common solution.

"Running a program is composing it with a host" A self-contained executable needs a language-netural typed interface to its host. Cross-language composition needs the same. This is actually the same problem.

"A software component is a unit of composition with contractually specified interfaces and explicit context dependencies only. A software component can be deployed independently and is subject to composition by third parties"
Clemens Szyperski: https://dl.acm.org/doi/10.5555/515228
TODO: verify the quote

## What is a WebAssembly Component?

To solve the above problems, this proposal defines the WebAssembly Component Model.

A WebAssembly component bundles WebAssembly modules together with higher-level interface definitions. This higher-level type information standardizes the ad-hoc practices performed in WebAssembly bindings (or "glue code") today, and enables the host (or other components) to automatically link and load WebAssembly code.

```
┌────────────────────────────────────┬──────────────────────────────────────────────────────────────────┐
│               Clause               │                            Mechanism                             │
├────────────────────────────────────┼──────────────────────────────────────────────────────────────────┤
│ unit of composition                │ the component binary                                             │
├────────────────────────────────────┼──────────────────────────────────────────────────────────────────┤
│ contractually specified interfaces │ typed imports/exports (component types, WIT)                     │
├────────────────────────────────────┼──────────────────────────────────────────────────────────────────┤
│ explicit context dependencies only │ everything is imported, no ambient authority, shared-nothing     │
├────────────────────────────────────┼──────────────────────────────────────────────────────────────────┤
│ deployed independently             │ self-contained binary, no toolchain glue                         │
├────────────────────────────────────┼──────────────────────────────────────────────────────────────────┤
│ composition by third parties       │ linking by instantiating and wiring, without source or relinking │
└────────────────────────────────────┴──────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────┬─────────────────────────────────────────────────────────────────────────┐
│                 Con                 │                                 Answer                                  │
├─────────────────────────────────────┼─────────────────────────────────────────────────────────────────────────┤
│ Only works within a language family │ language-neutral value types                                            │
├─────────────────────────────────────┼─────────────────────────────────────────────────────────────────────────┤
│ Serialization cost                  │ Canonical ABI copies memory-to-memory once, with no intermediate format │
├─────────────────────────────────────┼─────────────────────────────────────────────────────────────────────────┤
│ Process overhead                    │ isolation through separate linear memories in one process               │
├─────────────────────────────────────┼─────────────────────────────────────────────────────────────────────────┤
│ Synchronous APIs are hard           │ a synchronous call is just a call                                       │
├─────────────────────────────────────┼─────────────────────────────────────────────────────────────────────────┤
│ Only serializable types             │ resources (handles), plus futures and streams                           │
└─────────────────────────────────────┴─────────────────────────────────────────────────────────────────────────┘
```

## Why a new layer?

TODO.

## Examples

### Empty component

### Component that re-exports

### Instantiating a module

Can be nested module or an import.

### Exporting a function

### Lifting and lowering of value types

High level description of the ABI for lift/lower of value types.
Excluding resources and async.

### Importing a resource type

Own and borrow.
First importing functions without [method][constructor]

### Special labelname

Description of [method][constructor][getter][setter] function imports.

### Defining and exporting a resource type

resource.new
resource.rep

### Nesting a component

Defining and instantiating a component.

### Generativity of resource types

Explanation of how imported resource types work with nested components.

### Lowering a function as async

### Lifting a function as async

### Using futures

### Using streams

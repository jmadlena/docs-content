# Core

At the heart of the FormKit framework is `@formkit/core`. This zero-dependency package is responsible for nearly all of FormKit's low-level critical functions, such as:

- Configuration
- Value input/output
- Event bubbling
- Plugin management
- Tree state tracking
- Message management
- Lifecycle hooks

## Architecture

The functionality of FormKit core is not exposed to your application via a centralized instance but rather a distributed set of "nodes" (`FormKitNode`) where each node represents a single input.

This mirrors HTML — in fact DOM structure is actually a [general tree](https://opendsa-server.cs.vt.edu/ODSA/Books/Everything/html/GenTreeIntro.html) and FormKit core nodes reflect this structure. For example, a simple login form could be drawn as the following tree graph:

<figure>
  <simple-tree></simple-tree>
  <figcaption>Hover over each node to see its initial options.</figcaption>
</figure>

In this diagram, a `form` node is a parent to three child nodes — `email`, `password` and `submit`. Each input component in the graph "owns" a FormKit core node, and each node contains its own options, configuration, props, events, plugins, lifecycle hooks, etc. This architecture ensures that FormKit’s primary features are decoupled from the rendering framework (Vue) — a key to reducing side effects and maintaining blazing fast performance.

Additionally, this decentralized architecture allows for tremendous flexibility. For example — one form could use different plugins than other forms in an application, a group input could modify the configuration of its sub-inputs, and validation rules can even be written to use props from another input.

## Node

At the heart of `@formkit/core` is the node. Every `<FormKit>` component owns a single node, and each node must be one of three types:

- [Input](#input)
- [List](#list)
- [Group](#group)

<callout type="tip" label="Input vs node types">
Core nodes are always one of three types (input, list, or group). These are not the same as input types — of which there can be unlimited variation. Strictly speaking all inputs have 2 types: their node type (like <code>input</code>), and their input type (like <code>checkbox</code>).
</callout>

### Input

Most of FormKit’s native inputs have a node type of `input` — they operate on a single value. The value itself can be of any type, such as objects, arrays, strings, and numbers — any value is acceptable. However, nodes of type `input` are always leafs — they cannot have children.

```js
import { createNode } from '@formkit/core'

const input = createNode({
  type: 'input', // defaults to 'input' if not specified
  value: 'hello node world',
})

console.log(input.value)
// 'hello node world'
```

### List

A list is a node that produces an array value. Children of a list node produce a value in the list’s array value. The names of immediate children are ignored — instead each is assigned an index in the list’s array.

```js
import { createNode } from '@formkit/core'

const list = createNode({
  type: 'list',
  children: [
    createNode({ value: 'paprika@example.com' }),
    createNode({ value: 'bill@example.com' }),
    createNode({ value: 'jenny@example.com' }),
  ],
})

console.log(list.value)
// ['paprika@example.com', 'bill@example.com', 'jenny@example.com']
```

### Group

A group is a node that produces an object value. Children of a group node use their `name` to produce a property of the same name in the groups’s value object — `<FormKit type="form">` is an instance of a group.

```js
import { createNode } from '@formkit/core'

const group = createNode({
  type: 'group',
  children: [
    createNode({ name: 'meat', value: 'turkey' }),
    createNode({ name: 'greens', value: 'salad' }),
    createNode({ name: 'sweets', value: 'pie' }),
  ],
})

console.log(group.value)
// { meat: 'turkey', greens: 'salad', sweets: 'pie' }
```

### Options

In addition to specifying the `type` of node when calling `createNode()`, you can pass any of the following options:

| Options  | Default      | Description                                                                    |
| -------- | ------------ | ------------------------------------------------------------------------------ |
| children | `[]`         | Child `FormKitNode` instances.                                                 |
| config   | `{}`         | Configuration options. These become the defaults of the `props` object.        |
| name     | `{type}_{n}` | The name of the node/input.                                                    |
| parent   | `null`       | The parent `FormKitNode` instance.                                             |
| plugins  | `[]`         | An array of plugin functions.                                                  |
| props    | `{}`         | An object of key/value pairs that represent the current node instance details. |
| type     | `input`      | The type of `FormKitNode` to create (`list`, `group`, or `input`).             |
| value    | `undefined`  | The initial value of the input.                                                |

### Config & Props

FormKit uses an inheritance-based configuration system. Any values declared in the `config` option are automatically passed to children (and all descendants) of that node, but not passed to siblings or parents. Each node can override its inherited values by providing its own config, and these values will in turn be inherited by any deeper children and descendants. For example:

```js
const parent = createNode({
  type: 'group',
  config: {
    color: 'yellow',
  },
  children: [
    createNode({
      type: 'list',
      config: { color: 'pink' },
      children: [createNode(), createNode()],
    }),
    createNode(),
  ],
})
```

The above code will result in each node having the following configuration:

<figure>
  <config-tree></config-tree>
  <figcaption>Notice how the list subtree is pink.</figcaption>
</figure>

<callout type="tip" label="Use props to read config">
It is best practice to read configuration values from <code>node.props</code> rather than <code>node.config</code>. The next section details this feature.
</callout>

### Props

The `node.props` and `node.config` objects are closely related. `node.config` is best thought of as the initial values for `node.props`. `props` is an arbitrarily shaped object that contains details about the current _instance_ of the node.

The best practice is to always read configuration and prop data from `node.props` even if the original value is defined using `node.config`. Explicitly defined props take precedence over configuration options.

```js
const child = createNode({
  props: {
    flavor: 'cherry',
  },
})
const parent = createNode({
  type: 'group',
  config: {
    size: 'large',
    flavor: 'grape',
  },
  children: [child],
})
console.log(child.props.size)
// outputs: 'large'
console.log(child.props.flavor)
// outputs: 'cherry'
```

<callout type="tip" label="FormKit component props">
When using the <code>&lt;FormKit&gt;</code> component, any props defined for the input <code>type</code> are automatically set as <code>node.props</code> properties. For example: <code>&lt;FormKit label="Email" /&gt;</code> would result in <code>node.props.label</code> being <code>Email</code>.
</callout>

### Setting values

You can set the initial value of a node by providing the `value` option on `createNode()` — but FormKit is all about interactivity, so how do we update the value of an already defined node? By using `node.input(value)`.

```js
import { createNode } from '@formkit/core'

const username = createNode()
username.input('jordan-goat98')
console.log(username.value)
// undefined  👀 wait — what!?
```

In the above example `username.value` is still undefined immediately after it’s set because `node.input()` is asynchronous. If you need to read the resulting value after calling `node.input()` you can await the returned promise.

```js
import { createNode } from '@formkit/core'

const username = createNode()
username.input('jordan-goat98').then(() => {
  console.log(username.value)
  // 'jordan-goat98'
})
```

Because `node.input()` is asynchronous, the rest of our form does not need to recompute its dependencies on every keystroke. It also provides an opportunity to perform modifications to the unsettled value before it is "committed" to the rest of the form. However — for internal node use only — a `_value` property containing the unsettled value of the input is also available.

<callout type="danger" label="Don’t assign values">
You cannot <em>directly</em> assign the value of an input <code>node.value = 'foo'</code>. Instead, you should always use <code>node.input(value)</code>
</callout>

### Value settlement

Now that we understand `node.input()` is asynchronous, let's explore how FormKit solves the "settled tree" problem. Imagine a user quickly types in their email address and hits "enter" very quickly — thus submitting the form. Since `node.input()` is asynchronous, incomplete data would likely be submitted. We need a mechanism to know when the whole form has "settled".

To solve this, FormKit’s nodes automatically track tree, subtree, and node "disturbance". This means the form (usually the root node) always knows the settlement state of all the inputs it contains.

The following graph illustrates this "disturbance counting". Click on any input node (blue) to simulate calling `node.input()` and notice how the whole form is always aware of how many nodes are "disturbed" at any given time. When the root node has a disturbed count of `0` the form is settled and safe to submit.

<figure>
  <disturbance-tree></disturbance-tree>
  <figcaption>Click on the inputs (blue) to simulate calling a user input.</figcaption>
</figure>
To ensure a given tree (form), subtree (group), or node (input) is "settled" you can await the `node.settled` property:

```js
import { createNode } from '@formkit/node'

const form = createNode({
  type: 'group',
  children: [
    createNode()
    createNode()
    createNode()
  ],
})
// ...
// user interaction:
async function someEvent () {
  await form.settled
  // we now know the form is fully "settled"
  // and that form.value is accurate.
}
```

<callout type="tip" label="The form type">
The <code>&lt;FormKit type="form"&gt;</code> input already incorporates this await behavior. It will not call your <code>@submit</code> handler until your form is completely settled. However when building advanced inputs it can be useful to understand these underlying principles.
</callout>

## Traversal

To traverse nodes within a group or list use `node.at(address)` — where `address` is the `name` of the node being accessed (or the relative path to the name). For example:

```js
import { createNode } from '@formkit/core'

const group = createNode({
  type: 'group',
  children: [createNode({ name: 'email' }), createNode({ name: 'password' })],
})

// Returns the email node
group.at('email')
```

If the starting node has siblings, it will attempt to locate a match in the siblings (internally, this is what FormKit uses for validation rules like `confirm:address`).

```js
import { createNode } from '@formkit/core'

const email = createNode({ name: 'email' })
const password = createNode({ name: 'password' })
const group = createNode({
  type: 'group',
  children: [email, password],
})

// Accesses sibling to return the password node
email.at('password')
```

### Deep traversal

You can go deeper than one level by using a dot-syntax relative path. Here's a more complex example:

```js
import { createNode } from '@formkit/core'

const group = createNode({
  type: 'group',
  children: [
    createNode({ name: 'team' }),
    createNode({
      type: 'list',
      name: 'users',
      children: [
        createNode({
          type: 'group',
          children: [
            createNode({ name: 'email' }),
            createNode({ name: 'password', value: 'foo' }),
          ],
        }),
        createNode({
          type: 'group',
          children: [
            createNode({ name: 'email' }),
            createNode({ name: 'password', value: 'fbar' }),
          ],
        }),
      ],
    }),
  ],
})

// outputs: 'foo'
console.log(group.at('users.0.password').value)
```

Notice how traversing the `list` uses numeric keys, this is because the `list` type uses array indexes automatically.

<figure>
  <traversal-tree></traversal-tree>
  <figcaption>Traversal path of <code>group.at('users.0.password')</code> shown in red.</figcaption>
</figure>

<callout type="tip" label="Array paths">
Node addresses may also be expressed as arrays. For example <code>node.at('foo.bar')</code> could be expressed as <code>node.at(['foo', 'bar'])</code>.
</callout>

### Traversal tokens

Also available for use in `node.at()` are a few special "tokens":

| Token     | Description                                                                                                                         |
| --------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| `$parent` | The immediate ancestor of the current node.                                                                                         |
| `$root`   | The root node of the tree (the first node with no parent).                                                                          |
| `$self`   | The current node in the traversal.                                                                                                  |
| `find()`  | A function that performs a breadth-first search for a matching value and property. For example: `node.at('$root.find(555, value)')` |

These tokens are used in dot-syntax addresses just like you would use a node’s name:

```js
import { createNode } from '@formkit/core'

const secondEmail = createNode({ name: 'email' })

createNode({
  type: 'group',
  children: [
    createNode({ name: 'team', value: 'charlie@factory.com' }),
    createNode({
      type: 'list',
      name: 'users',
      children: [
        createNode({
          type: 'group',
          children: [
            createNode({ name: 'email', value: 'james@peach.com' }),
            createNode({ name: 'password', value: 'foo' }),
          ],
        }),
        createNode({
          type: 'group',
          children: [
            secondEmail, // We're going to start here.
            createNode({ name: 'password', value: 'fbar' }),
          ],
        }),
      ],
    }),
  ],
})

// Navigate from the second email to the first
console.log(secondEmail.at('$parent.$parent.0.email').value)
// outputs: charlie@factory.com
```

<figure>
  <traversal-tree2></traversal-tree2>
  <figcaption>Traversal path of <code>secondEmail.at('$parent.$parent.0.email')</code> shown in red.</figcaption>
</figure>

## Hooks

Hooks are middleware dispatchers that are triggered during pre-defined lifecycle operations. These hooks allow external code to extend the internal functionality of `@formkit/core`. The following table details all available hooks:

| Hook    | Value                                                                                                   | Description                                                                                                       |
| ------- | ------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| classes | <pre><code class="block">{<br> property: string,<br> classes: Record<string, boolean><br>}</code></pre> | Dispatched after all class operations have been run, before final conversion to a string.                         |
| commit  | `any`                                                                                                   | Dispatched when setting the value of a node _after_ the `input` and debounce of `node.input()` is called.         |
| error   | `string`                                                                                                | Dispatched when processing a thrown error — errors are generally inputs, and the final output should be a string. |
| init    | `FormKitNode`                                                                                           | Dispatched after the node is initially created but before it is returned in `createNode()`.                       |
| input   | `any`                                                                                                   | Dispatched synchronously on every input event (every keystroke) before `commit`.                                  |
| prop    | <pre><code class="block">{<br> prop: string,<br> value: any<br>}</code></pre>                           | Dispatched when any prop is being assigned.                                                                       |
| text    | [`FormKitTextFragment`](https://github.com/formkit/formkit/search?q=FormKitTextFragment)                | Dispatched when a FormKit-generated string needs to be displayed — allowing i18n or other plugins to intercept.   |

### Hook middleware

To make use of these hooks, you must register hook middleware. A middleware is simply a function that accepts 2 arguments — the value of the hook and `next` — a function that calls the next middleware in the stack and returns the value.

To register a middleware, pass it to the `node.hook` you want to use:

```js
import { createNode } from '@formkit/core'

const node = createNode()

// This would transform all labels to "Different label!"
node.hook.prop((payload, next) => {
  if ((payload.prop = 'label')) {
    payload.value = 'Different label!'
  }
  return next(payload)
})
```

<callout type="tip" label="Use with plugins">
Hooks can be registered anywhere in your application, but the most common place hooks are used is in a plugin.
</callout>

## Plugins

Plugins are the primary mechanism for extending the functionality of FormKit. The concept is simple — a plugin is just a function that accepts a node. These functions are then automatically called when a node is created, or when the plugin is added to the node. Plugins work similar to configuration options — they are automatically inherited by children and descendants.

```js
import { createNode } from '@formkit/core'

// A plugin to change a prop value.
const myPlugin = (node) => {
  if (node.type === 'group') {
    node.props.color = 'yellow'
  } else {
    node.props.color = 'teal'
  }
}

const node = createNode([
  plugins: [myPlugin],
  children: [createNode()]
])
```

In the example above, the plugin is only defined on the parent, but the child also inherits the plugin. The function `myPlugin` will be called twice — once for each node in the graph (which only has two in this example):

<figure>
  <plugin-tree></plugin-tree>
  <figcaption>The plugin is inherited by the child, but executed independently.</figcaption>
</figure>
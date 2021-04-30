The Edge components system is highly inspired by the frontend frameworks like Vue or Svelte. It borrows the concept of **reusability**, **isolated state**, **props** and **slots** from them.

However, do note that Edge is a backend template engine and hence some of the principles cannot be replicated because of the server side environment and the design of Edge. It includes

- **Reactivity**: There is no concept of reactivity on backend. You just generate the HTML and send it over the wire.
- **Scoped CSS**: Edge is not compiled using frontend build tools like Webpack and hence it does not bother itself with compiling and extract CSS from components. You must use existing frontend tooling for that.

## Creating a component

Components are represented using the regular edge template files. For example: You can create a file named `button.edge` with the following markup

```edge
<button type="{{ type }}">
  {{ text }}
</button>
```

And then use it as a component inside other templates.

```edge
@!component('button', {
  text: 'Login',
  type: 'submit'
})
```

The `@component` tag accepts a total of two arguments. First is the component path (relative from the views directory) and second is the component state (aka props).

:::note

Components do not have access to the state of the parent template. However, they can access templates [globals]() and [locals]().

:::

## Slots

Along with the props, components can also accept `slots`. Slots are named outlets, that the parent component can define markup for.

As an example, lets accept the text for our `button` component as a slot and not as a prop.

```edge
<button type="{{ type }}">
  {{{ await $slots.main() }}}
</button>
```

The component caller can define the markup for the main slot as follows.

```edge
@component('button', {
  type: 'submit'
})
  <i class="fa-icon-lock" />
  <span> Login </span>
@end
```

### Main slot

All the contents between the `@component` opening and the closing tag is part of the main slot, unless you move it inside a named slot.

In the following example, all the content outside the `@slot('trigger')` is part of the main slot.

```edge
@component('modal')
  @slot('trigger')
    <a href=""> Show modal </a>
  @end

  <h1> Modal title </h1>
  <p> Modal content </p>
@end
```

### Named slots

Named slots allow the component to accept markup for multiple outlets. For example, a modal component can accept markup for the modal header, body and its actions

```edge
// title: components/modal.edge
<div>
  <header>
    {{{ await $slots.header() }}}
  </header>

  <main>
    {{{ await $slots.main() }}}
  </main>

  <footer>
    {{{ await $slots.actions() }}}
  </footer>
</div>
```

The parent template can define them as follows

```edge
@component('components/modal')
  @slot('header')
    <h1> Delete post </h1>
  @end

  @slot('main')
    <div>
      <p>Are you sure, you want to delete the post</p>
    </div>
  @end

  @slot('actions')
    <div class="flex">
      <button>Yes, delete it</button>
      <button>Cancel</button>
    </div>
  @end
@end
```

#### Output

![](https://res.cloudinary.com/adonis-js/image/upload/q_auto,f_auto/v1617097757/v5/edge-modal-component.png)

### Slots scope

The slots have access to the state of the template in which they are defined (aka the parent template).

Following is the markup for the button component

```edge
// title: components/button.edge
@set('title', 'I am a button')

<button>
  {{{ await $slots.main() }}}
</button>
```

The parent template attempting to access the `title` property defined inside the `component`.

```edge
// title: home.edge
@component('components/button')
  <span>{{ title }}</span>
@end
```

```html
// title: Output
<button>
  <span>undefined</span>
</button>
```

You can pass the data from the component to the parent as arguments to the `slot` method.

```edge
// title: components/button.edge
@set('title', 'I am a button')

<button>
  // highlight-start
  {{{ await $slots.main({ title }) }}}
  // highlight-end
</button>
```

```edge
// title: home.edge
@component('components/button')
  // highlight-start
  @slot('main', scope)
    <span>{{ scope.title }}</span>
  @end
  // highlight-end
@end
```

To summarize:

- Parent template can pass data to the component using props or slots.
- The component can pass data only to the slots as arguments.

## Injecting data to the component tree

The data injection API of Edge is inspired by the [Svelte context API](https://svelte.dev/tutorial/context-api) and [Vue provide/inject API](https://v3.vuejs.org/guide/component-provide-inject.html#provide-inject).

The goal is to simplify the communication between the components inside a tree. However, do note that this is an advanced API and must only be used when you are self authoring a group of components and want transparent communication between them.

### Basic example

Let's start with the most basic example to see the injection API in action. Once, you understand the basics, you can view our screencasts on creating functional components using this API.

You can make use of the `@inject` tag to share an object with the component tree.

```edge
{{-- Declare a local variable --}}
@set('counter', { value: 0 })

{{-- Inject it to the components tree --}}
@inject({ counter })

{{{ await $slots.main() }}}
```

Since the `@inject` tag shares the object by reference, any part of the component tree can mutate it's properties, as shown in the following snippet.

The injected values are available on the `$context` variable.

```edge
@component('components/parent')
  <p> Value of counter is {{ $context.counter.value }} </p>

  {{-- Bump the value by one --}}
  @set($context, 'counter.value', $context.counter.value + 1)

  <p> Updated value is {{ $context.counter.value }} </p>
@end
```

## Components as tags
Edge allows you reference the components stored inside the `./resources/views/components` directory as edge tags.

Let's create the following template inside the `resources/views/components/button.edge` file.

```edge
// title: resources/views/components/button.edge
<button type="{{ type }}">
  {{ text }}
</button>
```

Now, instead of referencing the button component using the `@component` tag. You can also reference it as `@button` tag.

```edge
@!button({
  type: 'primary',
  text: 'Login'
})
```

You can reference the components inside the nested directories with a dot notation. For example, the file stored inside the `./components/form/input.edge` is referenced as follows:

```edge
@!form.input({
  type: 'text',
  placeholder: 'Enter username'
})
```

Following is a reference table to understand the transformations applied to the component path to compute the tag name.

| Template path | Tag name |
|---------------|-----------|
| form/input.edge | `@form.input` |
| tool_tip.edge | `@toolTip` |
| checkout_form/input.edge | `@checkoutForm.input` |
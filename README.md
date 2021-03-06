# Storeon Svelte

[![npm version](https://badge.fury.io/js/%40storeon%2Fsvelte.svg)](https://www.npmjs.com/package/@storeon/svelte)
[![Build Status](https://travis-ci.org/storeon/svelte.svg?branch=master)](https://travis-ci.org/storeon/svelte)


<img src="https://storeon.github.io/storeon/logo.svg" align="right" alt="Storeon logo by Anton Lovchikov" width="160" height="142">

[Svelte] is the smallest JS framework, but even so, it contains many built-in features. One of them is a `svelte/store`. But why we need to use a third-party store? `@storeon/svelte` has several advantages compared with the built-in one.

- **Size**. 179 bytes (+ Storeon itself) instead of 485 bytes (minified and gzipped).
- **Ecosystem**. Many additional [tools] can be combined with a store.
- **Speed**. Bind components to the changes in the exact store that you need.

[storeon]: https://github.com/storeon/storeon
[tools]: https://github.com/storeon/storeon#tools
[svelte]: https://github.com/sveltejs/svelte
[size limit]: https://github.com/ai/size-limit
[demo]: https://codesandbox.io/s/admiring-beaver-edi8m
[article]: https://evilmartians.com/chronicles/storeon-redux-in-173-bytes

## Install
```sh
npm install -S @storeon/svelte
```
or
```sh
yarn add @storeon/svelte
```
## How to use ([Demo])

Create store using `storeon` module:

#### `store.js`

```javascript
import { createStoreon } from 'storeon'

let counter = store => {
  store.on('@init', () => ({ count: 0 }))
  store.on('inc', ({ count }) => ({ count: count + 1 }))
}

export const store = createStoreon([counter])
```

Using TypeScript you can pass `State` and `Events` interface to the `createStoreon` function:

#### `store.ts`

```typescript
import { StoreonModule, createStoreon } from 'storeon'

interface State {
  count: number
}

interface Events {
  'inc': undefined
  'set': number
}

let counter = (store: StoreonModule<State, Events>) => {
  store.on('@init', () => ({ count: 0 }))
  store.on('inc', ({ count }) => ({ count: count + 1 }))
  store.on('set', (_, event) => ({ count: event}))
};

export const store = createStoreon<State, Events>([counter])
```

#### `App.svelte`

Provide store to Svelte Context using `provideStoreon` from `@storeon/svelte`

```html
<script>
  import { provideStoreon } from '@storeon/svelte'
  import { store } from './store'
  import Counter from './Counter.svelte'

  provideStoreon(store)
</script>

<Counter />
```

Import `useStoreon` function from our `@storeon/svelte` module and use it for getting state and dispatching new events:

#### `Child.svelte`

```html
<script>
  import { useStoreon } from '@storeon/svelte';

  const { count, dispatch } = useStoreon('count');

  function increment() {
    dispatch('inc');
  }
</script>

<h1>The count is {$count}</h1>

<button on:click={increment}>+</button>
```
Using typescript you can pass `State` and `Events` interfaces to `useStoreon` function to be full type safe
```html
<script lang="typescript">
  import { useStoreon } from '@storeon/svelte';
  import { State, Events } from './store'

  const { count, dispatch } = useStoreon<State, Events>('count');

  function increment() {
    dispatch('inc');
  }
</script>

<h1>The count is {$count}</h1>

<button on:click={increment}>+</button>
```

## Usage with [@storeon/router](https://github.com/storeon/router)
If you want to use the @storeon/svelte with the `@storeon/router` you should import the `router.createRouter` from `@storeon/router` and add this module to `createStoreon`

#### `store.js`
```js
import { createStoreon } from 'storeon'
import { createRouter } from '@storeon/router';

const store = createStoreon([
  createRouter([
    ['/', () => ({ page: 'home' })],
    ['/blog', () => ({ page: 'blog' })],
  ])
])
```

And use it like:
#### `App.svelte`
```html
<script>
  import { provideStoreon } from '@storeon/svelte'
  import { store } from './store'
  import Counter from './Child.svelte'

  provideStoreon(store)
</script>

<Counter />
```
#### `Child.svelte`
```html
<script>
  import { useStoreon } from '@storeon/svelte';
  import router from '@storeon/router'

  const { [router.key]: route } = useStoreon(router.key)
</script>

You can access the router like default svelte store via $:
{$route.match.page}
```

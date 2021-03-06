- Start Date: 2020-01-30
- Target Major Version: 3.x
- Reference Issues: https://github.com/vuejs/vue-next/issues/671
- Implementation PR:

# Summary

Vue 3 introduces the Composition API, and as part of that, a `watch` method. More information and examples can be found [here](https://vue-composition-api-rfc.netlify.com/api.html#watch).

In Vue 3, the code for the `watch` Composition API is shared with the code used by the `watch` Options API - they exhibit the same behavior. This default behavior is slighly different from Vue 2. The new `watch` Composition API will trigger immediately after the component is mounted. This is not the case for Vue 2; this behavior is opt in, via the `immediate` option.

This RFC proposes to keep this change moving forward in Vue 3 `watch` handlers will be invoke immediately, both when using the `watch` method and the `watch` Options API.

# Basic example

This is a simple component that works as-is with Vue 2.x and 3.x.

```js
const App = {
  data() {
    return {
      message: "Hello"
    };
  },
  watch: {
    message: {
      handler() {
        console.log("Immediately Triggered")
      }
    }
  }
}
```

Although this component is valid in both Vue 2 and Vue 3, it behaves differently.

- If this component is mounted in Vue 2, the `message` handler will not be triggered until `message` is changed for the first time. A user can opt to fire the `handler` immediately by passing `immediate: true`. 
- In Vue 3, the `message` handle will trigger immediately; this is the default behavior in Vue 3 - the opposite of Vue 2. To opt out of this behavior in Vue 3, the user can pass a `{ lazy: true }` option.

It is relevant to note that `watch` in Vue 3 is not fired synchronously (even for the initial call) - they are in fact deferred until the component is mounted. Even with immediate by default, watchers won't fire during SSR unless you explicitly use { flush: 'sync' } in the watcher's options.

# Motivation

It is better to be consistent across the Composition API and the Options API. While they look different, the two APIs Vue provides are just two different ways of accomplishing the same thing.

One common occurrence is the following:

```js
created() {
  this.fetchData(this.id)
},
watch: {
  id: fetchData
},
methods: {
  fetchData(id) {
    // ...
  }
}
```

An example of this is an application with a route like `/users/:id`, where you want to load the user on the initial load, and whenever the `id` param changes. With `immediate: true` as the default, the above can be written like this:

```js
watch: {
  id(id) {
    // fetch data...
  }
}
```

# Detailed design

The `watch` Options API should behave the same as the `watch` method provided by the Composition API.

# Drawbacks

The main cost of keeping this breaking change is the migration cost for existing codebases. Having `watch` trigger immediately when upgraing to Vue 3 may cause unintended behavior. The user will need to add `{ lazy: false }` to each watch API they do not want to trigger immediately.

It is possible many commonly used modules use the old `watch` behavior; this could break, or at least change the behavior of many applications and libraries.

# Alternatives

An alternative would be to implement an `immediate` option that defaults to `false` for the `watch` Options API in Vue 3. This would mean no breaking change, but a larger API surface with discrepancy between the Options API and Composition API.

# Adoption strategy

- The user will need to add `{ lazy: false }` to each watch API they do not want to trigger immediately in their existing codebases. 
- A codemod to assist with this migration should be possible in most cases. 
- It should be possible to write a runtime adapter to maintain the Vue 2 behavior.

# Unresolved questions

N/A

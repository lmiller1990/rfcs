- Start Date: 19.04.2020
- Target Major Version: 2.x
- Reference Issues: (fill in existing related issues, if any)
- Implementation PR: (leave this empty)

# Summary

VueTestUtils 2.x will introduce a few new methods and remove some less used ones.

-**Breaking:** Everything is Async.
-**Breaking:** `find` is now split into `find` and `findComponent`.
-**Breaking:** Removal of properties and methods, as they induce bad testing habits or are obsolete. 
-**Breaking:** Stubs no longer render slots by default.
-**Breaking:** `setProps` only works for the mounted component. 

**Note:** VueTestUtils v2.x will be aimed towards Vue 3. The API for VueTestUtils 1.x will stay the same and will support Vue 2.x. 

# Motivation

- Provide a smaller and easier to understand API
- Allow adding custom functionality via plugin system. [TODO]
- Remove methods that are bloat or lead to bad testing habits.
- Generally improve VTU Docs and Guides.

# Detailed design

* `createLocalVue` is removed. Vue now exposes `createApp`, which creates an isolated app instance. VTU does that under the hood.
* Fully `async`, each method that involves a mutation returns a promise. Methods like `setValue` and `trigger` can be awaited now. 
* rewritten completely in TypeScript, giving much improved type hints when writing tests.

## API 

### Mount

[Link](https://vuejs.github.io/vue-test-utils-next-docs/api/#mount)

Mount stays as the main method to mount a component for testing.

### Mount Options

#### data

[Link](https://vuejs.github.io/vue-test-utils-next-docs/api/#data)

Not changed from VTU Beta.

#### props

[Link](https://vuejs.github.io/vue-test-utils-next-docs/api/#props)

Renamed from `propsData`.

#### slots

[Link](https://vuejs.github.io/vue-test-utils-next-docs/api/#slots)

Passes slots down to the component. 

**Note:** Read about [ScopedSlots](#scopedslots)

#### global 

[Link](https://vuejs.github.io/vue-test-utils-next-docs/api/#global-components)

The `global` namespace is used to pass configuration to the `createApp` instance. Things like global components, directives and so on.

* global.components - register global components
* global.directives - register a global directive
* global.mixins - register a global mixin
* global.mocks - mock a globally registered property
* global.plugins - install a plugin
* global.provide - provide something via Provide/Inject api.
* global.stubs - stub out components.

### Properties

#### element

Returns the DOM element.

#### vm

Returns the VM of a `VueWrapper`.

### Methods

#### attributes

[Link](https://vuejs.github.io/vue-test-utils-next-docs/api/#attributes)

Unchanged.

#### classes

[Link](https://vuejs.github.io/vue-test-utils-next-docs/api/#classes)

- **New** - If Vue component has multiple roots, will throw error.

#### emitted

[Link](https://vuejs.github.io/vue-test-utils-next-docs/api/#emitted)

- **New** - Only available on `VueWrapper`.

#### exists

[Link](https://vuejs.github.io/vue-test-utils-next-docs/api/#exists)

Unchanged

#### find

[Link](https://vuejs.github.io/vue-test-utils-next-docs/api/#find)

- **Breaking** - Returns only `DOMWrapper`. Cannot find Component instances. see [findComponent](#findcomponent)
- **Breaking** - Accepts query selector only. 
- **New** - Can now return instance root element, or a fragment of the root.

#### findAll

[Link](https://vuejs.github.io/vue-test-utils-next-docs/api/#findall)

- **Breaking** - Returns only array of `DOMWrapper`. 
- **Breaking** - No longer returns `WrapperArray`.
- **Breaking** - Accepts query selector only. 

#### findComponent

[Link](https://vuejs.github.io/vue-test-utils-next-docs/api/#findcomponent)

- **New** - finds a Vue Component instance by `ref`, `name`, `query` or Component definition. Returns `VueWrapper`.
- **New** - Only available on `VueWrapper`.

#### findAllComponents

[Link](https://vuejs.github.io/vue-test-utils-next-docs/api/#findallcomponents)

- **New** - finds all Vue Components that match `name`, `query` or Component Definition. Returns array of `VueWrapper`.
- **New** - Only available on `VueWrapper`.

#### html

[Link](https://vuejs.github.io/vue-test-utils-next-docs/api/#html) 

Unchanged.

#### props

[Link](https://vuejs.github.io/vue-test-utils-next-docs/api/#props-2)

Unchanged.

#### setProps

[Link](https://vuejs.github.io/vue-test-utils-next-docs/api/#setprops)

- **Breaking** - Only works on mounted component.
- **New** - Returns promise

#### setValue

[Link](https://vuejs.github.io/vue-test-utils-next-docs/api/#setvalue)

- **Breaking** - Only works on `DOMWrapper` (for now).
- **New** - Unifies `setChecked` and `setSelected`.
- **New** - Returns promise

#### text 

[Link](https://vuejs.github.io/vue-test-utils-next-docs/api/#text)

Unchanged

#### trigger

[Link](https://vuejs.github.io/vue-test-utils-next-docs/api/#trigger)

Unchanged.

## Deprecated/Removed methods

### emittedByOrder

[Link](https://vue-test-utils.vuejs.org/api/wrapper/#emittedbyorder)

Rarely used, use `emitted` instead.

```js
expect(wrapper.emitted('change')[0]).toEqual(['param1', 'param2'])
```

### is

[Link](https://vue-test-utils.vuejs.org/api/wrapper/#is)

Use `element.tagName` or the `classes()` method.

```js
expect(wrapper.element.tagName).toEqual('div')
expect(wrapper.classes()).toContain('Foo')
```

### isEmpty

[Link](https://vue-test-utils.vuejs.org/api/wrapper/#isempty)

Use custom matcher like [jest-dom#tobeempty](https://github.com/testing-library/jest-dom#tobeempty) on the element.

```js
expect(wrapper.element).toBeEmpty()
```

### isVisible

[Link](https://vue-test-utils.vuejs.org/api/wrapper/#isvisible)

Use custom matcher like [jest-dom#tobevisible](https://github.com/testing-library/jest-dom#tobevisible)

```js
expect(wrapper.element).toBeVisible()
```

### isVueInstance

[Link](https://vue-test-utils.vuejs.org/api/wrapper/#isvueinstance)

No longer necessary, `find` always returns an `DOMWrapper` and `findComponent` returns a `VueWrapper`. Both return `ErrorWrapper` if failed.

### setMethods

[Link](https://vue-test-utils.vuejs.org/api/wrapper/#setmethods)

Anti-pattern. Vue does not support arbitrarily replacement of methods, nor should VTU.

If you need to stub out an action, extract the hard parts away. Then you can unit test them as well.

```js
// Component.vue
import { asyncAction } from 'actions'
const Component = {
    ...,
    methods: {
        async someAsyncMethod() {
            this.result = await asyncAction()
        }
    }	
}

// spec.js
import { asyncAction } from 'actions'
jest.mock('actions')
asyncAction.mockResolvedValue({ foo: 'bar' })

// rest of your test

```

## setChecked and setSelected
 
Merged with [setValue](#setvalue)

## Deprecated/Removed mount options

- **Wrapper.name** - [Link](https://vue-test-utils.vuejs.org/api/wrapper/#name) - Removed from core. Could be added as part of extended plugin.

## Deprecated/Removed Classes and properties

- **WrapperArray** - [Link](https://vue-test-utils.vuejs.org/api/wrapper-array/) - `find` and `findComponent` will just return an array of `VueWrapper` or `DOMWrapper` respectively.
- **config.methods** - [Link](https://vue-test-utils.vuejs.org/api/#methods) - Will no longer be able to replace methods.
- **config.silent** - [Link](https://vue-test-utils.vuejs.org/api/#silent)
- **Wrapper.options** - [Link](https://vue-test-utils.vuejs.org/api/wrapper/#options)

## Not yet implemented

- **Wrapper.selector** [Link](https://vue-test-utils.vuejs.org/api/wrapper/#selector)
- **shallowMount** - [Link](https://vue-test-utils.vuejs.org/api/#shallowmount) - **Stubs** work, so its halfway there.
- **render** - [Link](https://vue-test-utils.vuejs.org/api/#render)
- **renderToString** - [Link](https://vue-test-utils.vuejs.org/api/#rendertostring)
- **createWrapper** - [Link](https://vue-test-utils.vuejs.org/api/#createwrapper-node-options)
- **enableAutoDestroy** - [Link](https://vue-test-utils.vuejs.org/api/#enableautodestroy-hook)
- **Wrapper.contains** - [Link](https://vue-test-utils.vuejs.org/api/wrapper/#contains)

### scopedSlots

ScopedSlots are not ready yet. They will most probably be merged with normal ones, and will be a function with data, similar to VTU Beta.
It is yet to be decided.

# Drawbacks

- People will have to separate `find` into `findComponent` and `find`. We hope this would make tests easier to read and reason with.
- Snapshots would have to be updated.
- Some deprecated methods and functionality would have to be most likely installed via an extra plugin.

# Adoption strategy

- Rewrite Docs from ground up. 
- Add dedicated guides on how to write better and more maintainable tests for popular tools like Vuex, Router ec..
- Work with popular Vue ecosystem libraries and frameworks, like Quasar, Nuxt and Vuetify for better understanding of user needs.
- Deprecation build with warnings. 

# Unresolved questions

Optional, but suggested for first drafts. What parts of the design are still
TBD?

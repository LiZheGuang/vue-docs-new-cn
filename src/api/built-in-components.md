# Built-in Components

Built-in components can be used directly in templates without needing to be registered.

The `<Transition>`, `<TransitionGroup>`, `<KeepAlive>`, `<Teleport>` and `<Suspense>` components can all be tree-shaken by bundlers, so that they are only included in the build if they're used.They can also be imported explicitly if you need direct access to the component itself:

```js
// ESM build of Vue
import { Transition, TransitionGroup, KeepAlive, Teleport, Suspense } from 'vue'
```

```js
// CDN build of Vue
const { Transition, TransitionGroup, KeepAlive, Teleport, Suspense } = Vue
```

## `<Transition>`

- **Props:**

  - `name` - `string` Used to automatically generate transition CSS class names. e.g. `name: 'fade'` will auto expand to `.fade-enter`, `.fade-enter-active`, etc.
  - `appear` - `boolean`, Whether to apply transition on initial render. Defaults to `false`.
  - `persisted` - `boolean`. If true, indicates this is a transition that doesn't actually insert/remove the element, but toggles the show / hidden status instead. The transition hooks are injected, but will be skipped by the renderer. Instead, a custom directive can control the transition by calling the injected hooks (e.g. `v-show`).
  - `css` - `boolean`. Whether to apply CSS transition classes. Defaults to `true`. If set to `false`, will only trigger JavaScript hooks registered via component events.
  - `type` - `string`. Specifies the type of transition events to wait for to determine transition end timing. Available values are `"transition"` and `"animation"`. By default, it will automatically detect the type that has a longer duration.
  - `mode` - `string` Controls the timing sequence of leaving/entering transitions. Available modes are `"out-in"` and `"in-out"`; defaults to simultaneous.
  - `duration` - `number | { enter: number, leave: number }`. Specifies the duration of transition. By default, Vue waits for the first `transitionend` or `animationend` event on the root transition element.
  - `enter-from-class` - `string`
  - `leave-from-class` - `string`
  - `appear-class` - `string`
  - `enter-to-class` - `string`
  - `leave-to-class` - `string`
  - `appear-to-class` - `string`
  - `enter-active-class` - `string`
  - `leave-active-class` - `string`
  - `appear-active-class` - `string`

- **Events:**

  - `before-enter`
  - `before-leave`
  - `enter`
  - `leave`
  - `appear`
  - `after-enter`
  - `after-leave`
  - `after-appear`
  - `enter-cancelled`
  - `leave-cancelled` (`v-show` only)
  - `appear-cancelled`

- **Usage:**

  `<transition>` serve as transition effects for **single** element/component. The `<transition>` only applies the transition behavior to the wrapped content inside; it doesn't render an extra DOM element, or show up in the inspected component hierarchy.

  ```vue-html
  <!-- simple element -->
  <transition>
    <div v-if="ok">toggled content</div>
  </transition>

  <!-- dynamic component -->
  <transition name="fade" mode="out-in" appear>
    <component :is="view"></component>
  </transition>

  <!-- event hooking -->
  <div id="transition-demo">
    <transition @after-enter="transitionComplete">
      <div v-show="ok">toggled content</div>
    </transition>
  </div>
  ```

  ```js
  const app = createApp({
    ...
    methods: {
      transitionComplete (el) {
        // for passed 'el' that DOM element as the argument, something ...
      }
    }
    ...
  })

  app.mount('#transition-demo')
  ```

- **See also:** [`<Transition>` Guide](/guide/built-ins/transition.html)

## `<TransitionGroup>`

- **Props:**

  - `tag` - `string`, if not defined, renders without a root element.
  - `move-class` - overwrite CSS class applied during moving transition.
  - exposes the same props as `<transition>` except `mode`.

- **Events:**

  - exposes the same events as `<transition>`.

- **Usage:**

  `<transition-group>` provides transition effects for **multiple** elements/components. By default it doesn't render a wrapper DOM element, but one can be defined via the `tag` attribute.

  Note that every child in a `<transition-group>` must be [**uniquely keyed**](./built-in-special-attributes.html#key) for the animations to work properly.

  `<transition-group>` supports moving transitions via CSS transform. When a child's position on screen has changed after an update, it will get applied a moving CSS class (auto generated from the `name` attribute or configured with the `move-class` attribute). If the CSS `transform` property is "transition-able" when the moving class is applied, the element will be smoothly animated to its destination using the [FLIP technique](https://aerotwist.com/blog/flip-your-animations/).

  ```vue-html
  <transition-group tag="ul" name="slide">
    <li v-for="item in items" :key="item.id">
      {{ item.text }}
    </li>
  </transition-group>
  ```

- **See also:** [`<TransitionGroup>` Guide](/guide/built-ins/transition-group.html)

## `<KeepAlive>`

- **Props:**

  - `include` - `string | RegExp | Array`. Only components with matching names will be cached.
  - `exclude` - `string | RegExp | Array`. Any component with a matching name will not be cached.
  - `max` - `number | string`. The maximum number of component instances to cache.

- **Usage:**

  When wrapped around a dynamic component, `<keep-alive>` caches the inactive component instances without destroying them. Similar to `<transition>`, `<keep-alive>` is an abstract component: it doesn't render a DOM element itself, and doesn't show up in the component parent chain.

  When a component is toggled inside `<keep-alive>`, its `activated` and `deactivated` lifecycle hooks will be invoked accordingly, providing an alternative to `mounted` and `unmounted`, which are not called. (This applies to the direct child of `<keep-alive>` as well as to all of its descendants.)

  Primarily used to preserve component state or avoid re-rendering.

  ```vue-html
  <!-- basic -->
  <keep-alive>
    <component :is="view"></component>
  </keep-alive>

  <!-- multiple conditional children -->
  <keep-alive>
    <comp-a v-if="a > 1"></comp-a>
    <comp-b v-else></comp-b>
  </keep-alive>

  <!-- used together with `<transition>` -->
  <transition>
    <keep-alive>
      <component :is="view"></component>
    </keep-alive>
  </transition>
  ```

  Note, `<keep-alive>` is designed for the case where it has one direct child component that is being toggled. It does not work if you have `v-for` inside it. When there are multiple conditional children, as above, `<keep-alive>` requires that only one child is rendered at a time.

- **`include` and `exclude`**

  The `include` and `exclude` props allow components to be conditionally cached. Both props can be a comma-delimited string, a RegExp or an array:

  ```vue-html
  <!-- comma-delimited string -->
  <keep-alive include="a,b">
    <component :is="view"></component>
  </keep-alive>

  <!-- regex (use `v-bind`) -->
  <keep-alive :include="/a|b/">
    <component :is="view"></component>
  </keep-alive>

  <!-- Array (use `v-bind`) -->
  <keep-alive :include="['a', 'b']">
    <component :is="view"></component>
  </keep-alive>
  ```

  The match is first checked on the component's own `name` option, then its local registration name (the key in the parent's `components` option) if the `name` option is not available. Anonymous components cannot be matched against.

- **`max`**

  The maximum number of component instances to cache. Once this number is reached, the cached component instance that was least recently accessed will be destroyed before creating a new instance.

  ```vue-html
  <keep-alive :max="10">
    <component :is="view"></component>
  </keep-alive>
  ```

  ::: warning
  `<keep-alive>` does not work with functional components because they do not have instances to be cached.
  :::

- **See also:** [`<KeepAlive/>`](/guide/built-ins/keep-alive.html)

## `<Teleport>`

- **Props:**

  - `to` - `string`. Required prop, has to be a valid query selector, or an HTMLElement (if used in a browser environment). Specifies a target element where `<teleport>` content will be moved

  ```vue-html
  <!-- ok -->
  <teleport to="#some-id" />
  <teleport to=".some-class" />
  <teleport to="[data-teleport]" />

  <!-- Wrong -->
  <teleport to="h1" />
  <teleport to="some-string" />
  ```

  - `disabled` - `boolean`. This optional prop can be used to disable the `<teleport>`'s functionality, which means that its slot content will not be moved anywhere and instead be rendered where you specified the `<teleport>` in the surrounding parent component.

  ```vue-html
  <teleport to="#popup" :disabled="displayVideoInline">
    <video src="./my-movie.mp4">
  </teleport>
  ```

  Notice that this will move the actual DOM nodes instead of being destroyed and recreated, and it will keep any component instances alive as well. All stateful HTML elements (i.e. a playing video) will keep their state.

- **See also:** [Teleport component](/guide/built-ins/teleport.html#teleport)

## `<Suspense>`

// TODO

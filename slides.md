---
theme: default
class: text-center
css: unocss
---

# Understanding Vue.js Reactivity System

*How Vue efficiently manages data updates and UI rendering*

by Marius Heine (GEPROG GmbH)

made with https://sli.dev

---

# What is Reactivity in Vue?

Vue's reactivity system automatically tracks changes to data and updates the DOM to reflect those changes.

- Efficient DOM updates
- Track dependencies and changes
- Automate complex UI state management

---

# The Core of Vue's Reactivity

Vue achieves reactivity through two main tools:
- **Reactivity Tracking**: Vue tracks dependencies between data and DOM updates.
- **Reactivity Triggering**: When data changes, Vue triggers DOM updates.

Vue uses **proxies** in Vue 3 to intercept data access and changes.

---

# Vue 2 Reactivity

- In Vue 2, Vue used **Object.defineProperty** to make data reactive.
- Vue would wrap your data properties in `getters` and `setters`.
- **Limitation**: Only works with existing properties (added after Vue instance creation).

---

# Vue 3 Reactivity

- Vue 3 uses **Proxies** to enhance reactivity.
- **Advantages**: Reactivity is applied to the entire object, including dynamic properties.
- **More efficient**: Better handling of deeply nested data and better memory usage.

Example:
```js
const state = reactive({ count: 0 });
state.count++;  // Triggers a reactivity update
```

---

# Reactive Core APIs in Vue 3

Vue provides key reactivity APIs for developers:
- **reactive()**: Make an object reactive.
- **ref()**: Create reactive primitives (number, string, etc.).
- **computed()**: Define reactive computed properties.
- **watch()**: Watch for changes in reactive properties.

```js
import { reactive, ref, computed, watch } from 'vue';

// Example Usage
const state = reactive({ count: 0 });
const doubleCount = computed(() => state.count * 2);
watch(() => state.count, (newVal) => console.log(newVal));
```

---

# Dependency Tracking and Reactivity Flow

Vue automatically tracks dependencies when rendering a component:
- **Dependencies**: Reactive properties used within a component.
- **Effect function**: The function that Vue runs when dependencies change.

Flow:
1. Reactive data triggers change.
2. Vue collects dependencies.
3. UI updates automatically based on the dependency changes.

---

# Computed Properties

- Computed properties depend on reactive data.
- Vue caches computed properties to avoid unnecessary re-evaluations.
- **Benefits**: Efficiency, no manual tracking required.

Example:
```js
const price = ref(100);
const quantity = ref(2);

const totalPrice = computed(() => price.value * quantity.value);
```

---

# Watchers in Vue

- Watchers allow you to run custom logic in response to reactive changes.
- **watch()** and **watchEffect()** are key APIs:
  - `watch`: Watch specific data.
  - `watchEffect`: Automatically track reactive dependencies within a function.

Example:
```js
watchEffect(() => {
  console.log(state.count);  // Logs on every change to state.count
});
```

---

# Example: Real-World Scenario

- Imagine a shopping cart system.
- Cart total updates automatically when items or prices change.

```js
const cart = reactive({ items: [], total: 0 });

watchEffect(() => {
  cart.total = cart.items.reduce((sum, item) => sum + item.price, 0);
});
```

---

# Reactivity Caveats

There are a few caveats to be aware of:
- **Non-reactive properties**: Not all properties are reactive (e.g., non-reactive types like Symbols).
- **Array mutations**: Vue tracks array operations but not mutations directly on array elements.

Example of a caveat with arrays:
```js
const arr = reactive([1, 2, 3]);
arr[0] = 10; // Does not trigger reactivity!
arr.push(4); // This works!
```

---

# Are Props reactive?

- since version 3.5 yes
- before you have to use `toRef` or `toRefs`

---

# Scenario

When will `c` be recalculated?

```vue
<script setup lang="ts">
import { ref, computed, watch } from 'vue'

const data = ref<{ a: string; b: string; x: number }>({ a: 'a', b: 'b', x: 0 });
const b = computed(() => data.value.b);
const c = computed(() => ({ a: data.value.a, b: b.value }));

const recalculations = ref(0);
watch(c, () => recalculations.value = recalculations.value + 1);
</script>

<template>
  <pre>{{ data }}</pre>
  <div>Recalculations: {{ recalculations }}</div>
  <button @click="data.x++">Increase x</button>
  <button @click="data.a = `${data.a}${data.a}`">Change a</button>
</template>
```
<a href="https://play.vuejs.org/#eNp9U01vm0AQ/SujVSUTGeFU7cnBqG2UQ3poq7THPWRZxjYJLGg/HCTEf+/sbm2TJvKJnXlv3nwysq99nx0csjXLjdR1b8GgdT00Qu02nFnDWcFV3fadtjCCxm0Ksmt7Z7FK4UVYuYcJtrprYUE6C664kp0yFiphBWx8RD6CWIOxula7GyjPz2ENyrUlapiKJJAWYpF6xqKkL8HXMF3dnDVLEjxmT5Ir2BQhTXYQjcOsnFPlW2pMMQsQIVcZLco0j9coRSNdI2xNdmwkufaM0HQiU4iqr4n/tDz9HfcSPnqFfBVnTZMlw2LbEw3JAsh7jcU4xulNU77ydgCq+lA8vBJdAxH/q9OHeGYIKZ21nYIvsqnlM20ztD4sl7TTeyU1CoMw5KtIuxDi9/j4YYzv6fR4JJ3bPR0Kgpip5KtTRyylC6J5butd9mQ6RWc2+jSc+dXUDeqffaibM2rGIx4TTdO9fA8+qx2mR7/co3x+x/9kBu/j7JdGg/qAnJ0wK/QObYTvfv/Agd4nsO0q1xD7AviApmucrzHSvjlVUdkzXqj2PvwgdNR/zN1gUZljU75Qz5wCnzP6R24vtH4u91P2OcRxNbHpL5m4N7k=" target="_blank">Playground</a>

---

# Scenario v2

When will `c` be recalculated?

```vue
<script setup lang="ts">
import { ref, computed, watch } from 'vue'

const data = ref<{ a: string; b: string; x: number }>({ a: 'a', b: 'b', x: 0 });
const b = computed(() => data.value.b);
const c = computed(() => ({ a: data.value.a, b: b.value }));

const recalculations = ref(0);
watch(c, () => recalculations.value = recalculations.value + 1);
</script>

<template>
  <pre>{{ data }}</pre>
  <div>Recalculations: {{ recalculations }}</div>
  <button @click="data = { ...data, x: data.x + 1 }">Increase x</button>
  <button @click="data.a = `${data.a}${data.a}`">Change a</button>
</template>
```
<a href="https://play.vuejs.org/#eNp9U01v2zAM/SuEMMAuZjgdtlPqGNuKHrrDNnQ76lBZZhK3tmzoIzVg+L+PkprE3bqeTPI9Pn6IntiXYcgPDtmaFUbqZrBg0LoBWqF2G86s4azkqumGXluYQOM2A9l3g7NYZ/AkrNzDDFvdd5CQTsIVV7JXxkItrICNzygmEGswVjdqdwXV2RzXoFxXoYa5TAMpEUnmGUlFX4IvYb64OmtWJHisnqYXsClDmfwgWod5taTKf6mxxCJBhFpV9KjSMl+jFK10rbAN+XGQ9NIzwtCpzCCqviQ+a3n6K+H38MErFKu4a9osORa7gWhIHkAxaCynKW5vnouV9wNQN4fy7oXoGoj4V58+xTNDSuWs7RV8lm0jH+k1n59kgjzPvR1WHPYx+tZgpse+VVKjMAhjsYr5/9fKvdr9uyna88m4J53rPV0QglioFKvTqCyj06JFb5td/mB6Rfc3+TKc+TdrWtQ/hjAQZzSlRzwm2rZ/+hZiVjvMjnG5R/n4SvzBjD7G2U+NBvUBOTthVugd2gjf/PqOI9knsOtr1xL7DfAOTd8632OkfXWqprYXvNDtbfhz6Np/m5vRojLHoXyjnjkHPmf081y/Mfq53Y/5p5DH1czmPxgNPpA=" target="_blank">Playground</a>

---

# Scenario v3

When will `c` be recalculated?

```vue
<script setup lang="ts">
import { ref, computed, watch } from 'vue'

const data = ref<{ a: string; b: string; x: number }>({ a: 'a', b: 'b', x: 0 });
const a = computed(() => data.value.a);
const b = computed(() => data.value.b);
const c = computed(() => ({ a: a.value, b: b.value }));

const recalculations = ref(0);
watch(c, () => recalculations.value = recalculations.value + 1);
</script>

<template>
  <pre>{{ data }}</pre>
  <div>Recalculations: {{ recalculations }}</div>
  <button @click="data = { ...data, x: data.x + 1 }">Increase x</button>
  <button @click="data.a = `${data.a}${data.a}`">Change a</button>
</template>
```
<a href="https://play.vuejs.org/#eNp9U01v2zAM/SuEMCAuZjgdtlPqGNuKHrrDNnQ76lBZZhK3tmxIcmrA0H8fJeXD3bqcRJGPT48fmtiXvs/2A7IVy43UdW/BoB16aITarjmzhrOCq7rtO21hAo2bFGTX9oPFKoUXYeUOHGx018KCeBZccSU7ZSxUwgpY+4x8ArECY3WttjdQns1xBWpoS9TgiiSAFmKResSipJPC1+Cubs6cnvD4epJcwboIz2R70QyYiTm0vAgt51D5LzSqOaCDojLapGeeqlGKRg6NsDXdY7nJtUeE1iQyhUj4Gnjg8vA33O/hg2fIl3Ei1H+6WGx7giHdAPJeYzFNscfO5Ut/D4Gq3hcPr0hXQMC/dPoUjwwp5WBtp+CzbGr5TDM/DG6CLMu8HQYRejd6aeBoJe6V1CgMwpgvY/7/uTLP9vhuirY7GY/Ec7ujPUMQM5Z8eSqVpbSA1OhNvc2eTKdoSyf/DGd+XHWD+kcfCuKMqvQRHxNN0718Cz6raXpHv9yhfH7D/2RG7+Psp0aDeo+cnWJW6C3aGL779R1Hsk/BtquGhtAXgg9oumbwGiPs66Aqkj3DBbX34X/Rn/ht7kaLyhyL8kI90gU8Z/TFbi+Ufpb7MfsU8rhyzP0BomhK7g==" target="_blank">Playground</a>

---

# Summary of Vue's Reactivity System
- Reactivity is the heart of Vue's efficient DOM updates.
- Vue 3’s **Proxy-based reactivity** overcomes limitations of Vue 2.
- Core concepts:
  - **reactive()**, **ref()**, **computed()**, **watch()**
- Understanding these tools allows for efficient, scalable UI management.

Read the docs
- https://vuejs.org/guide/essentials/reactivity-fundamentals.html
- https://vuejs.org/guide/components/props.html#props

Q&A / Discussion?

---

# Questions?
Feel free to ask!

---

<div class="mt-40 text-center text-[5rem]">Vielen Dank!</div>

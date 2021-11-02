# Installation

## Getting the code

FormKit can be downloaded using a package manager like `npm` or `yarn` or it can be used directly via CDN.

## From npm

Most new projects use a build tool like Vite, Snowpack, or WebPack — this makes installing npm dependencies a piece of cake 🍰.

```sh
npm install @formkit/vue
```

The `@formkit/vue` package ships with a Vue plugin and a default configuration that makes installation a breeze.

```js
import { createApp } from 'vue'
import App from 'App.vue'
import { plugin, defaultConfig } from '@formkit/vue'

createApp(App).use(plugin, defaultConfig).mount('#app')
```

That's it! You're not ready to use the `<FormKit>` component in your Vue application. The `defaultConfig` includes all of FormKit's inputs, validation rules, and the english language. You can replace the `defaultConfig` with your own configuration — this allows for improved tree-shaking (only include the rules and languages you want to actually use) and more fine-grained control.

<callout type="warning" label="Vue 2">
FormKit only supports Vue 3. If you're required to use Vue 2 on a project, consider using the spiritual ancestor of FormKit — [Vue Formulate](https://vueformulate.com/).
</callout>


## From a CDN

FormKit can also be used directly from a CDN with a simple `<script>` tag.

```html
<script src="https://unpkg.com/vue@next"></script>
<script src="https://unpkg.com/@formkit/vue/formkit-vue.js"></script>
```

Then anywhere after that point in your application, you can access the `FormKitVue` global variable which includes everything you need to boot up FormKit.

```js
// Extract from FormKitVue global.
const { plugin, defaultConfig, FormKit } = FormKitVue

// Some root application
const App = {
  template: `
  <div>
    <h1>Your App!</h1>
    <FormKit type="text" />
  </div>`
}

Vue.createApp(App).use(plugin, defaultConfig).mount('#app')
```
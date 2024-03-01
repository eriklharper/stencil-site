---
title: Publishing A Component Library
sidebar_label: Publishing
description: Publishing A Component Library
slug: /publishing
---

There are numerous strategies to publish and distribute your component library to be consumed by external projects. One of the benefits of Stencil is that is makes it easy to generate the various [output targets](../output-targets/01-overview.md) that are right for your use-case.

## Use Cases

For using Stencil components in other projects there are two important output targets necessary: [`dist`](distribution) and [`dist-custom-elements`](custom-elements). Both export your components for different use cases. Luckily, all builds can be generated at the same time, using the same source code, and shipped in the same distribution. It would be up to the consumer of your component library to decide which build to use.

### Lazy Loading

If you prefer to have your components automatically loaded when used in your application, we recommend to enable the [`dist`](distribution) output target. The bundle gives you a small entry file that registers all your components and only loads the full component logic when it gets rendered in your application. It doesn't matter if the actual application is written in HTML or created with vanilla JavaScript, jQuery, React, etc.

Your users can import your component library, e.g. called `my-design-system`, either via a `script` tag:

```html
<script type="module" src="https://unpkg.com/my-design-system"></script>
```

or by importing it in the bootstrap script of your application:

```ts
import 'my-design-system';
```

To ensure that the right entry file is loaded when importing the project, define the following fields in your `package.json`:

```json
{
  "exports": "./dist/esm/my-design-system.js",
  "main": "./dist/cjs/my-design-system.js",
  "unpkg": "dist/my-design-system/my-design-system.esm.js",
}
```

Read more about various options when it comes to distributing your components lazily in the [`dist`](distribution) output target section.

#### Considerations

Loading Stencil components in an application lazily is a great approach, especially if you develop a large design system with many components. It enables you to import a small single file and have all components available through the entire application.

However be aware that this approach is not ideal in all cases. It requires your application to ship the bundled components as static assets in order for them to load properly. Furthermore, having many nested component dependencies can have an impact on the performance of your application. For example, given you have a component `CmpA` which uses a Stencil component `CmpB` which itself uses another Stencil component `CmpC`. In order to fully render `CmpA` the browser has to load 3 scripts sequentially which can result in undesired rendering delays.

### Standalone

The [`dist-custom-elements`](custom-elements) output target builds each component as a stand-alone class that extends `HTMLElement`. The output is a standardized custom element with the styles already attached and without any of Stencil's lazy-loading. This may be preferred for projects that are already handling bundling, lazy-loading and defining the custom elements themselves.

You can use these standalone components by importing them via:

```ts
import { MyComponent, defineCustomElementMyComponent } from 'my-design-system'

// register to CustomElementRegistry
defineCustomElementMyComponent()

// or extend custom element via
class MyCustomComponent extends MyComponent {
  // ...
}
define('my-custom-component', MyCustomComponent)
```

To ensure that the right entry file is loaded when importing the project, define the following fields in your `package.json`:

```json
{
  "exports": {
    ".": {
      "import": "./dist/components/index.js",
      "types": "./dist/components/index.d.ts"
    },
    "my-component": {
      "import": "./dist/components/my-component.js",
      "types": "./dist/components/my-component.d.ts"
    }
  },
  "types": "dist/components/index.d.ts",
}
```

If you define exports targets for all your components as shown above and by using `customElementsExportBehavior: 'auto-define-custom-elements'` as output target option, you can skip the `defineCustomElement` call and directly import the component where you need it:

```ts
import 'my-design-system/my-component'
```

:::note
If you are distributing both the `dist` and `dist-custom-elements`, then it's best to pick one of them as the main entry depending on which use case is more prominent.
:::

Read more about various options when it comes to distributing your components as standalone components in the [`dist-custom-elements`](custom-elements) output target section.

#### Considerations

This distribution strategy is useful when you use an external bundler such as [Vite](https://vitejs.dev/), [WebPack](https://webpack.js.org/) or [Rollup](https://rollupjs.org) to compile your application. They ensure that only the components used within your application are bundled into compilation.

#### Usage in TypeScript

If you plan to support consuming your component library in TypeScript you'll need to set `generateTypeDeclarations: true` on the your output target in `stencil.config.ts`, like so:

```tsx title="stencil.config.ts"
import { Config } from '@stencil/core';

export const config: Config = {
  outputTargets: [
    {
      type: 'dist-custom-elements',
      generateTypeDeclarations: true,
    },
    // ...
  ],
  // ...
};
```

Then you can set the `types` property in `package.json` so that consumers of your package can find the type definitions, like so:

```tsx title="package.json"
{
  "types": "dist/components/index.d.ts",
  "dependencies": {
    "@stencil/core": "latest"
  },
  ...
}
```

:::note
If you set the `dir` property on the output target config, replace `dist/components` in the above snippet with the path set in the config.
:::

## Publishing to NPM

The first step and highly recommended step is to [publish the component library to NPM](https://docs.npmjs.com/getting-started/publishing-npm-packages). [NPM](https://www.npmjs.com/) is an online software registry for sharing libraries, tools, utilities, packages, etc. Once the library is published to NPM, other projects are able to add your component library as a dependency and use the components within their own projects.

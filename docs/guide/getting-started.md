# Getting Started

## Installation

```bash
$ yarn add qiankun # or npm i qiankun -S
```

## Register Sub Apps In Master Application

```ts
import { registerMicroApps, start } from 'qiankun';

registerMicroApps([
  {
    name: 'react app', // app name registered
    entry: '//localhost:7100',
    render: ({ appContent, loading }) => yourRenderFunction({ appContent, loading }),
    activeRule: location => yourActiveRule(location),
  },
  {
    name: 'vue app',
    entry: { scripts: ['//localhost:7100/main.js'] },
    render: ({ appContent, loading }) => yourRenderFunction({ appContent, loading }),
    activeRule: location => yourActiveRule(location),
  },
]);

start();
```

After the sub-application information is registered, the matching logic of the qiankun will be automatically triggered once the browser url changes, and all the render methods corresponding to the subapplications whose activeRule methods returns `true` will be called, at the same time the subapplications' exposed lifecycle hooks will be called in turn.

Usually, the render and activeRule configurations of our sub-applications can be extracted into public methods. Take react as an example:

```ts
import { registerMicroApps, start } from 'qiankun';

function render({ appContent, loading }) {
  const container = document.getElementById('container');
  ReactDOM.render(<Framework loading={loading} content={appContent} />, container);
}

function genActiveRule(routerPrefix) {
  return location => location.pathname.startsWith(routerPrefix);
}

registerMicroApps([
  { name: 'react app', entry: '//localhost:7100', render, activeRule: genActiveRule('/react') },
  { name: 'vue app', entry: { scripts: ['//localhost:7100/main.js'] }, render, activeRule: genActiveRule('/vue') },
]);

start();
```

## Exports Lifecycles From Sub App Entry

The child application needs to export `bootstrap`,`mount`, `unmount` three lifecycle hooks in its own entry js (usually the entry js of webpack you configure) for the main application to call at the appropriate time.

```ts
/**
 * The bootstrap will only be called once when the child application is initialized.
 * The next time the child application re-enters, the mount hook will be called directly, and bootstrap will not be triggered repeatedly.
 * Usually we can do some initialization of global variables here,
 * such as application-level caches that will not be destroyed during the unmount phase.
 */
export async function bootstrap() {
  console.log('react app bootstraped');
}

/**
 * The mount method is called every time the application enters,
 * usually we trigger the application's rendering method here.
 */
export async function mount(props) {
  console.log(props);
  ReactDOM.render(<App />, document.getElementById('react15Root'));
}

/**
 * Methods that are called each time the application is switched/unloaded,
 * usually in this case we uninstall the application instance of the subapplication.
 */
export async function unmount() {
  ReactDOM.unmountComponentAtNode(document.getElementById('react15Root'));
}
```

As qiankun based on single-spa, you can find more documentation about the sub-application lifecycle [here](https://single-spa.js.org/docs/building-applications.html#registered-application-lifecycle).

## Config Sub App Bundler

In addition to exposing the corresponding life-cycle hooks in the code, in order for the main application to correctly identify some of the information exposed by the sub-application, the sub-application bundler needs to add the following configuration:

#### webpack:

```js
const packageName = require('./package.json').name;

module.exports = {
  output: {
    library: `${packageName}-[name]`,
    libraryTarget: 'umd',
    jsonpFunction: `webpackJsonp_${packageName}`,
  },
};
```

You can check the configuration description from [webpack doc](https://webpack.js.org/configuration/output/#outputlibrary)。

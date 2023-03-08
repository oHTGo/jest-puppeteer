# 🎪 jest-puppeteer

[![npm version](https://img.shields.io/npm/v/jest-puppeteer.svg)](https://www.npmjs.com/package/jest-puppeteer)
[![npm dm](https://img.shields.io/npm/dm/jest-puppeteer.svg)](https://www.npmjs.com/package/jest-puppeteer)
[![npm dt](https://img.shields.io/npm/dt/jest-puppeteer.svg)](https://www.npmjs.com/package/jest-puppeteer)

## Installation

### Install packages

```bash
npm install --save-dev jest-puppeteer puppeteer jest
```

### Update your Jest configuration

```json
{
  "preset": "jest-puppeteer"
}
```

> **Note**
> Be sure to remove any existing `testEnvironment` option from your Jest configuration.

### Write tests

```js
import "expect-puppeteer";

describe("Google", () => {
  beforeAll(async () => {
    await page.goto("https://google.com");
  });

  it('should display "google" text on page', async () => {
    await expect(page).toMatchTextContent("google");
  });
});
```

## Running puppeteer in CI environments

Most continuous integration platforms limit the number of threads one can use. If you have more than one test suite running puppeteer chances are that your test will timeout. This is because jest will try to run puppeteer in parallel and the CI platform won't be able to handle all the parallel jobs in time. A fix to this is to run your test serially when in a CI environment. Users have discovered that [running test serially in such environments can render up to 50%](https://jestjs.io/docs/en/troubleshooting#tests-are-extremely-slow-on-docker-and-or-continuous-integration-ci-server) of performance gains.

This can be achieved through the CLI by running:

```sh
jest --runInBand
```

Alternatively, you can set jest to use as a max number of workers the amount that your CI environment supports:

```
jest --maxWorkers=2
```

## Recipes

### TypeScript

TypeScript is natively supported from v8.0.0, for previous versions, you have to use [types provided by the community](https://github.com/DefinitelyTyped/DefinitelyTyped).

### Writing tests using Puppeteer

Writing integration test can be done using [Puppeteer API](<(https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md)>) but it can be complicated and hard because API is not designed for testing.

To make it simpler, [expect-puppeteer API](https://github.com/smooth-code/jest-puppeteer/tree/master/packages/expect-puppeteer/README.md#api) add some specific matchers if you make expectation on a [Puppeteer Page](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#class-page).

Some examples:

#### Find a text in the page

```js
// Assert that current page contains 'Text in the page'
await expect(page).toMatchTextContent("Text in the page");
```

#### Click a button

```js
// Assert that a button containing text "Home" will be clicked
await expect(page).toClick("button", { text: "Home" });
```

#### Fill a form

```js
// Assert that a form will be filled
await expect(page).toFillForm('form[name="myForm"]', {
  firstName: "James",
  lastName: "Bond",
});
```

### Put in debug mode

Debugging tests can be hard sometimes and it is very useful to be able to pause tests in order to inspect the browser. Jest Puppeteer exposes a method `jestPuppeteer.debug()` that suspends test execution and gives you opportunity to see what's going on in the browser.

```js
await jestPuppeteer.debug();
```

### Start a server

Jest Puppeteer integrates a functionality to start a server when running your test suite. It automatically closes the server when tests are done.

To use it, specify a server section in your `jest-puppeteer.config.cjs`.

```js
// jest-puppeteer.config.cjs

/** @type {import('jest-environment-puppeteer').JestPuppeteerConfig} */
module.exports = {
  server: {
    command: "node server.js",
    port: 4444,
  },
};
```

Other options are documented in [jest-dev-server](https://github.com/smooth-code/jest-puppeteer/tree/master/packages/jest-dev-server).

### Configure Puppeteer

Jest Puppeteer automatically detects the best config to start Puppeteer but sometimes you may need to specify custom options.

To run Puppeteer on Firefox, you can set the `launch.product` property to `firefox`. By default, the value is `chrome` which will use Puppeteer on Chromium.

The browser context can be also specified. By default, the browser context is shared (value of `default`). The `incognito` value is also available, in case you want more isolation between running instances. More information available in [jest-puppeteer-environment readme](https://github.com/smooth-code/jest-puppeteer/blob/master/packages/jest-environment-puppeteer/README.md)

Default config values:

```js
// jest-puppeteer.config.cjs

/** @type {import('jest-environment-puppeteer').JestPuppeteerConfig} */
module.exports = {
  launch: {
    dumpio: true,
    headless: process.env.HEADLESS !== "false",
    product: "chrome",
  },
  browserContext: "default",
};
```

### Configure ESLint

Jest Puppeteer exposes five globals: `browser`, `page`, `context`, `puppeteerConfig` and `jestPuppeteer`. If you want to avoid errors, you can add them in your ESLint config:

```js
// .eslintrc.js
module.exports = {
  env: {
    jest: true,
  },
  globals: {
    page: true,
    browser: true,
    context: true,
    puppeteerConfig: true,
    jestPuppeteer: true,
  },
};
```

### Custom `setupTestFrameworkScriptFile` or `setupFilesAfterEnv`

If you use custom setup files, you'll need to include `expect-puppeteer` yourself in order to use the matchers it provides. Add the following to your setup file.

```js
// setup.js
require("expect-puppeteer");

// Your custom setup
// ...
```

```js
// jest.config.js
module.exports = {
  // ...
  setupTestFrameworkScriptFile: "./setup.js",
  // or
  setupFilesAfterEnv: ["./setup.js"],
};
```

You may want to consider using multiple projects in Jest since setting your own `setupFilesAfterEnv` and `globalSetup` can cause globals to be undefined.

```js
module.exports = {
  projects: [
    {
      displayName: "integration",
      preset: "jest-puppeteer",
      transform: {
        "\\.tsx?$": "babel-jest",
        ".+\\.(css|styl|less|sass|scss|png|jpg|ttf|woff|woff2)$":
          "jest-transform-stub",
      },
      moduleNameMapper: {
        "^.+\\.(css|styl|less|sass|scss|png|jpg|ttf|woff|woff2)$":
          "jest-transform-stub",
      },
      modulePathIgnorePatterns: [".next"],
      testMatch: [
        "<rootDir>/src/**/__integration__/**/*.test.ts",
        "<rootDir>/src/**/__integration__/**/*.test.tsx",
      ],
    },
    {
      displayName: "unit",
      transform: {
        "\\.tsx?$": "babel-jest",
        ".+\\.(css|styl|less|sass|scss|png|jpg|ttf|woff|woff2)$":
          "jest-transform-stub",
      },
      moduleNameMapper: {
        "^.+\\.(css|styl|less|sass|scss|png|jpg|ttf|woff|woff2)$":
          "jest-transform-stub",
      },
      globalSetup: "<rootDir>/setupEnv.ts",
      setupFilesAfterEnv: ["<rootDir>/setupTests.ts"],
      modulePathIgnorePatterns: [".next"],
      testMatch: [
        "<rootDir>/src/**/__tests_/**/*.test.ts",
        "<rootDir>/src/**/__tests__/**/*.test.tsx",
      ],
    },
  ],
};
```

### Extend `PuppeteerEnvironment`

Sometimes you want to use your own environment, to do that you can extend `PuppeteerEnvironment`.

First, create your own js file for custom environment.

```js
// custom-environment.js
const PuppeteerEnvironment = require("jest-environment-puppeteer");

class CustomEnvironment extends PuppeteerEnvironment {
  async setup() {
    await super.setup();
    // Your setup
  }

  async teardown() {
    // Your teardown
    await super.teardown();
  }
}

module.exports = CustomEnvironment;
```

Then, assigning your js file path to the [`testEnvironment`](https://facebook.github.io/jest/docs/en/configuration.html#testenvironment-string) property in your Jest configuration.

```js
{
  // ...
  "testEnvironment": "./custom-environment.js"
}
```

Now your custom `setup` and `teardown` will be triggered before and after each test suites.

### Create your own `globalSetup` and `globalTeardown`

It is possible to create your own [`globalSetup`](https://facebook.github.io/jest/docs/en/configuration.html#globalsetup-string) and [`globalTeardown`](https://facebook.github.io/jest/docs/en/configuration.html#globalteardown-string).

For this use case, `jest-environment-puppeteer` exposes two methods: `setup` and `teardown`, so that you can wrap them with your own global setup and global teardown methods as the following example:

```js
// global-setup.js
const setupPuppeteer = require("jest-environment-puppeteer/setup");

module.exports = async function globalSetup(globalConfig) {
  await setupPuppeteer(globalConfig);
  // Your global setup
};
```

```js
// global-teardown.js
const teardownPuppeteer = require("jest-environment-puppeteer/teardown");

module.exports = async function globalTeardown(globalConfig) {
  // Your global teardown
  await teardownPuppeteer(globalConfig);
};
```

Then assigning your js file paths to the [`globalSetup`](https://facebook.github.io/jest/docs/en/configuration.html#globalsetup-string) and [`globalTeardown`](https://facebook.github.io/jest/docs/en/configuration.html#globalteardown-string) property in your Jest configuration.

```js
{
  // ...
  "globalSetup": "./global-setup.js",
  "globalTeardown": "./global-teardown.js"
}
```

Now your custom `globalSetup` and `globalTeardown` will be triggered once before and after all test suites.

### Create React App

You can find an [example of create-react-app setup in this repository](https://github.com/smooth-code/jest-puppeteer/tree/master/examples/create-react-app).

## API

### `global.browser`

Give access to the [Puppeteer Browser](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#class-browser).

```js
it("should open a new page", async () => {
  const page = await browser.newPage();
  await page.goto("https://google.com");
});
```

### `global.page`

Give access to a [Puppeteer Page](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#class-page) opened at start (you will use it most of time).

```js
it("should fill an input", async () => {
  await page.type("#myinput", "Hello");
});
```

### `global.context`

Give access to a [browser context](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#class-browsercontext) that is instantiated when the browser is launched. You can control whether each test has its own isolated browser context using the `browserContext` option in your config file.

### `global.expect(page)`

Helper to make Puppeteer assertions, [see documentation](https://github.com/smooth-code/jest-puppeteer/tree/master/packages/expect-puppeteer/README.md#api).

```js
await expect(page).toMatchTextContent("A text in the page");
// ...
```

### `global.jestPuppeteer.debug()`

Put test in debug mode.

- Jest is suspended (no timeout)
- A `debugger` instruction to Chromium, if Puppeteer has been launched with `{ devtools: true }` it will stop

```js
it("should put test in debug mode", async () => {
  await jestPuppeteer.debug();
});
```

### `global.jestPuppeteer.resetPage()`

Reset global.page

```js
beforeEach(async () => {
  await jestPuppeteer.resetPage();
});
```

### `global.jestPuppeteer.resetBrowser()`

Reset global.browser, global.context, and global.page

```js
beforeEach(async () => {
  await jestPuppeteer.resetBrowser();
});
```

### Config

Jest Puppeteer uses [cosmiconfig](https://github.com/davidtheclark/cosmiconfig) for configuration file support. This means you can configure Jest Puppeteer via (in order of precedence):

- A `"jest-puppeteer"` key in your `package.json` file.
- A `.jest-puppeteerrc` file written in JSON or YAML.
- A `.jest-puppeteerrc.json`, `.jest-puppeteerrc.yml`, `.jest-puppeteerrc.yaml`, or `.jest-puppeteerrc.json5` file.
- A `.jest-puppeteerrc.js`, `.jest-puppeteerrc.cjs`, `jest-puppeteer.config.js`, or `jest-puppeteer.config.cjs` file that exports an object using `module.exports`.
- A `.jest-puppeteerrc.toml` file.

By default it looks for config at the root of the project. You can define a custom path using `JEST_PUPPETEER_CONFIG` environment variable.

It should export a config object or a Promise that returns a config object.

```ts
interface JestPuppeteerConfig {
  /**
   * Puppeteer connect options.
   * @see https://pptr.dev/api/puppeteer.connectoptions
   */
  connect?: ConnectOptions;
  /**
   * Puppeteer launch options.
   * @see https://pptr.dev/api/puppeteer.launchoptions
   */
  launch?: PuppeteerLaunchOptions;
  /**
   * Server config for `jest-dev-server`.
   * @see https://www.npmjs.com/package/jest-dev-server
   */
  server?: JestDevServerConfig | JestDevServerConfig[];
  /**
   * Allow to run one browser per worker.
   * @default false
   */
  browserPerWorker?: boolean;
  /**
   * Browser context to use.
   * @default "default"
   */
  browserContext?: "default" | "incognito";
  /**
   * Exit on page error.
   * @default true
   */
  exitOnPageError?: boolean;
  /**
   * Use `runBeforeUnload` in `page.close`.
   * @see https://pptr.dev/api/puppeteer.page.close
   * @default false
   */
  runBeforeUnloadOnClose?: boolean;
}
```

#### Sync config

```js
// jest-puppeteer.config.cjs

/** @type {import('jest-environment-puppeteer').JestPuppeteerConfig} */
module.exports = {
  launch: {
    dumpio: true,
    headless: process.env.HEADLESS !== "false",
  },
  server: {
    command: "node server.js",
    port: 4444,
    launchTimeout: 10000,
    debug: true,
  },
};
```

#### Async config

This example uses an already running instance of Chrome by passing the active web socket endpoint to `connect`. This is useful, for example, when you want to connect to Chrome running in the cloud.

```js
// jest-puppeteer.config.cjs
const dockerHost = "http://localhost:9222";

async function getConfig() {
  const data = await fetch(`${dockerHost}/json/version`).json();
  const browserWSEndpoint = data.webSocketDebuggerUrl;
  /** @type {import('jest-environment-puppeteer').JestPuppeteerConfig} */
  return {
    connect: {
      browserWSEndpoint,
    },
    server: {
      command: "node server.js",
      port: 3000,
      launchTimeout: 10000,
      debug: true,
    },
  };
}

module.exports = getConfig();
```

## Inspiration

Thanks to Fumihiro Xue for his great [Jest example](https://github.com/xfumihiro/jest-puppeteer-example).

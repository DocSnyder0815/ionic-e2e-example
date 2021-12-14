# Ionic E2E Example

This example app demonstrates how to build web and native end-to-end (E2E) tests with Ionic and Cordova or Capacitor. This example uses popular tools like [WebdriverIO](https://webdriver.io) and [Appium](https://appium.io) to enable cross-platform tests for iOS, Android, Web, and more.

Additionally, this example comes with some helpers that make it easier to write tests against Ionic or hybrid apps in general.

## About the Testing Stack

We've chosen WebdriverIO as the primary test runner and API for test authoring. WebdriverIO is the leading Node.js-based test automation framework and supports a wide variety of tools that support the WebDriver protocol.

WebdriverIO orchestrates tools like Appium and Chromedriver to actually run tests on target devices or a web browser.

One of the benefits of this stack compared to popular tools like [Cypress.io](https://cypress.io) is that it can test your actual native app, the same app that you'll ship to end users, but with a similar test authoring API.

## Exploring the Tests

Explore the [tests](https://github.com/ionic-team/ionic-e2e-example/tree/main/tests) folder to find a typical [Page Object Pattern](https://webdriver.io/docs/pageobjects/) test layout. In the `pageobjects` folder we find a typescript file corresponding to every logical page in our app. A Page Object is a testing abstraction over a page that defines properties and methods that we will interact with in our test specs.

<p align="center">
  <img src="docs/tutorial-page.png" width="320" />
</p>

For example, see the [Tutorial](https://github.com/ionic-team/ionic-e2e-example/blob/main/tests/pageobjects/tutorial.page.ts) Page Object, which defines the `slides`, `skipButton`, and `continueButton` properties corresponding to elements on the page that we will interact with in our test specs. Additionally, we define the `swipeLeft()`, `swipeRight()`, `skip()`, and `continue()` methods which are actions we will take against the page in our test specs.

```typescript
import { IonicButton, IonicSlides } from "../helpers";
import Page from "./page";

class Tutorial extends Page {
  get slides() {
    return new IonicSlides("swiper");
  }
  get skipButton() {
    return IonicButton.withTitle("Skip");
  }
  get continueButton() {
    return IonicButton.withTitle("Continue");
  }

  async swipeLeft() {
    return this.slides.swipeLeft();
  }

  async swipeRight() {
    return this.slides.swipeRight();
  }

  async skip() {
    return this.skipButton.tap();
  }

  async continue() {
    await this.continueButton.tap();
  }
}

export default new Tutorial();
```

We see that this is a simple representation of our page, exposing components and methods that our tests will need to interact with, and nothing more.

In the `specs` folder we find test specs corresponding to each page, and this is where our actual test assertions live. Let's explore the [Tutorial Test Spec](https://github.com/ionic-team/ionic-e2e-example/blob/main/tests/specs/app.tutorial.spec.ts) to see the actual tests performed against the Tutorial Page Object.

```typescript
import {
  clearIndexedDB,
  pause,
  getUrl,
  url,
  setDevice,
  switchToWeb,
  Device,
  waitForLoad,
} from "../helpers";

describe("tutorial", () => {
  before(async () => {
    await waitForLoad();
  });

  beforeEach(async () => {
    await switchToWeb();
    await url("/tutorial");
    await setDevice(Device.Mobile);
    await clearIndexedDB("_ionicstorage");
  });

  it("Should get to schedule", async () => {
    await Tutorial.swiper.swipeLeft();
    await Tutorial.swiper.swipeLeft();
    await Tutorial.swiper.swipeLeft();

    await Tutorial.continue();

    await pause(1000);

    await expect((await getUrl()).pathname).toBe("/app/tabs/schedule");
  });

  /* ... more tests ... */
});
```

In this test spec we first wait for the webview to load before we run any tests (that's our `waitForLoad` helper), next before each test we do four things: Switch to the Web View context, set the current url to `/tutorial`, set the device to mobile, and then clear IndexedDB since this page uses it to store a flag indicating whether the tutorial was finished.

For the test shown above, note that we merely interact with the methods available on the Page Object for the Tutorial page. We `swipeLeft()` three times which progresses the slides, and then we `continue()`. We wait one second for the client-side navigation to occur, then verify we navigated to the `/app/tabs/schedule` page.

Using this strategy we can build tests for any of our pages while keeping our test specs short and not reliant on DOM structure or any details that could break with small design or layout changes in the app.

## Test Helpers

This example comes with a number of test helpers specifically meant for testing hybrid apps and those using Ionic's UI components. Some of these helpers merely wrap [WebdriverIO API calls](https://webdriver.io/docs/api) to make them simpler, but some specifically operate on the unique structure of Ionic apps and are important to use.

### Querying Elements

For developers using Ionic's UI components and Ionic Angular/React/Vue, the most important helper is `Ionic$` and the related Ionic components in [helpers/ionic/components](https://github.com/ionic-team/ionic-e2e-example/tree/main/tests/helpers/ionic/components).

Ionic is highly optimized for performance, and because of that it does a few things that can trick the built-in `$` and `$$` queries, since elements might be in the DOM but are not visible, and querying for elements without considering this can cause tests to fail.

Instead, when using Ionic, always use the `Ionic$` helper to query elements, or one of the special Ionic components as listed below.

```typescript
import { Ionic$ } from "../helpers";

await Ionic$.$("#one-element");
await Ionic$.$$(".multiple-elements");
```

### Ionic Component Helpers

## Everything is Async

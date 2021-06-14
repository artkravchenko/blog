# 🚧 Lessons learned from end-to-end testing with TestCafe

This is a deep dive into the workflow with TestCafe, an overview of pain points, proposed solutions, and clarification of things that are missing or insufficiently described in the documentation.

Contents:

- [what to know if you're integrating end-to-end testing at the first time](#what-to-know-if-youre-integrating-end-to-end-testing-at-the-first-time)
- [treat tests as carefully as the application's codebase](#treat-tests-as-carefully-as-the-applications-codebase)
- [don't rely on implementation details in tests too much](#dont-rely-on-implementation-details-in-tests-too-much)
- 🚧 don't rely on unstable test data
- 🚧 [use Page Object pattern](#-use-page-object-pattern)
- 🚧 [TestCafe APIs to use with caution](#-testcafe-apis-to-use-with-caution)
- 🚧 how to find elements on the page: nuances of `Selector`
- 🚧 debug effectively
- 🚧 how to speed up test execution
- 🚧 [useful extensions to TestCafe and recipes](#-useful-extensions-to-testcafe-and-recipes)
- 🚧 [further reading](#-further-reading)

The article is continually updated. [Leave a comment](https://github.com/artkravchenko/blog/discussions/2). Give any feedback or ask questions. Let me know your use cases!

🚧 means "work in progress" i.e. sections are incomplete.

___

## What to know if you're integrating end-to-end testing at the first time

Although this article will hopefully make your TestCafe journey easier, you'll save another ton of time if you get organizational and foundational things done first:

- what you want to achieve with end-to-end testing
- alternatives to automated end-to-end testing
- areas of application to cover with end-to-end tests
- testing strategy and plan
- how to prepare stable test data

I'm going to publish a separate article on that.

<!-- TODO: attach a link to the article -->

## Treat tests as carefully as the application's codebase

### In approach to writing tests

- readability is one of the top priorities since automated testing pays off compared to manual testing if tests live long (year and more) almost untouched and if they can be easily updated by other people
- tests should be stable; otherwise they will hardly be worth writing; stability doesn't come out of nowhere and requires test design, data preparation, and bullet-proof implementation
- fragmentation is one of the things you should avoid the most; use the same approach, styleguide, tools, and utilities in the team, and continually improve them all
- keep complexity within clear boundaries rather than spreading it across the codebase

### In tooling

#### TypeScript

<!--
- E2E tests are slow in general, so getting errors right away at compile-time in the editor will save you a lot of time
- you're testing the application, so it could be good to prevent type-related bugs in the testing codebase should be avoided
- the rest benefits are similar to JavaScript vs. TypeScript comparisons
- if you haven't tried TypeScript yet in the application codebase, it could be good to start with it in the testing codebase
  - TypeScript can be integrated file-by-file, so it isn't an issue for the application codebase as well
  - if you're creating a testing codebase, it could be good to start with TypeScript from scratch
-->

- accelerates writing tests
  - catches typos and type-related issues at compile-time rather than at runtime
  - provides a nice autocomplete in editors such as VS Code
    - so you wouldn't have to remember every single method of TestCafe or your utilities
- increases test maintainability
  - serves as up-to-date code documentation
  - enforces safer development practices thus making code more predictable and understandable
  - supports modeling complex domains due to strongly-typed OOP and generics, compared to plain JavaScript

<!--
🚧 **TODO:** check whether DX is good with `tsc` + JSDoc instead of TypeScript. For multidisciplinary teams which don't have experience in strongly-typed programming languages, JSDoc might be simpler to start, yet providing basic type-checking and giving auto-completion in editors.
- https://medium.com/@trukrs/type-safe-javascript-with-jsdoc-7a2a63209b76
- https://www.mobiquity.com/insights/typescript-to-javascript-with-jsdoc-and-back
-->

#### ESLint

- accelerates writing tests by catching basic issues at compile-time (or rather writing time), not after minutes of debugging
  - it catches more than just formatting issues: it can catch missing `await`, presence of `any`, or an accidentally skipped test
- increases test maintainability by enforcing a consistent code style

#### Prettier

- accelerates writing tests by automating the code formatting
- increases development speed by eliminating debates around code formatting
- just like ESLint, it also increases test maintainability by enforcing a consistent code style

## Don't rely on implementation details in tests too much

Be careful with negative assertions, [because they can pass for the wrong reason](https://glebbahmutov.com/blog/negative-assertions/). Prefer positive assertions. For example, if you want to make sure that the table has been loaded, it's generally better to check that at least one row is visible rather than checking that the loading indicator (e.g. skeleton, spinner, or load bar) is hidden.

Avoid making assumptions about the implementation of AUT (application under test), prefer checking what's visible to the end user. When you rely on implementation details such as specific HTML attributes or network requests, keep such logic in one place, under a facade. Otherwise, when the implementation changes, you'll have to go through each affected test and update it separately.

## 🚧 Don't rely on unstable test data
## 🚧 Use Page Object pattern

I strongly suggest reading these 3 articles first:

- [PageObject](https://martinfowler.com/bliki/PageObject.html) by Martin Fowler
- [Page object models](https://www.selenium.dev/documentation/en/guidelines_and_recommendations/page_object_models/) in Selenium documentation
- [Domain specific language](https://www.selenium.dev/documentation/en/guidelines_and_recommendations/domain_specific_language/) in Selenium documentation

These articles do a great job describing what Page Object is and shape its usage scenarios. I'll focus on differences with other approaches and provide real-world examples with TestCafe.

Reading [Page Model](https://devexpress.github.io/testcafe/documentation/guides/concepts/page-model.html) article in TestCafe documentation alone is not enough, because it lacks explanation of the pattern. That doesn't draw the whole picture, and I discovered that it led people to wrong assumptions about the pattern.

### Drawbacks of putting `Selector`s in the tests

Let's assume that we assert a text field based on Material UI's [`TextField`](https://material-ui.com/components/text-fields/). It might have a label, value, placeholder, helper text, contextual action, required state, disabled state, error state, focused state, and loading state. It's a fairly basic UI element, yet it has lots of aspects.

When we rely on how the text field expresses its state in the DOM in each test, we spread the text field's implementation details across the codebase. This increases cognitive complexity and makes onboarding more difficult. Sometimes these states can't be retrieved in one line of code, but even when they can, it generally makes tests less readable, especially for another person after half a year.

You might have lots of different components, and each of them expresses the state in a special way. You have to keep in mind or double-check all such details while writing tests. When you've made a mistake, most of the time you find this out after tens of seconds or even minutes, not immediately.

When we inline implementation details instead of abstracting them, we tend to choose the shortest path to solve a particular task. It makes code very specific to the task it solves. As a result, we make each piece of code unique, make the codebase fragmented, and give up code reusability. It also reduces space for optimizations of any kind in both tests and the application.

When implementation details change, we have to find each affected test and fix it. It's a matter of "when", not "if". When your tests live long, they face breaking changes of libraries, migration to other libraries, introducing additional libraries for special cases, and different people. It's beneficial to leave some room for these long-term events.

### 🚧 Drawbacks of using helper functions instead of Page Object

### 🚧 TestCafe specifics to Page Object pattern

### 🚧 Suggested codestyle

## 🚧 TestCafe APIs to use with caution

### `ClientFunction`

[Documentation](https://devexpress.github.io/testcafe/documentation/reference/test-api/clientfunction/).

- ...

### `Selectors`'s custom properties and methods

[Documentation](https://devexpress.github.io/testcafe/documentation/guides/basic-guides/select-page-elements.html#extend-selectors-with-custom-properties-and-methods).

Custom property getters and methods are essentially `ClientFunction`s with even more limitations:

- TypeScript typedefs are verbose and flaky, there's almost no type-safety
- no way to pass dependencies from the outer scope, only function arguments are supported

When you want to extend `Selector`, you typically model something specific to your UI or domain model e.g. a date picker. You probably want to interact with this element or its children or even other elements on the page, or simply retrieve data from them.

For these purposes, use Page Object instead.

<details>
  <summary>🚧 Example with <code>Selector::addCustomMethods()</code></summary>
  
  ```typescript
import { Selector } from 'testcafe';

interface CalendarSelector extends Selector {
    getDay: (monthName: string) => Selector;
}

function getCalendar(): CalendarSelector {
    const calendar = Selector('[data-testid="calendar"]');

    // TODO: check whether we need to use the return value of `addCustomMethods` or it assigns methods to `this` as well
    calendar.addCustomMethods({
        // TODO: check whether we're actually alowed to return `Element[]` from here
        getDay(calendarsRaw, monthName: string): Element[] {
            // `calendarsRaw` is `Element`, but according to documentation, it's `Element[]` if `returnDOMNodes: true`
            // https://devexpress.github.io/testcafe/documentation/reference/test-api/selector/addcustommethods.html
            const calendars = calendarsRaw as unknown as Element[];

            const months = calendars.flatMap(calendar => {
                const allMonths = Array.from(calendar.querySelectorAll('[data-testid="month"]'));
                const matchingMonths = allMonths.filter(month => (month as HTMLElement).innerText.includes(monthName));
                return matchingMonths;
            });
            const days = months.flatMap(month => {
                const allDays = Array.from(month.querySelectorAll('[data-testid="day"]'));
                return allDays;
            });

            return days;
        },
    }, { returnDOMNodes: true });

    return calendar as CalendarSelector;
}

test('allows a user to select date on a calendar', async t => {
    const calendar = getCalendar();
    const day = calendar.getDay('January').withExactText('1');
    await t.click(day);
});
  ```

  [Playground link](https://www.typescriptlang.org/play?target=99#code/JYWwDg9gTgLgBAbzgZQKYBtUGMbTgXzgDMoIQ4ByGVAZxiwEMjUKBuAKHeADtqoiGWVHADCDTNwAmDKGkw48qAB7UpNFBmy4oidnH1wA5qhgARBgE8AXHAAUICLwAWAOQYhUNulB6GAlHAAvAB8GvLaHPicRACu3DjAjkYmYhLSULZ+NqmoUjJyWngIegZYjnRwjGkyQWGFGRQA2tIwDAC01HTAkoEARFW56b0AuhR+HCX6APRTcAAqAPKmCzZYTtgA1nAA7usw6zrbwtyoqJJwuHAxNML7wlAmMVDccABu4jHCEERwAAYMkkkImuuBAAFkTE4IJIaL84HhgPAGDQaMBDNx1B59tD1JdfvtgLC4Midhh0JNKuJBjIAHQAoEgsgQ7Ew2zFAwcuAzeZLFaVdZYLa7SGoQ4sB7EnAxcToCzE9AQI7nS4PGBPF6-ACimA8vEawzhJDIcAOqApHOMZkstgGeSgNAASgxtgAaOAOZxuDxeGA+bj+Gza1C6mD63SciNc2a-W3pR3OuGEv5BkO-N0AIxiSKwZSgkl8FwgcEkECwMRDDBgiW4bsRFHUWp1uVDBrgwB+v1V6uWYJc0NoNl9n1+5sj3KcMBgYBoVhmklQr2UYAeKJphkRThi6ZpiSmnXoTFQUxLZYrVccUwezAe8SP+7aDDAwCmN3C0Cm9LLdDIWKhMJpE4gOSkacmUGLwLGMjqIElLVPaTrbMS6hxBs3CKi8JIps2+oTCBHJgRUHr7NBsHUvaNJEOglZgo+NpUnaQShOyeERgRSLoOgYKOMRtQAIJQFAlgUaQIB0XBNIAI6fFAFgFAoUC8RxthNC07T7t0fREU4IxjOMo4sWx7qVmsvhcc4JEymZxEUcA6B8PY3FOIxdhaUhcAABJzGCAAyWG8H4O7cCcUBzMoMCBVg6AxPONAOZ67ioH4eksRGXbPEZ9BOKZjk0BwKUEMlKWGdIFgkVpNAUVRMA0WAcX7M5zH5fohkyuYpV8QJQlGqJWmSdJsmaPJinoMpzSVmptBVj0vQlTpSV5U1+hpRhHFtbl+mcvgekbRyy3FpY60gfgLoUsdiBwMtPZ9jFg5QJ8BW4QYe2QToJI5HackROwUTsPuykyoq6gMFcNw6Jcr5aPt1DwhhpF2hQbrIhY8QXA1FKGS9tSWu96SZAtzXlPAJW1C9a4mG1ykAFIMNw0oyWMNLbBumpKIIMChSoykAIxjPjxLbAwiIXDSkXAIKtglXpW2sEAA)
</details>

<details>
  <summary>🚧 Example with Page Objects</summary>

  ```typescript
import { Selector } from 'testcafe';

class Calendar {
    public readonly root: Selector;
    public readonly month: Selector;

    constructor() {
        this.root = Selector('[data-testid="calendar"]');
        this.month = this.root.find('[data-testid="month"]');
    }

    public getDay(monthName: string): Selector {
        const matchingMonth = this.month.withText(monthName);
        const day = matchingMonth.find('[data-testid="day"]');
        return day;
    }
}

test('allows a user to select date on a calendar', async t => {
    const calendar = new Calendar();
    const day = calendar.getDay('January').withExactText('1');
    await t.click(day);
});
  ```

  [Playground link](https://www.typescriptlang.org/play?target=99&ssl=1&ssc=1&pln=23&pc=4#code/JYWwDg9gTgLgBAbzgZQKYBtUGMbTgXzgDMoIQ4ByGVAZxiwEMjUKBuAKHa3QZprgDCDTADsAJgyiJ2cWXDABXAEbpgWOFFQMxEEegCeGiBBgAuFBmy4oHOfOWr1m7boNwQumAAtzaTDmgOGTksXTooBQCoAAoASmk7O29gGgA6UhM4AF4Lf2toigBtCRgGAFpqOmAxLIAiRlEJKFqAXQpY20TZZLSPEW9suB704xhUomBxAuKGUoraGGq6vu9W9s7ZfE47RRU1OABzVBgAEQZ9aJWvADkGEFRzcMmD2N9LKISuuFCROndZrBeZ4AWU8XkGwyuqQA7sBvAAVVAADxglzBt3uHWCXR+fwkhhyIABQJEB1B-S840mYmmJXKlUWNVq+LWWK+GmOCigIjg+I2BHYW3YDIKwnQEGh-AYcAUNFQUlwcDleV5s1QcF0cGlDVQ4kkFAANFqaPoROp4FkAHyfWS4+A6vVSHIiVDQwTCXVNOL8u2qgnfD2O1JHU7nAoAKQYIgUkn07RhcK8AFEkQwcIiUQUAIzrbFa6EMOFDVLcNQAa2i+Kx+A6QA)
</details>

Summary:

- no way to pass dependencies from the outer scope, unlike `ClientFunction`
  - so there's no way to compose multiple extended selectors
- custom methods don't support element manipulation inside e.g. calling `t.click()`, just like `ClientFunction`
- TypeScript typedefs are verbose and flaky, type-safety is limited
- note that the property getter or method's first argument (`node`) may be `null` if the element doesn't exist, even though `testcafe` TypeScript typedefs say otherwise

### Further reading

- [[docs] Describe TestCafe current limitations · Issue #5196 · DevExpress/testcafe](https://github.com/DevExpress/testcafe/issues/5196)

## 🚧 Debug effectively

What we typically want to debug:

- selectors which don't match any existing elements
- failing assertions or actions

How it might look like in an ideal world:

- we can enter the debugger, pause in the breakpoint, and interact with the failed Selector or write a new one via debugger console to highlight all matching elements
- we can rerun, adjust or skip each failed step or statement without restarting the entire test

A typical flow in the real world right now:

- run tests with `--debug-on-fail` and non-headless browser e.g. `chrome`
- if a test fails, inspect the page via browser DevTools
  - go to the Elements tab and imitate failed `Selector` by querying the document manually e.g. `document.querySelectorAll('element')`
  - go to the Network tab
  - go to the Console tab to check uncaught exceptions
- to inspect the page from a point in time before the error, call `await t.debug();` in the test
- if there's `fixture.after` or `test.after` with UI manipulation inside, disable them for debugging because they are executed even if `--debug-on-fail` is activated

Current debugging feature set:

- ways to debug `Selector`:
  - read error messages
    - sometimes a chain of Selectors is displayed if Selector doesn't match any element
  - get a snapshot of underlying DOM Node, if `Selector` matches any, but only as a part of the test, not in the debugger console, so it's like `console.log` and restarting the test from scratch every time
  - write queries to browser DevTools
    - `document.querySelectorAll('element')`
    - this way we can imitate what we've written in tests with `Selector`
    - it seems to be the most effective way to debug `Selector` now
  - writing assertions from top-level to deeply nested elements
    - e.g. page -> section -> subsection -> another small element
    - this way you can quickly identify the error scope
- ways to debug failed assertions or actions:
  - no way to skip or re-run failed steps out of the box, requires patching TestCafe / its Babel plugins
  - you can enter debug mode the same way as above and execute failed actions manually by clicking on the page
  - assertions and actions typically fail because of Selectors, so the suggestions listed above can be applied here
- using `debugger` is supported, but not usable enough in the real world:
  - the browser will be disconnected if the browser script or test gets paused for more than 30 seconds:
  - progress: https://github.com/DevExpress/testcafe/issues/2819
- TestCafe APIs targeting the browser don't support execution in debugger console
  - no DOM Node snapshots supported
  - no Selector and ClientFunction execution supported
  - no `await t.eval()` supported
  - progress: https://github.com/DevExpress/testcafe/issues/3810

## 🚧 How to find elements on the page: nuances of `Selector`
## 🚧 How to speed up test execution
## 🚧 Useful extensions to TestCafe and recipes

### How to log all browser network requests per test
### How to fail tests by default if an error notification appears
### How to save all browser logs per test
### How to open the browser with the same viewport size on CI and locally
### How to generate tests dynamically
### How to integrate TestCafe with Bitbucket Pipelines
### How to configure tests and utilities based on user roles
### How to resolve TypeScript modules relative to the root to avoid `../../`
### How to perform multiple custom assertions in one step

## 🚧 Further reading

- [Conditional Testing | Cypress Documentation](https://docs.cypress.io/guides/core-concepts/conditional-testing)

# 🚧 Lessons learned from end-to-end testing with TestCafe

This is a deep dive into workflow with TestCafe, overview of pain points, proposed solutions, and clarification of things which are missing or insufficiently described in the documentation.

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

- readability is one of the top priorities since automated testing pays off compared to manual testing if tests live long (year and more) almost untounched and if they can be easily updated by other people
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
    - so you don't have to remember every single method in TestCafe or your own utilities
- increases test maintainability
  - serves as up-to-date code documentation
  - enforces safer development practices thus making code more predictable and understandable
  - supports modelling complex domains due to strongly-typed OOP and generics, compared to plain JavaScript

<!--
🚧 **TODO:** check whether DX is good with `tsc` + JSDoc instead of TypeScript. For multidisciplinary teams which don't have experience in strongly-typed programming languages, JSDoc might be simpler to start, yet providing basic type-checking and giving auto-completion in editors.
- https://medium.com/@trukrs/type-safe-javascript-with-jsdoc-7a2a63209b76
- https://www.mobiquity.com/insights/typescript-to-javascript-with-jsdoc-and-back
-->

#### ESLint

- accelerates writing tests by catching basic issues at compile-time (or rather writing time), not after minutes of debugging
  - it catches more than just formatting issues: it can catch missing `await`, presense of `any`, or an accidentally skipped test
- increases test maintainability by enforcing a consistent code style

#### Prettier

- accelerates writing tests by automating the code formatting
- increases development speed by eliminating debates around code formatting
- just like ESLint, it also increases test maintainability by enforcing a consistent code style

## Don't rely on implementation details in tests too much

Be careful with negative assertions, [because negative assertions can pass for the wrong reason](https://glebbahmutov.com/blog/negative-assertions/). Prefer positive assertions. For example, if you want to make sure that table has been loaded, it's generally better to check that at least one row is visible rather than checking that loading indicator (e.g. skeleton, spinner, or loadbar) is hidden.

Avoid making assumptions about implementation of AUT (application under test), prefer checking what's visible to the end user. When you rely on implementation details such as specific HTML attributes or network requests, keep such logic in one place, under a facade. Otherwise, when implementation changes, you'll have to go through each affected test and update it separately.

## 🚧 Don't rely on unstable test data
## 🚧 Use Page Object pattern

I strongly suggest reading these 3 articles first:

- [PageObject](https://martinfowler.com/bliki/PageObject.html) by Martin Fowler
- [Page object models](https://www.selenium.dev/documentation/en/guidelines_and_recommendations/page_object_models/) in Selenium documentation
- [Domain specific language](https://www.selenium.dev/documentation/en/guidelines_and_recommendations/domain_specific_language/) in Selenium documentation

These articles do a great job describing what Page Object is and shape its usage scenarios. I'll focus on differences with other approaches and provide real-world examples with TestCafe.

Reading [Page Model](https://devexpress.github.io/testcafe/documentation/guides/concepts/page-model.html) article in TestCafe documentation alone is not enough, because it lacks explanation of the pattern. That doesn't draw the whole picture, and I discovered that it led people to wrong assumptions about the pattern.

### Drawbacks of putting `Selector`s in the tests

Let's assume that we assert a text field based on Material UI's [`TextField`](https://material-ui.com/components/text-fields/). It might have label, value, placeholder, helper text, contextual action, required state, disabled state, error state, focused state, and loading state. It's a fairly basic UI element, yet it has lots of aspects.

When we rely on how text field expresses its state in the DOM in each test, we spread text field's implementation details across the codebase. This increases cognitive complexity and makes onboarding more difficult. Sometimes these states can't be retrieved in one line of code, but even when they can, it generally makes tests less readable, especially for another person after half a year.

You might have lots of different components, and each of them expresses the state in a special way. You have to keep in mind or double check all such details while writing tests. When you've made a mistake, most of the time you find this out after tens of seconds or even minutes, not immediately.

When we inline implementation details instead of abstracting them, we tend to choose the shortest path to solve a particular task. It makes code very specific to the task it solves. As a result, we make each piece of code unique, make the codebase fragmented, and give up code reusability. It also reduces space for optimizations of any kind in both tests and application.

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

- 🚧 [Example with `Selector::addCustomMethods()`](https://www.typescriptlang.org/play?target=99#code/JYWwDg9gTgLgBAbzgZQKYBtUGMbTgXzgDMoIQ4ByGVAZxiwEMjUKBuAKHeADtqoiGWVHADCDTNwAmDKGkw48qAB7UpNFBmy4oidnH1wA5qhgARBgE8AXHAAUICLwAWAOQYhUNulB6GAlHAAvAB8GvLaHPicRACu3DjAjkYmYhLSULZ+NqmoUjJyWngIegZYjnRwjGkyQWGFGRQA2tIwDAC01HTAkoEARFW56b0AuhR+HCX6APRTcAAqAPKmCzZYTtgA1nAA7usw6zrbwtyoqJJwuHAxNML7wlAmMVDccABu4jHCEERwAAYMkkkImuuBAAFkTE4IJIaL84HhgPAGDQaMBDNx1B59tD1JdfvtgLC4Midhh0JNKuJBjIAHQAoEgsgQ7Ew2zFAwcuAzeZLFaVdZYLa7SGoQ4sB7EnAxcToCzE9AQI7nS4PGBPF6-ACimA8vEawzhJDIcAOqApHOMZkstgGeSgNAASgxtgAaOAOZxuDxeGA+bj+Gza1C6mD63SciNc2a-W3pR3OuGEv5BkO-N0AIxiSKwZSgkl8FwgcEkECwMRDDBgiW4bsRFHUWp1uVDBrgwB+v1V6uWYJc0NoNl9n1+5sj3KcMBgYBoVhmklQr2UYAeKJphkRThi6ZpiSmnXoTFQUxLZYrVccUwezAe8SP+7aDDAwCmN3C0Cm9LLdDIWKhMJpE4gOSkacmUGLwLGMjqIElLVPaTrbMS6hxBs3CKi8JIps2+oTCBHJgRUHr7NBsHUvaNJEOglZgo+NpUnaQShOyeERgRSLoOgYKOMRtQAIJQFAlgUaQIB0XBNIAI6fFAFgFAoUC8RxthNC07T7t0fREU4IxjOMo4sWx7qVmsvhcc4JEymZxEUcA6B8PY3FOIxdhaUhcAABJzGCAAyWG8H4O7cCcUBzMoMCBVg6AxPONAOZ67ioH4eksRGXbPEZ9BOKZjk0BwKUEMlKWGdIFgkVpNAUVRMA0WAcX7M5zH5fohkyuYpV8QJQlGqJWmSdJsmaPJinoMpzSVmptBVj0vQlTpSV5U1+hpRhHFtbl+mcvgekbRyy3FpY60gfgLoUsdiBwMtPZ9jFg5QJ8BW4QYe2QToJI5HackROwUTsPuykyoq6gMFcNw6Jcr5aPt1DwhhpF2hQbrIhY8QXA1FKGS9tSWu96SZAtzXlPAJW1C9a4mG1ykAFIMNw0oyWMNLbBumpKIIMChSoykAIxjPjxLbAwiIXDSkXAIKtglXpW2sEAA)
- 🚧 [Example with Page Objects](https://www.typescriptlang.org/play?target=99&ssl=1&ssc=1&pln=23&pc=4#code/JYWwDg9gTgLgBAbzgZQKYBtUGMbTgXzgDMoIQ4ByGVAZxiwEMjUKBuAKHa3QZprgDCDTADsAJgyiJ2cWXDABXAEbpgWOFFQMxEEegCeGiBBgAuFBmy4oHOfOWr1m7boNwQumAAtzaTDmgOGTksXTooBQCoAAoASmk7O29gGgA6UhM4AF4Lf2toigBtCRgGAFpqOmAxLIAiRlEJKFqAXQpY20TZZLSPEW9suB704xhUomBxAuKGUoraGGq6vu9W9s7ZfE47RRU1OABzVBgAEQZ9aJWvADkGEFRzcMmD2N9LKISuuFCROndZrBeZ4AWU8XkGwyuqQA7sBvAAVVAADxglzBt3uHWCXR+fwkhhyIABQJEB1B-S840mYmmJXKlUWNVq+LWWK+GmOCigIjg+I2BHYW3YDIKwnQEGh-AYcAUNFQUlwcDleV5s1QcF0cGlDVQ4kkFAANFqaPoROp4FkAHyfWS4+A6vVSHIiVDQwTCXVNOL8u2qgnfD2O1JHU7nAoAKQYIgUkn07RhcK8AFEkQwcIiUQUAIzrbFa6EMOFDVLcNQAa2i+Kx+A6QA)

Summary:

- no way to pass dependencies from the outer scope, unlike `ClientFunction`
  - so there's no way to compose multiple extended selectors
- custom methods don't support element manipulation inside e.g. calling `t.click()`, just like `ClientFunction`
- TypeScript typedefs are verbose and flaky, type-safety is limited
- note that the property getter or method's first argument (`node`) may be `null` if element doesn't exist, even though `testcafe` TypeScript typedefs say otherwise

### Further reading

- [[docs] Describe TestCafe current limitations · Issue #5196 · DevExpress/testcafe](https://github.com/DevExpress/testcafe/issues/5196)

## 🚧 Debug effectively
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

---
title: React component unit testing
date: 2019-04-07 02:30:43
tags:
- React
- TDD
- testing
---

# Core concepts

## ATDD

Analogous to test-driven development, Acceptance Test Driven Development (ATDD) involves team members with different perspectives (customer, development, testing) collaborating to write acceptance tests in advance of implementing the corresponding functionality.  The collaborative discussions that occur to generate the acceptance test is often referred to as the three amigos, representing the three perspectives of customer (what problem are we trying to solve?), development (how might we solve this problem?), and testing (what about...).

These acceptance tests represent the user's point of view and act as a form of requirements to describe how the system will function, as well as serve as a way of verifying that the system functions as intended. In some cases the team automates the acceptance tests.

## Snapshot testing

Snapshot tests are a very useful tool whenever you want to make sure your UI does not change unexpectedly.

A typical snapshot test case for a mobile app renders a UI component, takes a snapshot, then compares it to a reference snapshot file stored alongside the test. The test will fail if the two snapshots do not match: either the change is unexpected, or the reference snapshot needs to be updated to the new version of the UI component.

# Frameworks and Libraries for testing

## jest

Jest is a library for testing JavaScript code. It's an open source project maintained by Facebook, and it's especially well suited for React code testing, although not limited to that: it can test any JavaScript code. Jest is very fast and easy to use.

## vs. Mocha

### Strengths of Jest

- works out of the box with minimal setup or configuration.
- supports snapshot testing

### Weaknesses of Jest

- being newer and less widely used among JavaScript developers.
- less tooling and library support available.
- more difficult to use Jest across the board for larger projects that utilize different types of testing.

### Strengths of Mocha

- Mocha's greatest strength is its flexibility. It doesn't come with an assertion library or mocking framework.
- a lot of tooling built up around Mocha.
- the Mocha community is large.

### Weaknesses of Mocha

- requires more configuration.
- You can use snapshot testing with Mocha, but it's not as easy to integrate.

## enzyme

Enzyme is a JavaScript Testing utility for React that makes it easier to test your React Components' output. You can also manipulate, traverse, and in some ways simulate runtime given the output.

**Focus on the output, not the implementation**


### Difference between Shallow, Mount and render of Enzyme

https://gist.github.com/fokusferit/e4558d384e4e9cab95d04e5f35d4f913

- Shallow: Real unit test (isolation, no children render)
- Mount: The only way to test componentDidMount and componentDidUpdate. Full rendering including child components.
- Render: only calls render but renders all children.

# Basic setup

## jest.config.js

```js
module.exports = {
  setupFiles: [
    '<rootDir>/setupTests.js'
  ],
  snapshotSerializers: ['enzyme-to-json/serializer'],
  collectCoverageFrom: [
    "**/*.{js,jsx,ts,tsx}",
    "!**/node_modules/**",
  ],
};
```
## setupTest.js

```js
/** Jest test setup file. */

const { configure } = require('enzyme');
const Adapter = require('enzyme-adapter-react-16');
const ReactDOM = require('react-dom');
const { initializeIcons } = require('@uifabric/icons');

// Initialize icons.
initializeIcons('');

// optional
window.requestAnimationFrame = callback => {
  setTimeout(callback, 0);
};

configure({ adapter: new Adapter() });

// https://github.com/facebook/react/issues/11565
ReactDOM.createPortal = node => node;

```

# Best practices

## Create a separate file with the global variables

## Use 'describe' and 'it 'keywords appropriately in tests

- 'describe' helps to decompose/break your test suite into components. It helps to break the tasks of a component and helps to visualize the big picture of the tests. Also, you can use nested 'describe' statements to further subdivide the test suite.
- 'it' explains each test that you are going to perform. For example, in a component named 'Add', can have a test specified as “it('should render 2 \<input>s'……….”. You should not be able to subdivide tests further when you use 'it'.

##  Main categories to test

It is better if you can categorize the tests under ‘describe’ keyword. Following are some examples.

- Rendering — You can categorize the tests like rendering components in a parent component. Examples: How many text boxes rendered?, Whether some element is rendered under some condition? etc.

- Interactions — You can categorize the tests which help to check the interactions in UI. Examples: onClick method of a button, onChange method, state changes etc.

- Lifecycle Method Calls — You can categorize tests which help to know whether a particular lifecycle method is called. Examples: To test componentDidMount method and associated state changes etc.

## Things to be done Before and After tests

```js
beforeEach(() => { someInitializationFunction(); });
afterEach(() => { someClearFunction(); });
beforeAll(() => { someOneTimeInitializationFunction(); });
afterAll(() => { someOneTimeClearFunction(); });
```

##  Passing props

When there are more than one props to be passed it is better to use a separate function to create the props (createTestProps()) and assigned them to a variable (props) by calling the function inside beforeAll() or beforeEach() or in any place you need. Below is an example of such function usage.

```js
function createTestProps(props) {
    return {
        apiId: '6e770272-212b-404e-ab9c-333fdba02f2f',
        cancelButton: true,
        allComments: [],
        theme: { custom: { maxCommentLength: 1300 } },
        ...props,
    };
}
let wrapper, props;
beforeEach(() => {
    props = createTestProps();
    wrapper = shallow(<Comment {...props} /> );
});
```

# Full example

```js
import "jest";
import * as React from "react";
import { shallow } from "enzyme";

import { MyInput } from "../src/index";

const createTestProps = (props = {}) => {
  return {
    initialValue: "",
    onChange: (newValue: string): void => {
      console.log(newValue);
    },
    validate: (value: string): boolean => {
      return true;
    },
    ...props,
  };
};

describe("MyInput", () => {
  it("renders correctly", () => {
    const props = createTestProps();
    const wrapper = shallow(
      <MyInput {...props} />,
    );
    expect(wrapper).toMatchSnapshot();
  });

  it("renders correctly with initial value", () => {
    const props = createTestProps({
      initialValue: "test init",
    });
    const wrapper = shallow(
      <MyInput {...props} />,
    );
    expect(wrapper).toMatchSnapshot();
  });

  it("validation works correctly", () => {
    const props = createTestProps({
      validate: (value: string) => {
        return value.length < 10;
      }
    });
    const wrapper = shallow(
      <MyInput {...props} />,
    );
    expect(wrapper).toMatchSnapshot();
    wrapper.find("input").simulate("onChange", { target: { value: "test" }});
    expect(wrapper).toMatchSnapshot();
    wrapper.find("input").simulate("onChange", { target: { value: "testtesttest" }});
    expect(wrapper).toMatchSnapshot();
  });
});

```
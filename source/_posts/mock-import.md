---
title: Mocking ES6 module import
date: 2018-08-08 09:28:22
tags:
---

```js
// feature.js module
import { fetchData } from './backend';

export function doSomething() {
  // some code which calls fetchData
}
```

```js
// feature.test.js module
import { expect } from 'chai';
import sinon from 'sinon';
import { doSomething } from './feature';
import * as Backend from './backend';

describe.only('doSomething', () => {
  it('does something meaningful', () => {
    sinon.stub(Backend, 'fetchData');
    // use doSomething with mocked fetchData
  });
});
```

## Mocking with Dependency Injection

```js
// feature.js module
import { fetchData as originalFetchData } from './backend';

let fetchData = originalFetchData;

export function mock(mockedFetchData) {
  fetchData = mockedFetchData || originalFetchData;
}

export function doSomething() {
  // some code which calls fetchData
}
```

```js
// feature.test.js module
import { expect } from 'chai';
import { mock, doSomething } from './feature';

const dummyFetchData = () => {};

describe('doSomething', () => {
  beforeEach(() => { mock(dummyFetchData); });
  afterEach(() => { mock(); });

  it('does something meaningful', () => {
    // use doSomething with mocked fetchData
  });
});
```

## Mocking Module

```js
// feature.js module
import * as Backend from './backend';

let { fetchData, saveData, deleteData } = Backend;

export function mock(mockedBackend) {
  ({ fetchData, saveData, deleteData } = mockedBackend || Backend);
}

// some code which calls functions from Backend
```
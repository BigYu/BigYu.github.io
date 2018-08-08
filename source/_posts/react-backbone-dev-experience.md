---
title: Develop experience - Backbone vs. React
date: 2018-03-22 10:05:26
tags:
---

React > Backbone
1. For Backbone, we need to manage the lifecycle of the view hierarchy. Derministic view is basically to solve this issue. So with React, memory leaks because of non-disposal of views will be gone.
Kevin has fixed a but related to location targeting module,  PR
2. For a react component, you can have private function, while in Backbone, it needs more code to make a function real private.
3. In Backbone.Event is a good thing, but it also brings a lot of not-easy-to-solve bugs, we might often meet the bug that we don’t know where the event is triggered.
4. Less file to handle. with the benefit of JSX
5. Backbone: lots of hooks/events between parents-children in hierarchy. React: lifting state up/centralized state management. (E.g. steps-wizard)
6. Backbone: inline style which is hard to be protected by unit tests. React: styles support.
7. Backbone: More code to add a event listener for clicking, hovering, etc. React: jsx makes it much simpler.
8. Backbone: have to clean up/stop listening events when destorying.
9. React: more eslint rules to guarantee the qualities(e.g. a11y)
10. Backbone: Views are possible to break each other by jQuery operations without any warnings.
11. Backbone: difficult to implement fail-fast constructing parameters check. React: propTypes.
12. Backbone: need to draw the real page and do the integration/smoke test. React: unit testing over states can cover a lot.

Backbone > React
1. Backbone proxies to Underscore.js to provide 9 object functions on Backbone.Model. React: have to introduce underscore/lodash.
2. Backone: easy to use jQuery plugins. React: if there is no React version, it is hard to wrap.
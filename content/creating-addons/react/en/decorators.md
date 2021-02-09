---
title: 'Decorators'
tocTitle: 'Decorators'
description: 'Interacting with the stories'
---

Almost there. So far, we created a tool, added it to the toolbar and it even tracks state. We now need to respond to this state and show/hide the outlines.

[Decorators](https://storybook.js.org/docs/react/writing-stories/decorators) wrap stories and add-in extra rendering functionality. We are going to create a decorator that responds to the outline global and handles CSS injection. Which in turn, draw outlines around all HTML elements.

In the previous step we defined the `outlineActive` global, let's wire it up! We can consume globals in a decorator using the `useGlobals` hook.

```js:title=src/withGlobals.js
/* eslint-env browser */
import { useEffect, useGlobals } from '@storybook/addons';

export const withGlobals = (StoryFn, context) => {
  const [{ outlineActive }, updateGlobals] = useGlobals();
  // Is the addon being used in the docs panel
  const isInDocs = context.viewMode === 'docs';

  useEffect(() => {
    // Execute your side effect here
    // For example, to manipulate the contents of the preview
    const selectorId = isInDocs ? `#anchor--${context.id} .docs-story` : `root`;

    displayToolState(selectorId, { outlineActive, isInDocs });
  }, [outlineActive]);

  return StoryFn();
};

function displayToolState(selector, state) {
  const rootElement = document.getElementById(selector);
  let preElement = rootElement.querySelector('pre');

  if (!preElement) {
    preElement = document.createElement('pre');
    preElement.style.setProperty('margin-top', '2rem');
    preElement.style.setProperty('padding', '1rem');
    preElement.style.setProperty('background-color', '#eee');
    preElement.style.setProperty('border-radius', '3px');
    preElement.style.setProperty('max-width', '600px');
    rootElement.appendChild(preElement);
  }

  preElement.innerText = `This snippet is injected by the withGlobals decorator.
It updates as the user interacts with the ⚡ tool in the toolbar above.
${JSON.stringify(state, null, 2)}
`;
}
```

## Injecting the outline CSS

Adding and clearing styles is a side-effect, therefore, we need to wrap that operation in `useEffect`. Which in turn is triggered by the `outlineActive` global. The Kit code comes with an example but, let's update it to handle the outline CSS injection.

```js:title=src/withGlobals.js
/* eslint-env browser */
import { useEffect, useMemo, useGlobals } from '@storybook/addons';

import { clearStyles, addOutlineStyles } from './helpers';
import outlineCSS from './outlineCSS';

export const withGlobals = (StoryFn, context) => {
  const [{ outlineActive }, updateGlobals] = useGlobals();
  // Is the addon being used in the docs panel
  const isInDocs = context.viewMode === 'docs';

  const outlineStyles = useMemo(() => {
    const selector = isInDocs ? `#anchor--${context.id} .docs-story` : '.sb-show-main';

    return outlineCSS(selector);
  }, [context.id]);

  useEffect(() => {
    const selectorId = isInDocs ? `addon-outline-docs-${context.id}` : `addon-outline`;

    if (!isActive) {
      clearStyles(selectorId);
      return;
    }

    addOutlineStyles(selectorId, outlineStyles);

    return () => {
      clearStyles(selectorId);
    };
  }, [isActive, outlineStyles, context.id]);
};
```

Ok, that seems like a big jump. Let’s walk through all the changes.

The addon can be active in both docs and story view modes. The actual DOM node for the preview `iframe` is different in these two modes. In fact, the docs mode renders multiple story previews on one page. Therefore, we need to pick the appropriate selector for the DOM node where the styles will be injected. Also, the CSS needs to be scoped to that particular selector.

<div class="aside"><b>Note:</b> the <code>useMemo</code> and <code>useEffect</code> here come from <a href="https://storybook.js.org/docs/react/addons/addons-api">@storybook/addons</a> and not React. This is because the decorator code is running in the preview part of Storybook. That's where the user's code is loaded which may not contain React. Therefore, to be framework agnostic, Storybook implements a React-like hook library which we can use!</div>

Next, as we inject the styles into the DOM, we need to keep track of them to clear them when the user toggles it off or switches the view mode.

To manage all this CSS logic, we need a few helpers. These use DOM APIs to inject and remove stylesheets.

```js:title=src/helpers.js
/* eslint-env browser */
export const clearStyles = selector => {
  const selectors = Array.isArray(selector) ? selector : [selector];
  selectors.forEach(clearStyle);
};

const clearStyle = selector => {
  const element = document.getElementById(selector);
  if (element && element.parentElement) {
    element.parentElement.removeChild(element);
  }
};

export const addOutlineStyles = (selector, css) => {
  const existingStyle = document.getElementById(selector);
  if (existingStyle) {
    if (existingStyle.innerHTML !== css) {
      existingStyle.innerHTML = css;
    }
  } else {
    const style = document.createElement('style');
    style.setAttribute('id', selector);
    style.innerHTML = css;
    document.head.appendChild(style);
  }
};
```

And the outline CSS itself is based on what [Pesticide uses](https://github.com/mrmrs/pesticide/blob/master/css/pesticide.css).

```js:title=src/outlineCSS.js
/**
 * From pesticide v1.3.0 . @mrmrs . MIT
 */
export default function(selector) {
  return /*css*/ `
    ${selector} body {
      outline: 1px solid #2980b9 !important;
    }
    ${selector} article {
      outline: 1px solid #3498db !important;
    }
    ${selector} nav {
      outline: 1px solid #0088c3 !important;
    }
    ${selector} aside {
      outline: 1px solid #33a0ce !important;
    }
    ${selector} section {
      outline: 1px solid #66b8da !important;
    }
    ${selector} header {
      outline: 1px solid #99cfe7 !important;
    }
    ${selector} footer {
      outline: 1px solid #cce7f3 !important;
    }
    ${selector} h1 {
      outline: 1px solid #162544 !important;
    }
    ${selector} h2 {
      outline: 1px solid #314e6e !important;
    }
    ${selector} h3 {
      outline: 1px solid #3e5e85 !important;
    }
    ${selector} h4 {
      outline: 1px solid #449baf !important;
    }
    ${selector} h5 {
      outline: 1px solid #c7d1cb !important;
    }
    ${selector} h6 {
      outline: 1px solid #4371d0 !important;
    }
    ${selector} main {
      outline: 1px solid #2f4f90 !important;
    }
    ${selector} address {
      outline: 1px solid #1a2c51 !important;
    }
    ${selector} div {
      outline: 1px solid #036cdb !important;
    }
    ${selector} p {
      outline: 1px solid #ac050b !important;
    }
    ${selector} hr {
      outline: 1px solid #ff063f !important;
    }
    ${selector} pre {
      outline: 1px solid #850440 !important;
    }
    ${selector} blockquote {
      outline: 1px solid #f1b8e7 !important;
    }
    ${selector} ol {
      outline: 1px solid #ff050c !important;
    }
    ${selector} ul {
      outline: 1px solid #d90416 !important;
    }
    ${selector} li {
      outline: 1px solid #d90416 !important;
    }
    ${selector} dl {
      outline: 1px solid #fd3427 !important;
    }
    ${selector} dt {
      outline: 1px solid #ff0043 !important;
    }
    ${selector} dd {
      outline: 1px solid #e80174 !important;
    }
    ${selector} figure {
      outline: 1px solid #ff00bb !important;
    }
    ${selector} figcaption {
      outline: 1px solid #bf0032 !important;
    }
    ${selector} table {
      outline: 1px solid #00cc99 !important;
    }
    ${selector} caption {
      outline: 1px solid #37ffc4 !important;
    }
    ${selector} thead {
      outline: 1px solid #98daca !important;
    }
    ${selector} tbody {
      outline: 1px solid #64a7a0 !important;
    }
    ${selector} tfoot {
      outline: 1px solid #22746b !important;
    }
    ${selector} tr {
      outline: 1px solid #86c0b2 !important;
    }
    ${selector} th {
      outline: 1px solid #a1e7d6 !important;
    }
    ${selector} td {
      outline: 1px solid #3f5a54 !important;
    }
    ${selector} col {
      outline: 1px solid #6c9a8f !important;
    }
    ${selector} colgroup {
      outline: 1px solid #6c9a9d !important;
    }
    ${selector} button {
      outline: 1px solid #da8301 !important;
    }
    ${selector} datalist {
      outline: 1px solid #c06000 !important;
    }
    ${selector} fieldset {
      outline: 1px solid #d95100 !important;
    }
    ${selector} form {
      outline: 1px solid #d23600 !important;
    }
    ${selector} input {
      outline: 1px solid #fca600 !important;
    }
    ${selector} keygen {
      outline: 1px solid #b31e00 !important;
    }
    ${selector} label {
      outline: 1px solid #ee8900 !important;
    }
    ${selector} legend {
      outline: 1px solid #de6d00 !important;
    }
    ${selector} meter {
      outline: 1px solid #e8630c !important;
    }
    ${selector} optgroup {
      outline: 1px solid #b33600 !important;
    }
    ${selector} option {
      outline: 1px solid #ff8a00 !important;
    }
    ${selector} output {
      outline: 1px solid #ff9619 !important;
    }
    ${selector} progress {
      outline: 1px solid #e57c00 !important;
    }
    ${selector} select {
      outline: 1px solid #e26e0f !important;
    }
    ${selector} textarea {
      outline: 1px solid #cc5400 !important;
    }
    ${selector} details {
      outline: 1px solid #33848f !important;
    }
    ${selector} summary {
      outline: 1px solid #60a1a6 !important;
    }
    ${selector} command {
      outline: 1px solid #438da1 !important;
    }
    ${selector} menu {
      outline: 1px solid #449da6 !important;
    }
    ${selector} del {
      outline: 1px solid #bf0000 !important;
    }
    ${selector} ins {
      outline: 1px solid #400000 !important;
    }
    ${selector} img {
      outline: 1px solid #22746b !important;
    }
    ${selector} iframe {
      outline: 1px solid #64a7a0 !important;
    }
    ${selector} embed {
      outline: 1px solid #98daca !important;
    }
    ${selector} object {
      outline: 1px solid #00cc99 !important;
    }
    ${selector} param {
      outline: 1px solid #37ffc4 !important;
    }
    ${selector} video {
      outline: 1px solid #6ee866 !important;
    }
    ${selector} audio {
      outline: 1px solid #027353 !important;
    }
    ${selector} source {
      outline: 1px solid #012426 !important;
    }
    ${selector} canvas {
      outline: 1px solid #a2f570 !important;
    }
    ${selector} track {
      outline: 1px solid #59a600 !important;
    }
    ${selector} map {
      outline: 1px solid #7be500 !important;
    }
    ${selector} area {
      outline: 1px solid #305900 !important;
    }
    ${selector} a {
      outline: 1px solid #ff62ab !important;
    }
    ${selector} em {
      outline: 1px solid #800b41 !important;
    }
    ${selector} strong {
      outline: 1px solid #ff1583 !important;
    }
    ${selector} i {
      outline: 1px solid #803156 !important;
    }
    ${selector} b {
      outline: 1px solid #cc1169 !important;
    }
    ${selector} u {
      outline: 1px solid #ff0430 !important;
    }
    ${selector} s {
      outline: 1px solid #f805e3 !important;
    }
    ${selector} small {
      outline: 1px solid #d107b2 !important;
    }
    ${selector} abbr {
      outline: 1px solid #4a0263 !important;
    }
    ${selector} q {
      outline: 1px solid #240018 !important;
    }
    ${selector} cite {
      outline: 1px solid #64003c !important;
    }
    ${selector} dfn {
      outline: 1px solid #b4005a !important;
    }
    ${selector} sub {
      outline: 1px solid #dba0c8 !important;
    }
    ${selector} sup {
      outline: 1px solid #cc0256 !important;
    }
    ${selector} time {
      outline: 1px solid #d6606d !important;
    }
    ${selector} code {
      outline: 1px solid #e04251 !important;
    }
    ${selector} kbd {
      outline: 1px solid #5e001f !important;
    }
    ${selector} samp {
      outline: 1px solid #9c0033 !important;
    }
    ${selector} var {
      outline: 1px solid #d90047 !important;
    }
    ${selector} mark {
      outline: 1px solid #ff0053 !important;
    }
    ${selector} bdi {
      outline: 1px solid #bf3668 !important;
    }
    ${selector} bdo {
      outline: 1px solid #6f1400 !important;
    }
    ${selector} ruby {
      outline: 1px solid #ff7b93 !important;
    }
    ${selector} rt {
      outline: 1px solid #ff2f54 !important;
    }
    ${selector} rp {
      outline: 1px solid #803e49 !important;
    }
    ${selector} span {
      outline: 1px solid #cc2643 !important;
    }
    ${selector} br {
      outline: 1px solid #db687d !important;
    }
    ${selector} wbr {
      outline: 1px solid #db175b !important;
    }`;
}
```

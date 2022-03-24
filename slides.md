---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://source.unsplash.com/collection/94734566/1920x1080
# apply any windi css classes to the current slide
class: "text-center"
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
# persist drawings in exports and build
drawings:
  persist: false
---

# FC Innovation Week 2

# Feature Flags

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---

# What's The Project About?

<br/>
Most of the components we build in FC use feature flags in one form or another. But there is no unified way of doing this.

Tooling doesn't support our different ways of feature flagging e.g.:

- ❌ Hard to run tests with FF enabled.
- ❌ Examples website doesn't let you switch FF easily, etc...
- ❌ On product side every FF requires a code change, which reduces benefits of dogfooding our changes on product fabric.

---

# Ideas to Reduce Friction of Using Feature Flags:

<ul>
<li>Build a common helper that can integrate with LaunchDarkly or custom FF providers and use it across our components to provide access to feature flags.</li>
<li>Provide FF context to analytics events.</li>
<li>Allow toggling FF in specific contexts: e.g. only in comments, etc...</li>
<li>Support running tests with FF locally and in CI.</li>
<li>yarn changeset support for feature flags – prompt to add feature flag to a change set for major/minor changes. Use this data in testing pipeline to automatically run CI with and without a feature flag.</li>
<li>Build a visual panel for FF toggling in the examples website.</li>
<li>PR bot to ask to run with FF.</li>
<li>FF static analysis to create tickets.</li>
</ul>


---

# What We Focused On:

<ul>
<li>Build a common helper that can integrate with LaunchDarkly or custom FF providers and use it across our components to provide access to feature flags.</li>
<li style="opacity: 0.3">Provide FF context to analytics events.</li>
<li style="opacity: 0.3">Allow toggling FF in specific contexts: e.g. only in comments, etc...</li>
<li>Support running tests with FF locally and in CI.</li>
<li style="opacity: 0.3">yarn changeset support for feature flags – prompt to add feature flag to a change set for major/minor changes. Use this data in testing pipeline to automatically run CI with and without a feature flag.</li>
<li>Build a visual panel for FF toggling in the examples website.</li>
<li style="opacity: 0.3">PR bot to ask to run with FF.</li>
<li style="opacity: 0.3">FF static analysis to create tickets.</li>
</ul>

---
layout: center
class: text-center
---

# Common Feature Flags Helper

A common helper that can integrate with LaunchDarkly or custom FF providers and use it across our components to provide access to feature flags.

---

<img src="/assets/structure.png" width="2797" height="2806" style="height: auto;" />

---

# Integrating In A Component

One of the main requirements was that featrure flags utils must be well typed. That's why we need to define a configuration for available for a component feature flags.

```ts
export const featureFlagConfig: FeatureFlagConfig = {
  **name**: {
    type: 'string',
    default: 'default',
  },
  toggle: {
    type: 'boolean',
    default: false,
  },
} as const;
```

---

# Integrating In A Component

Next step is to configure a feature flag consumer using previously created feature flag config.

```tsx {all|2|5-7|all}
function FeatureFlagsExample() {
  const ffConsumer = useFeatureFlagConsumer(featureFlagConfig);

  return (
    <FeatureFlagConsumerContextProvider value={ffConsumer}>
      <FeatureFlagsChild />
    </FeatureFlagConsumerContextProvider>
  );
}
```

---

# Integrating In A Component

Now all children of the `FeatureFlagsExample` component can consume feature flags using a React Hook.

```tsx {all|2-3|14-20|all}
export default function FeatureFlagsChild() {
  const name = useTypedFeatureFlag('name');
  const checked = useTypedFeatureFlag('toggle');
  return (
    <>
      <Wrapper>Hello world 👋 {name}</Wrapper>
      <div>
        <input type="checkbox" checked={checked} />
      </div>
    </>
  );
}

/**
 * Feature Flag Config
 * {
 *  name: { type: 'string', default: 'default' },
 *  toggle: { type: 'boolean', default: false },
 * }
 */
```
---

# Feature Flags Toggle Panel

Having feature flags setup using common configuration lets us build some cool stuff on top of it.

<video src="/assets/feature-flag-panel.mp4" autoplay loop />

---
layout: center
class: text-center
---

# Testing With Feature Flags

Support writing and running tests with feature flags locally.

---

# Testing With Feature Flags

Example component that is using feature flags:

```tsx {all|5|6}
import React from 'react';
import { useFeatureFlag } from '@atlaskit/feature-flags/hooks';

export function Dull() {
  const isDull = useFeatureFlag('editor.feature.dull');
  return <div>{isDull ? 'Dull!' : 'Not Dull!'}</div>;
}
```

---

# Testing With Feature Flags

This is how we would test it with and without feature flags?

```tsx {all|1|4|5|9-14|16|all}
withFeatureFlag('editor.feature.dull')
  .describe('Dull Feature', () => {
    it('should be dull', () => {
      const hello = mount(<Dull />);
      expect(hello.html()).toBe('<div>Dull!</div>');
    });
  });

describe('Dull Feature w/o ff', () => {
  it('should not be dull', () => {
    const hello = mount(<Dull />);
    expect(hello.html()).toBe('<div>Not Dull!</div>');
  });
});

withFeatureFlag({ key: 'editor.feature.dull', name: 'Dull Feature'}, false,)
  .describe('Dull Feature', () => {
    it('should not be dull', () => {
      const hello = mount(<Dull />);
      expect(hello.html()).toBe('<div>Not Dull!</div>');
    });
  });

```

---

# Testing With Feature Flags

Now we can run the test:

<video src="/assets/testing2.mp4" autoplay />

---

# Run All Tests With A Feature Flag

Now we can pass `--with-ff="<feature-flag>"` to `yarn test` to run all tests with a feature flag:

<video src="/assets/testing3.mp4" autoplay />

---

# Important!

* It's all just a very time constrained spike, which we definitely should explore more
* There is a branch: [atlassian-frontend/branch/ff-iw](https://bitbucket.org/atlassian/atlassian-frontend/branch/ff-iw) (beware of the IW code quality and hacks ⚠️)
* There is a lot of work to even scratch the surface of some of the ideas that we had pre innovation week.

---
layout: center
class: text-center
---

# Thank you! ⭐️


---
title: 'Creating Addons'
description: 'Learn how to build your own addons that will super charge your development'
heroDescription: 'Addons are one of the top reasons for Storybook’s popularity. With the addon API, you can add new features, automate workflows, and integrate with their favourite libraries. What’s more, you can share those addons with the community as reusable packages.'
overview: 'While the Storybook community offers over 200 addons, you can also build one tailored to your specific needs. Getting started is straightforward. This guide will introduce you to the Addon Kit and walk through the associated APIs and help you build your first Storybook addon.'
order: 4
themeColor: '#FF4785'
codeGithubUrl: 'https://github.com/chromaui/learnstorybook-code'
heroAnimationName: null
toc:
  [
    'introduction',
    'getting-started',
    'register-an-addon',
    'track-state',
    'decorators',
    'preset',
    'add-to-catalog',
  ]
coverImagePath: '/guide-cover/creating-addons.svg'
thumbImagePath: '/guide-thumb/creating-addons.svg'
contributorCount: '+3'
authors:
  [
    {
      src: 'https://avatars2.githubusercontent.com/u/42671',
      name: 'Varun Vachhar',
      detail: 'Storybook DX',
    },
  ]
contributors: []
---

<h2>What you'll build</h2>

<div class="badge-box">
  <div class="badge">
    <img src="/frameworks/logo-react.svg"> <a href="/creating-addons/react/en/introduction/"> React</a>
  </div>
</div>

![Outline Addon](/images/outline-addon-hero.gif)

Follow along as we code the Outline addon—used for visually debugging CSS layout and alignment. It adds a toolbar button that outlines all UI elements, making it easy to verify positioning and placement at a glance.

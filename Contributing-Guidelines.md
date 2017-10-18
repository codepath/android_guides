## Overview

This guide is about contributing to the Android guides! Hopefully you are finding the Android guides helpful in your work with Android and are here because you are interested in contributing back. The best way to contribute is to either improve  / update existing guides or to create your own for missing topics.

### Missing Topics

Check the [issues](https://github.com/codepath/android_guides/issues) for this repository to see what types of contributions would be most impactful. In particular, I will [maintain an issue](https://github.com/codepath/android_guides/issues/2) that contains the most important missing topics. Also look for items in the cliffnotes with the **Needs Attention** mark which indicates the guide needs some love.

## Getting Started

### Cloning Guides Locally with Git

If you want to contribute to these guides with local offline development, start by cloning the guides repo locally on your computer:

```bash
git clone git@github.com:codepath/android_guides.git
cd android_guides
git submodule init
git submodule update
cd guides
git checkout master
```

You can see all the docs as markdown files in the `guides` directory. Update your local copy periodically with the latest guides using:

```
git pull origin master
git submodule update
```

After changes have been made, you can post the updated topic drafts online as outlined below.

### Creating Topic Drafts Online

For new guide, if you prefer to write your guide drafts online and keep them separate from this wiki, you can use the [hackmd.io](https://hackmd.io) for collaborative markdown drafts.

<a href="https://hackmd.io"><img src="https://i.imgur.com/Pg8BycW.png" width="400" /><a/>

Using collaborative online markdown tools, you can create new guide content. When the guide is ready to be reviewed, you can [submit an issue to this repo](https://github.com/codepath/android_guides/issues) with a link to the guide. Once the guide is ready, we can merge the guide back into the master wiki. 

### Modifying Guides on Wiki Directly

If you want to change the guides more easily, you can [edit the wiki](https://github.com/codepath/android_guides/wiki) directly. The choice is up to your discretion but we recommend for small changes such as correcting typos that you update directly through the wiki.

## Guidelines

The cliffnotes philosophy is all about cutting down the content keeping the guides **lean and practical**. In particular:

 * **Don't** include long preambles, blocks of text or extensive vocabulary lessons
 * **Do** focus on the **practical** and be to-the-point
 * **Do** include screencaps and images (and even video) that supplements the content 
 * **Do** discuss third-party open-source libraries that save you tons of time
 * **Do** put the code blocks first. **Always include practical code samples**
 * **Don't** use elaborate examples that take way too much time to understand.
 * **Do** mention the implications and practical conclusions you've made.

## Principles

The most important thing is to **save people as much time as possible**. Include the maximum amount of useful code that we can in the smallest amount of space and 90% should be code in most cases. A few key principles to keep in mind:

 * Have short introductions that explain the reason why the concept matters
 * Always share references at the bottom of the guide that cites all sources
 * Feel free to use gists for large bodies of code you want to link to

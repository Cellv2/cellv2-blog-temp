---
title: "Upgrade npm packages to latest"
date: "2020-10-04T18:56:15.046Z"
description: "Forcefully update all of your npm packages, regardless of semver ranges"
---

This will be short and sweet, but still a TL;DR:<br/>
`npx npm-check-updates -u`<br/>
`npm i`

Firstly, ensure that you have a backup, whether that be through a VCS (such as git) or a copy on disk, because this could make things go really south.

Secondly, ensure you have [npx](https://www.npmjs.com/package/npx) installed - this should come with npm 5.2+, and you need NodeJS 5.6+ to run this.

Thirdly, go to the command line and put the following:<br/>
`npx npm-check-updates -u`

Finally, run the below to install the latest packages.<br/>
`npm i`

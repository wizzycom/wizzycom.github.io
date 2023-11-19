---
title: Prestashop 1.7 - Classic theme SCSS to CSS
date: 2017-06-14 19:22:26.000000000 +03:00
type: post
categories:
- How to
tags:
- Prestashop
permalink: "/prestashop-1-7-classic-theme-scss-to-css/"
---
For the past 3 days, I was looking for a way to make changes to the classic's theme CSS. As an amateur, I thought that if I make the changes I want to the files that are located on the **\_dev** directory inside my theme, changes will be converted to CSS....... **WRONG !!!**

So I started looking harder. There are many solutions on the internet. Some almost worked, some not, some were to complicated to understand. After spending hours and hours o this, I final made it. So here it goes.

Classic theme CSS is based on a module bundler called **webpack.js**. At this time I am trying to understand how it works...

So to be able to make changes on the .scss files and then merge the changes on theme.css, you must bundle all the scss files with the help of webpack.js. Complicated. More complicated if you are using windows. I know, trust me. Webpack.js can be operated via node.js, a JavaScript runtime. So this must be installed first.

First we need to install node.js. You can download it from [here](https://nodejs.org/en/). Use version 6.11.0 ( when I started this tutorial was the latest LTS version)

Go to Prestashop Github and clone branch 1.7.1.x. We need some files contained in \_dev directory that are not included on the normal download.

After you have decompress the archive, open  **Node.js** command prompt (a dedicated cmd is installed with node.js called "Node.js command prompt") and move to **themes\\classic\\\_dev** and run the following command :

```
npm install
```

This will install all the dependencies needed. After that, make your changes on the scss files and when you are done, run :

```
npm run build
```

The new files generated, can be found on **themes\\classic\\assets**.

If you receive messages of Bourbon deprecated stuff, it means that a newest version of burbon is installed via the **npm**. With the following command you will install an older to overcome this problem :

```
npm install bourbon@4.2.6
```

Also some info on Linux can be found [here](https://www.prestashop.com/forums/topic/538034-help-using-webpack/?p=2376769).

Good luck

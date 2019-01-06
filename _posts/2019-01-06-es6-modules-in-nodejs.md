---
layout: post
title: "Using ES6 Modules in NodeJS"
author: "Kwabena Boadu"
categories: journal
tags: [javascript,node,guidebook,lint,babel,nodemon,es6,modules]
---

While working on [Node Guidebook](https://github.com/biblicalph/guidebook){:target="_blank"}, a project I started to share my thoughts on building NodeJS applications, I needed to answers a couple of questions.

1. Which do I use? `commonjs` (aka `module.exports` and `require`) or the new ES6 modules? 
Note: As at NodeJS v11, ES6 modules are not  supported in NodeJS.
2. How do I ensure a consistent code format and style? 

## Commonjs or ES6 Modules?
I decided to go with ES6 modules for Guidebook as it is the future of module management in Javascript. 
I also decided to not use `--experimental-modules` NodeJS flag - I personally don't like the `.mjs` extensions required to use it.
As a result, I chose to write ES6 modules code and compile down to ES5 `commonjs` code using [babel](https://babeljs.io/). 
This seems rather counterintuitive - why not use `commonjs` modules directly and skip the compile step? Well, I reasoned that sometime in the near future, ES6 modules will be fully supported in NodeJS. When that day arrives, I can safely remove `babel` and not have to touch any of my code. 

### Setting Up Babel
1. Create a new directory for the ES6 modules app
```
mkdir es6-modules-app
cd es6-modules-app
```
2. Create a `package.json` file in the project from step 1
```
npm init -y
```
Note: You can run `npm init` and follow the on-screen prompts to setup a `package.json` file
3. Install babel and nodemon development tools
```
npm install --save-dev babel-cli babel-preset-env nodemon
```
Note: `nodemon` will be used to start a development server that reloads anytime we make code changes
3. Create `.babelrc` file - the babel configuration file, with the following contents
```
{
  "presets": [
    "env"
  ]
}
```
4. Add `build` and `start` commands to `package.json`. Your package.json file should look something like this
```
{
  "name": "es6-modules-app",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "babel src -d dist",
    "start": "node dist/index.js",
    "start:dev": "nodemon src/index.js --exec babel-node"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    ...
  }
}
```
The build command will compile code from our source directory, `src` and output it to a destination directory `dist`. Note: you are free to decide the directory names.
5. Create a `src` directory
`mkdir src`
6. Create the following files with their contents
`math.js`
```js
export const sum = a => b => a + b
export const prod = a => b => a * b
export const divide = a => b => {
  if (b > 0) return a / b;
  throw new Error('Cannot divide by 0');
}
```
`index.js`
```
import { sum, prod, divide } from './math.js'

console.log('sum of %d and %d = %d', 8, 10, sum(8)(10));
console.log('product of %d and %d = %d', 6, 5, prod(6)(5));
console.log('%d divided by %d = %d', 22, 5, divide(22)(5));
```

To start the development server, run `npm run start:dev`. While the server is running, editing either `math.js` or `index.js` reloads the server.
To deploy your chances to production, run `npm run build` - this creates a `dist` directory which can be deployed to the production server.

Hope you enjoyed this article. See you in the follow up article. In the meantime, feel free to leave comments and suggestions. 
# VS Code, NPM, AirBnB, ESLint, Prettier and Babel for Node development

Tools in the Javascript ecosystem get a bad rep. There are a lot of them and you often hear about tool fatigue. Over the past year the community has become far more opinionated about the set of tools you should be using. This has been in part helped by boiler plate and starter project scripts such as Dan Abramov’s create-react-app or Yeoman.

Setting up my development environment makes me feel dumb. I question where the lines of Prettier starts and where ESLint ends. So here are my thoughts and the patterns I use when starting new NodeJS projects or hacks for quick startup and seamless integration to my code editor.

## Toolset

**VS Code:** My current favourite code editor at the moment. Built on the same framework as Atom but feels more responsive — closer to Sublime or other native applications. Despite being quite new to family of editors, the JavaScript community has picked it up and already has a healthy selection of extensions.
**NPM:** de facto package manager for NodeJS, is also a great task runner. No need for Grunt or Gulp here.
**AirBnB JS Styleguide:** Opinionated style guides such as  Jordan Harband’s fab set of rules from AirBnB has made code reviews less taxing, as well as code which is simpler to read and maintain. Jordan’s outreach to the community also creates a healthy discussion and frequent updates to the prefered syntax as the language mutures.
**ESLint:** A versitile and muture code linter. Not the easiest to get started with due to its flexibility but with its wide adoption, documentation and support has improved.
**Prettier:** Code formatter especially handy when copying from external sources. Have you just adopted some code where the original engineer doesn’t beleive in semi colons or uses tabs instead of spaces? Don’t worry, Prettier has you covered. I try not to lean on formatters whilst writing code, it can make you lazy. If you can write perfect code, or at least respond when ESLint tells you to fix something, it will help you learn the language better.
**Babel:** When writing open source code for the community, being able to support a wide range of NodeJS versions is helpful. You might be running Node 9.0.0, utilising async-await or some of the ES2017 Object methods but think of the poor souls using V4 (or shudder to think 0.12). Babel will transpile your bleeding edge code so older versions can run your package.

## VS Code Setup
### Extensions
**Babel:** Download
Despite being called Babel, there is no transpiling here, this extension is all about code highlighting. When using your favourite colour theme, it may not work as expected with all your fantastic ES2017 syntax, add Babel and your code will look great again.
**ESLint:** Download
This extension will notify you about all those times you use a let instead of a const and all the other errors that may conflict with AirBnB’s style guide.   At the bottom right of your screen will be a number of errors and warnings for your project that you need to fix.
**Prettier:** Download
Where ESLint shouts at you for making syntax mistakes, Prettier will fix them for you right in your editor. As mentioned above I use this sparingly, usually when working with other peoples code where it would take ages to reformat to your standards. I worry the overuse of formatters can have detrimental effects on learning how to write clean code.

### Settings
Once the above extensions have been installed, your editor requires some workspace preferences to be set in order for them to work effectively.
```json
{
  // prevents other formatters running on save
  "editor.formatOnSave": false,
  // run prettier on save when editing a javascript file
  "[javascript]": {
    "editor.formatOnSave": true
  },
  // prevents default VS Code JS linting
  "javascript.format.enable": false,
  
  // tells prettier to run eslint --fix after runnning
  "prettier.eslintIntegration": true
}
```

## Node Project Setup
At the heart of your NodeJS project will be NPM. NPM is known for being a great package manager but it is also pretty good at running simple tasks. To get started, lets install the developer packages that we need.
### Installing Packages

```bash
(export PKG=eslint-config-airbnb; npm info "$PKG@latest" peerDependencies --json | command sed 's/[\{\},]//g ; s/: /@/g' | xargs npm install --save-dev "$PKG@latest" eslint prettier eslint-plugin-prettier eslint-config-prettier nodemon babel-cli babel-preset-env)
```
The above installs
- **eslint** Linting Tool
- **prettier** Formatting Tool
- **eslint-config-airbnb** Extends the ESLint rules to include AirBnb Styleguide
- **eslint-config-prettier** Makes sure that Prettier and ESLint doesn’t conflict
- **eslint-plugin-prettier** After Prettier has run, ESLint will continue Linting
- **nodemon** Watches for changes in your project.
- **babel-cli** Transpiles your code
- **babel-preset-env** Allows Babel to target a very of Node

### Configuring ESLint
ESLint requires a configutation file at the root of your project called `.eslintrc.` In this file you are able to add custom rules as I have below but these are entirely personal to you and your project.
```json
{
  "extends": ["airbnb", "prettier"],
  "rules": {
    "no-plusplus": 0
  }
}
```

The above extends the internal lint rules with airbnb’s style guide which will also play nice with prettier. I have added `“no-plusplus”: 0` so that I can use `i++;` to increment my numbers rather than `i += 1`.

### Configuring Babel
As mentioned above Babel transpiles your modern code down to something that will execute on older version of node using polyfils. You can specify which version of NodeJS you want to support by creating a `.babelrc` file at the root of your project. If you wanted to support V4 LTS, it may look something like this
```json
{
  "presets": [
    ["env", {
      "targets": {
        "node": "4.8.5",
      }
    }
  }
}
```
### NPM Scripts
In the fly-by-night world of Javascript tooling, you don’t see too many new projects with Grunt and Gulp. On the front end, though not strictly task runners, Webpack and Rollup have replaced the need for them. When developing for the server, NPM and Nodemon meet the majority of the requirments.
```json
"scripts": {
  "watch": "nodemon --exec npm run lts",
  "build": "npm run lint && npm run test && npm run transpile",
  "lts": "npm run lint && npm run test && npm run server",
  "lint": "./node_modules/eslint/bin/eslint.js src/**",
  "test": "echo \"add a test framework\"",
  "server": "node ./src/index.js",
  "transpile": "./node_modules/babel-cli/bin/babel.js src --out-dir lib --copy-files"
},
```

The main two scripts that you would use here are watch and build. Lets break down what is happening. When you start working on your project, by executing `npm run watch` from either the terminal or the built in VS Code terminal, your `src` directory will be monitored for changes. On change, ESLint will run to give you additional errors and warnings, then any tests will run and finally the index file in your src will be executed.


`npm run build` sends your source file to be transpiled to the `lib` directory ready to be distributed.
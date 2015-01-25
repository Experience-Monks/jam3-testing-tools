# jam3-testing-tools

This is a brief introduction to some common practices when it comes to testing frontend modules, and *test-driven development.* The goal is to test *while* you develop the module, so that you end up with a test for each feature you've added. 

## contents

- [global tools](#global-tools)
- [tape](#tape)
- [nodemon](#nodemon)
- [beefy](#beefy)
- [smokestack](#smokestack)
- [pretty-printing](#pretty-printing)
- [alternatives](#alternatives)

## global tools

The first step is to install some global tools that we will use for testing and development. Usually it's recommended to install these tools locally (per-module), but we will get to that a bit later. For now, global tools help us get up and running a bit quicker.

```sh
# installs some global tools
npm i beefy nodemon browserify faucet -g
```

<sup>*TIP* - `npm i` is shorthand for `npm install`</sup>

## tape

Many modules are generic and not tied specifically to NodeJS or the browser. For example, a `rgb-to-hsl` color conversion module does not need to rely on the DOM or on any OS-level features. For these modules we can use [tape](https://www.npmjs.com/package/tape), which works in Node and the browser.

First, stub out a `test.js` file. If you plan to have many tests and assets, you might want to put this in a `tests` folder. Some tools like [module-generator](https://www.npmjs.com/package/module-generator) might already stub this out for you.

```js
var rgb2hsl = require('./')
var test = require('tape')

test('converts rgb colors to hsl', function(t) {
    //... unit tests here
    t.end()
})
```

Then, you can update the `scripts` field of your `package.json`. This might already be done if you used a module generator. 

```json
  "scripts": {
    "test": "node test.js"
  }
```

You can now test the module with `npm test` (alias for `npm run test`), or just running `node test.js`.

## nodemon

When working with a generic module like `rgb-to-hsl`, it's usually faster to develop it in a terminal than to set up HTML and a browser. During development, we can use [nodemon](https://www.npmjs.com/package/nodemon) to live-reload the test file. The process looks like this:

```sh
# start listening to our test.js file
nodemon test.js
```

Now when you save the `test.js` file, it will auto-reload the terminal script and print the results. 

## adding unit tests

Now you can start developing your core module, and testing while you do so. For example, your tests might look like this after a few iterations. We use `equal` to compare two values with strict equality, and then `deepEqual` to compare arrays and objects. Make sure to end tests with `t.end()`. 

You can read more about tape, async tests, and other features [here](http://substack.net/how_I_write_tests_for_node_and_the_browser). 

```js
var rgb2hsl = require('./')
var test = require('tape')

test('converts rgb colors to hsl', function(t) {
    var color = [125, 125, 125]
    t.equal(rgb2hsl(color).length, 3, 'should return [H, S, V] array')
    t.equal(rgb2hsl(color)[1], 0, 'should be zero saturation')
    t.deepEqual(rgb2hsl([255, 0, 0]), [0, 100, 50], 'should be pure red')
    //.. some other tests..
    t.end()
})
```

## beefy

The above `nodemon` tool won't work with browser code. If we're making a browser-specific module (like [dom-css](https://github.com/mattdesl/dom-css)) we can do some initial development with [beefy](https://www.npmjs.com/package/beefy) using the following command:

```sh
#run beefy and open the browser
beefy test --open
```

<sup>*TIP* - `beefy foo` is shorthand for `beefy foo.js`</sup>

This will serve the file at `localhost:9966`. Now you can save the `test.js` file and reload the browser to see the results in the console. Beefy is useful for rapid prototyping, and modules that are harder to automate (like WebGL code, user interactions, etc). It's also a good way to deliver [demo / example code](https://github.com/Jam3/delaunify/blob/e83a2118df8e1fc7f5dc03311d288fc8dc3b8cb6/package.json#L27) to the user.

It's typically a good idea to save these tools locally, so that anybody cloning your repo is running the same versions you are. 

```sh
npm install beefy browserify --save-dev
```

Users cloning the repo won't be able to run `beefy test.js` from terminal. Instead, locally-installed tools are run via `npm-scripts` (it first looks inside `node_modules`). So now your package.json looks like this:

```json
  "scripts": {
    "test": "beefy test.js --open"
  }
```

Now running `npm test` will open beefy. 

## smokestack

Although beefy is pretty good, it would be better if we could automate our tests and eventually have them running in the cloud (like with [zuul](https://github.com/defunctzombie/zuul) and SauceLabs). It can also be tedious to have to open DevTools just to see the test results. 

One alternative to beefy is to use [smokestack](https://www.npmjs.com/package/smokestack) for automation. This launches Chrome or FireFox, so it allows us to test the full range of Web APIs (WebGL, WebAudio, etc).

We will install it locally, as well as a couple of other tools:

```sh
npm install smokestack tap-closer browserify --save-dev
```

Then our tests are run like so:

```json
  "scripts": {
    "test": "browserify test.js | tap-closer | smokestack"
  }
```

Now when we run `npm test` it will run the tests in a browser and print the result to the console. These solutions allow us to test the module in the cloud on a variety of browsers and operating systems, like [dom-css](https://github.com/mattdesl/dom-css) which uses SauceLabs for testing and Travis for continuous integration. 

You may also be interested in [testling](https://www.npmjs.com/package/testling) which has similar goals, except focuses more on PhantomJS (i.e. runs heedlessly but doesn't support full range of browser capabilities). 

## pretty-printing

![img](http://i.imgur.com/bROGMVq.png)

We can use [faucet](https://www.npmjs.com/package/faucet) or [tap-spec](https://www.npmjs.com/package/tap-spec) to pretty-print the test results, making them coloured and more compact. Example: 

```sh
nodemon test | faucet 
```

<sup>needs a globally-installed faucet</sup>

If you use it in your package.json, you should install it locally.

```js 
npm install faucet --save-dev
```

For example, if we are using smokestack:

```json
  "scripts": {
    "test": "browserify test.js | tap-closer | smokestack | faucet"
  }
```

## alternatives

- [wzrd](https://www.npmjs.com/package/wzrd) or [prova](https://www.npmjs.com/package/prova) can be used instead of `beefy`
- [testling](https://www.npmjs.com/package/testling) can be used instead of `smokestack`
- [a bunch of alternatives](https://github.com/substack/tape#pretty-reporters) to `faucet`
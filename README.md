# React And Webpack from scratch to Production

In this I will outlay the steps needed to create your react app from scratch using webpack. I won't explain everything in detail, however I'll try to give you an understanding of concepts seperately and how you may use them. These steps assume that you have Node setup and atleast some bare bone knowledge of React.

### We'll Focus on
[React.Js](#reactjs)<br />
[WebPack](#webpack)<br />
[WebPack & React](#webPack-&-react)<br />
[Dev Server](#dev-server)<br />
[Webpack Loaders](#webpack-loaders)<br />
[Hot Reloading](#hot-reloading)<br />
[Building For Production](#building-for-production)<br />



## ReactJs
Let's start with React and see how it works as a library. Follow the steps below.
- Create a folder with your `project_name` && cd into your project directory
- Run `npm init -y` to add a package.json file
- Run `npm i react react-dom`
    > Note: `package-lock.json` will be created these enable new developers to get the same version as everyone else.

- Create a folder named dist at the root and add an index.hml file then add a script tag pointing to this React file in the node_modules just before the closing `</body>` tag. Should look like this

    > Note: `<script src="../node_modules/react/umd/react.development.js"></script>`

- Just below it add another script tag to create a element. Takes  three args: an HTML *tag name*, *props* and *children*.
    >  `<script>
    console.log(React.createElement('h1', null, 'Hello from react!'))
  </script>`

- After reload this, it prints a Js object in the `clg`, can't be rendered to screen yet. That's what ReactDOM is going to do for us.
- ReactDOM has a render method which takes two arguments: A React Element (which we created earlier) and an HTML element container to wrap our React Element.
- Just like React, we'll also add a script tag for ReactDOM. It's somehow similar
    > `<script src="../node_modules/react-dom/umd/react-dom.development.js"></script>`

- Add a `<div />` tag to the `body` and give it an id="root". This is ReactDOM render's second argument.
- So lets mashup React and ReactDOM to render some something on the screen.
 ```javascript
<script>
    ReactDOM.render(React.createElement('h1', null, 'Hello from react!'), document.getElementById("root"))
</script>
```

- Reload the screen and voila!!

## Webpack
**We'll focus on**

Webpack bundles all our js into static files which can easily be run by the browser while also minifing the code for optimization.

- Create a new folder named src at the root of the project. We intend to bundle all our js in this src folder and inject it into the index.html
- Install webpack as a dev-dependency `npm install webpack --save-dev`
- Lets put webpack to a test, add an `index.js` file to the src folder. Add the following log `console.log("Hello Webpack")`. In your terminal enter the following command `node_modules/.bin/webpack --mode=development src/index.js`. This command is divided into three
    -  **node_modules/.bin/webpack** - Running webpack
    -  **--mode=development** - specify development mode
    -   **src/index.js** - specify what we are bundling

    After running this command we will notice that `dist/main.js` is created at the root directory. `dist/main.js` contains our bundled code, to run it simply require it with the `<script />` tag inside our `dist/index.html`. Refresh and check your console "hello Webpack". An example to bundle multiple files, create another file in `src/` named maybe `index2.js` add any console.log statement, then require it inside `index.js` (add `require(index2.js)` at the top of the file) then rerun the command `node_modules/.bin/webpack --mode=development src/index.js`, refresh and check the console.


## Webpack & React
Now that we have seen React & Webpack work seperately, let's mash them up.
- Empty `src/index.js` and copy the React code from the script tag in `dist/index.html` to `src/index.js`. Also remove all script tags `dist/index.html` except one with path `dist/main.js`. You may also delete index2.js if you created it.
- At the top of `src/index.js` require *React* and *ReactDOM* i.e. 
```javascript
const React = require('react'); 
const ReactDOM = require('react-dom');
```

**Configurations**
- We won't have to run that webpack command all the time so lets automate.
- First we need an entry and output file. Entry file is that which requires all the other files (also files that require others and so on) and the output file is where our bundled code goes. We already have these files; entry - `src/index.js`, output - `dist/main.js`
- Create our configuration file a `webpack.config.js` at the root of the project and add.
```javascript
module.exports = {
  mode: 'development',
  entry: __dirname + '/src/index.js',
  output: {
    path: __dirname + '/dist',
    filename: 'main.js',
    publicPath: '/',
  }
}
```
  > we have extracted variables from our inline command (`node_modules/.bin/webpack --mode=development  src/index.js`) and added them to our configuration.
- To run our command all we need is webpack command `node_modules/.bin/webpack`

## Dev Server
It may get tiring to reload the page manually whenever a change is made. DevServer makes it possible for our application to automatically reload when changes are to our code.
- First we'll install the dev server `npm install webpack-dev-server --save-dev`.
- Add this to our `webpack.config.js` configuration file before the closing brace
```javascript
devServer: {
    contentBase: './dist',
    historyApiFallback: true,
    inline: true
}
```
**contentBase** - specifies the folder to serve <br />
**historyApiFallback** - makes it possible for an SPA to have navigation with multiple pages <br />
**inline** - Automatically refresh the page on file changes 

- Add `"start": "node_modules/.bin/webpack-dev-server --open"` to package.json `script`.
- Run `npm run start` and modify code - notice auto reloading

## Webpack Loaders
 **Babel loader** - We will need to write ES6 as opposed to ES5 (what we've been writing). Unfortunately not all browsers support ES6, to enable this support we'll need a transpiler to convert our written ES6 to ES5 which is understandable by the other browsers. This is where babel comes in. We are going to add babel to our webpack build process so that we can transpile our code before bundling.
- Firstly, run `npm i @babel/core babel-loader @babel/preset-env @babel/preset-react babel-plugin-transform-class-properties -D` to install the babel and other plugins.

    > - **babel-loader**- is the most important among the bunch, it gets the code transpile it then pass it for bundling.<br />
    > - **babel-plugin-transform-class-properties**- enables us to use arrow functions outside render and also limit the reference scope of `this` to the App not global. In other words prevents us from using `bind(this)` every where
- Add a module `object` with a `rules` array to our webpack.config.js. In the rules array is where we add an object to specify the loader configurations.
```javacript
module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: 'babel-loader',
        query: {
          presets: ['@babel/preset-react', '@babel/preset-env'],
          plugins: ['transform-class-properties']
        }
      }
    ]
  },
```
**test**: Specifies the types of files to be transpiled <br />
**exclude**: What should be skipped <br />
**loader**: What our loader is called <br />
**query**: What presets will be used by the loader <br />

- Run `npm run start`, if there are no errors - lets proceed add React code
- In our `src/index.js` change the `React.createElement(...)` to `<h1>...</h1>`. also convert the require statements to import statements.
- From now on you are free to use React.

## Hot Reloading
Imaging reloading our application while preserving the current state and not reloading its entirety. Hot reloading enables you to update specific parts of the UI without reloading everything on the web app.
We'll be using [react-hot-loader](https://github.com/gaearon/react-hot-loader) <br />
**Steps to add hot reloading**
- Install react-hot-loader `npm install react-hot-loader`
- Add `react-hot-loader/babel` to babel plugins in `webpack.config.js` to make sure that our hot reloaded files are compiled by *babel*

    ```javascript
       module.exports = {
            ...
            module: {
                rules: [
                    {
                        ...
                        query: {
                            ...
                            plugins: ['transform-class-properties', 'react-hot-loader/babel']
                         }
                    }
                ]
            }
    ```
- Make the root component *hot exported* - this will send a new version of our app to ReactDOM.render when our files change
    ```javascript
      import React from 'react'
      import { hot } from 'react-hot-loader/root';

      const App = () => {
          return (
              <div>
                  <h1>Hello Webpack</h1>
              </div>
          )
       }

       export default hot(App)
    ```
- Finally to turn on hot reloading, add the `--hot` flag to your *start* script in `package.json`

    ```javascript
       // package.json
       "scripts": {
           ...
           "start": "node_modules/.bin/webpack-dev-server --open --hot"
        },
    ```
Or  add `hot: true` to devServer in you `webpack.config.js`

 ```javascript
       // webpack.config.js
       devServer: {
           ...
           hot: true
        },
```

- Hot reloading should be working now. Try editing the text, you notice that it will update without reloading the page.

But that's not all, because it does not support *React Hooks*. Lets add that support. We'll use `@hot-loader/react-dom` to replace react-dom because it has support for hot loading. Here is how.
- Install the package `npm i @hot-loader/react-dom -D`
- Add it to webpack alias in `webpack.config.js`

    ```javascript
    module.exports = {
        ...
        resolve: {
            alias: {
                'react-dom': '@hot-loader/react-dom',
            },
        }
    }
    ```
After this Hot reloading should be working fine with *React Hooks*. 
    >For more check the [documentation](https://github.com/gaearon/react-hot-loader)


## Building For Production
We've so far been working with development, now let's focus on deployment to production. 
When deploying we need to focus on the bare minimum of files required by the remote server to run our app.
- An index.html file (Minified)
- A CSS file (Minified)
- A Javascript file (Minified)
- All image assets
- An asset manifest

Firstly, we'll use webpack to minify the index.html file and automatically add the appropriate style links and script tags, then add it to a build folder. To do this make a copy of our `webpack.config.js` file, name the new copy `webpack.config.prod.js`, as you might have already guessed this will contain our webpack configurations for production. We are going to make a few tweaks to this file since it contains things that may not be needed for production such as the dev-server and hot-reloading.
- Remove all things concerning the devServer and hot-reloading i.e. the devServer key and it's object value, also the `react-hot-loader/babel` value in the babel plugins array.
- Since we are using `build` this time we'll change the pathname in output to `__dirname + '/dist'` and optionally filename to `bundle.js`
- Set the environment to production to avoid react warnings. require `webpack` and add this to the plugins array at the root our webpack.config.prod.js.
```javascript
  plugins: [
    new webpack.DefinePlugin({
      'process.env': {
        NODE_ENV: JSON.stringify('production')
      }
    }),
 ]
```
- Run `npm i html-webpack-plugin -D` to install `html-webpack-plugin` which will make the html minification for us.
- Require `html-webpack-plugin` in our `webpack.config.prod.js` and add this below to our plugins array at the root of the object.
```javascript
    new HtmlWebpackPlugin({
      inject: true,
      template: __dirname + "/dist/index.html",
      minify: {
        removeComments: true,
        collapseWhitespace: true,
        removeRedundantAttributes: true,
        useShortDoctype: true,
        removeEmptyAttributes: true,
        removeStyleLinkTypeAttributes: true,
        keepClosingSlash: true,
        minifyJS: true,
        minifyCSS: true,
        minifyURLs: true,
      },
    }),
```
`inject` - simply inserts a script tag inside out minified code. as we'll see <br />
`template` - Points to what we are minifying<br />
`minify` - minification specifications<br />
- Let's see how it works. first,  change the build command in the `package.json` to `"webpack --config webpack.config.prod.js"`. The `--config` fag specifies the configuration file to use. Run `npm run build` to see what happens.

    > You'll notice a directory `build/index.html` created, When you open `index.html`. you'll the minified code however the problem is that it has two identical script tags. This is because of HtmlWebpackPlugin's inject key. To remove this problem, head back to your `dist/index.html` and delete the script tag, since as we already noticed one is injected for us automatically. Also copy our HtmlWebpackPlugin to `webpack.config.js` but without the `minify` key then rerun `npm run build`.

If you haven't noticed already your app can't render css. Let's enable this. 
- Run `npm i css-loader style-loader -D` check the documentation for more [here](https://webpack.js.org/loaders/css-loader/)
- Add this to `rules` in both webpack config files under `babel-loader`
```javascript
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader']
      },
```
    > What happens here is that the loader grab our css files and inject there into our javascript

Now to our javscript in bundle.js, you'll notice that it spans a number of lines. We are going to limit it to only one line to make it faster. To achieve this, we are going to `npm i uglifyjs-webpack-plugin -D`, require it and then this to the root of out object
```javascript
optimization: {
    minimizer: [
      new UglifyJsPlugin({
        sourceMap: true,
        uglifyOptions: {
          warnings: false,
          output: {
            comments: false,
          },
        },
      }),
    ],
  },
```
- Run `npm run build`, you'll notice a single line of js in bundle.js

We want to make sure that our assets are copied from `dist` to `build`, to make sure they are accessible
- Let's run `npm i file-loader -D` to install the file-loader
- Add this to our webpack files 
```javascript
     {
        test: /\.(png|jpe?g|gif)$/i,
        loader: 'file-loader',
     }
```
File-loader only deals with files required by Javascript, but how about those require by our `index.html` like the favicon. To achieve this we'll not use webpack but javascript.
- Create a folder `scripts` at the root of the project and a file `copy_scripts.js`
- Run `npm i fs-extra` to install fs-extra, require it in our `copy_scripts.js`
- Add a script to copy all files in `dist` except `index.html` to `build`. `copy_assets.js` should look like this
```javascript
const fs = require('fs-extra');

fs.copySync('dist', 'build', {
  dereference: true,
  filter: file => file !== 'dist/index.html'
})
```
- Change build in `package.json` to `node scripts/copy_assets.js && webpack --config webpack.config.prod.js` and run `npm run build`

**Add an Asset Manifest**
We want to list all static assets that we are generating, this will be useful when we start caching them to save load times.
- Run `npm i webpack-manifest-plugin -D`, require webpack-manifest-plugin and then use it in our plugins
```javascript
    new ManifestPlugin({
      fileName: 'asset-manifest.json',
    })
```
- Run `npm run build` an asset-manifest.json will be created in build. opening `build/index.html` will produce the same outcome as when you run `npm run start`

### Congrats !!!

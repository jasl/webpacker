# Webpack


## Configuration

Webpacker gives you a default set of configuration files for test, development and
production environments in `config/webpack/*.js`. You can configure each individual
environment in their respective files or configure them all in the base
`config/webpack/environment.js` file.

By default, you shouldn't have to make any changes to `config/webpack/*.js`
files since it's all standard production-ready configuration. However,
if you do need to customize or add a new loader, this is where you would go.


## Loaders

You can add additional loaders beyond the base set that webpacker provides by
adding it to your environment. We'll use `json-loader` as an example:

```
yarn add json-loader
```

```js
// config/webpack/environment.js
const { environment } = require('@rails/webpacker')

environment.loaders.add('json', {
  test: /\.json$/,
  use: 'json-loader'
})

module.exports = environment
```

Finally add `.json` to the list of extensions in `config/webpacker.yml`. Now if you `import()` any `.json` files inside your javascript
they will be processed using `json-loader`. Voila!

You can also modify the loaders that webpacker pre-configures for you. We'll update
the `babel` loader as an example:

```js
// config/webpack/environment.js
const { environment } = require('@rails/webpacker')

// Update an option directly
const babelLoader = environment.loaders.get('babel')
babelLoader.options.cacheDirectory = false

module.exports = environment
```


## Plugins

The process for adding or modifying webpack plugins is the same as the process
for loaders above:

```js
// config/webpack/environment.js
const { environment } = require('@rails/webpacker')

// Get a pre-configured plugin
environment.plugins.get('ExtractText') // Is an ExtractTextPlugin instance

// Add an additional plugin of your choosing
environment.plugins.add('Fancy', new MyFancyWebpackPlugin)

module.exports = environment
```


### Add common chunks

The CommonsChunkPlugin is an opt-in feature that creates a separate file (known as a chunk), consisting of common modules shared between multiple entry points. By separating common modules from bundles, the resulting chunked file can be loaded once initially, and stored in the cache for later use. This results in page speed optimizations as the browser can quickly serve the shared code from the cache, rather than being forced to load a larger bundle whenever a new page is visited.

Add the plugins in `config/webpack/environment.js`:

```js
environment.plugins.set('CommonsChunkVendor', new webpack.optimize.CommonsChunkPlugin({
  name: 'vendor',
  minChunks: (module) => {
    // this assumes your vendor imports exist in the node_modules directory
    return module.context && module.context.indexOf('node_modules') !== -1;
  }
}))

environment.plugins.set('CommonsChunkManifest', new webpack.optimize.CommonsChunkPlugin({
  name: 'manifest',
  minChunks: Infinity
}))
```

Now, add these files to your `layouts/application.html.erb`:

```erb
<%# Head %>

<%= javascript_pack_tag 'manifest' %>
<%= javascript_pack_tag 'vendor' %>

<%# If importing any styles from node_modules in your JS app %>

<%= stylesheet_pack_tag 'vendor' %>
```

More detailed guides available here: [Webpack guides](https://webpack.js.org/guides/)

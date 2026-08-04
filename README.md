# ResolvePlugin

A small webpack/enhanced-resolve plugin that redirects module resolution between two directories inside the same repository. It lets a project directory fall back to a shared directory (or vice‑versa) so multiple projects in a monorepo can share implementations without duplicating files.

> Core behavior: when resolving a request under `toDirectory`, if the corresponding path does not exist there but exists under `fromDirectory`, the resolver will return the file from `fromDirectory` instead. See `index.test.js` for exact behavior and examples.

## Installation

Install with npm or pnpm:

```bash
npm install --save resolve-webpack-plugin
# or
pnpm add --save resolve-webpack-plugin
```

(Development/test deps in this repo: enhanced-resolve, webpack@^4, jest)

## Quick example (webpack)

This example shows how to register the plugin in a webpack config using `resolve.plugins`.

```js
// webpack.config.js
const path = require('path');
const ResolvePlugin = require('resolve-webpack-plugin');

module.exports = {
  // ... other webpack config
  resolve: {
    plugins: [
      new ResolvePlugin({
        fromDirectory: path.resolve(__dirname, 'shared'),
        toDirectory: path.resolve(__dirname, 'main')
      })
    ]
  }
};
```

## Example (enhanced-resolve ResolverFactory)

The test in this repo uses `enhanced-resolve` directly — this is the exact pattern used in tests:

```js
const ResolverFactory = require('enhanced-resolve').ResolverFactory;
const ResolvePlugin = require('resolve-webpack-plugin');
const resolver = ResolverFactory.createResolver({
  fileSystem: require('fs'),
  plugins: [new ResolvePlugin({
    fromDirectory: path.resolve('./__mocks__/shared'),
    toDirectory: path.resolve('./__mocks__/main')
  })]
});

resolver.resolve({}, __dirname, './__mocks__/main/baz.js', {}, (err, result) => {
  // if main/baz.js doesn't exist, plugin makes resolver return shared/baz.js
});
```

## Options

- `fromDirectory` (string) — required. The directory that acts as the fallback/shared source.
- `toDirectory` (string) — required. The directory where normal resolution happens; when a file is missing here, the plugin checks `fromDirectory` for the same relative path.

If either option is missing the plugin will throw an Error when constructed (see `index.js`).

## How it works (implementation notes)

- The plugin registers two hooks on the enhanced-resolve `resolver`:
  - `after-described-relative`: when resolving inside `toDirectory`, it temporarily marks the request and rewrites the path to `fromDirectory` to allow relative resolution there.
  - `before-existing-file`: when resolving a path inside `fromDirectory`, it checks the corresponding path inside `toDirectory`; if that file exists, it resolves to it.
- A small `statp` helper promisifies `fs.stat` calls used to detect whether files/directories exist.

See `index.js` and `index.test.js` for exact logic and tests.

## Tests

Run the test suite with:

```bash
npm install
npm test
```

The repository contains `index.test.js` which demonstrates usage and the expected resolution behavior.

## Compatibility

- The repo's devDependencies use `webpack@^4` and `enhanced-resolve@^4.0.0`. The plugin is written against enhanced-resolve's plugin API; it should work with webpack configurations that accept `resolve.plugins`.
- If you need explicit support for newer webpack/enhanced-resolve versions, please open an issue or a PR describing the target version and I'll help update compatibility.

## Contributing

PRs and issues are welcome. If you change behavior, update or add tests under `index.test.js` and adjust `__mocks__` as needed.

## License

MIT

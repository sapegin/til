<!-- 2022-04-19 javascript,npm,esm -->

# Transpiling ESM files inside node_modules

This is a gigantic hack but it seems to work, until ECMAScript modules (ESM) are more widely supported. Many npm modules are already published as ECMAScript modules but not all apps could import them. Iʼm using it on [one of my Gatsby sites](https://tacohuaco.co/).

The idea is to use [esbuild](https://esbuild.github.io/) to compile ESM files to CommonJS directly inside the `node_modules` folder on npm install. Weʼll detect packages that need to be compiled by checking that the `type` filed in the `package.json` has the `module` value.

1. Add dependencies:

```
npm install -D tiny-glob esbuild
```

2. Add a new script, `scripts/esmtocjs.js`

```js
const { dirname } = require('path');
const { readFile, writeFile } = require('fs/promises');
const glob = require('tiny-glob');
const { build } = require('esbuild');

/**
 * Read and parse a JSON file
 */
const readJson = async (filepath) => {
	const file = (await readFile(filepath)).toString();
	try {
		return JSON.parse(file);
	} catch (err) {
		console.error(`Cannot parse JSON file: ${filepath}`);
		return {};
	}
};

async function transpileNodeModules() {
  // Get all package.json file inside node_modules, including in subfolders
	const allPackages = await glob(`node_modules/**/package.json`);

	await allPackages.forEach(async (packageJson) => {
		const json = await readJson(packageJson);

    // Skip unless the type of the package is ESM
		if (!json.name || json.type !== 'module') {
			return;
		}

		console.log(`🦀 Transpiling ${json.name}...`);

		const dir = dirname(packageJson);

    // Get all .js files unless they are in a nested node_modules folder
		const entryPoints = (await glob(`${dir}/**/*.js`)).filter(
			(d) => !d.startsWith(`${dir}/node_modules/`)
		);

		if (entryPoints.length === 0) {
			return;
		}

    // Compile all ESM files to CommonJS
		await build({
			entryPoints,
			outdir: dir,
			allowOverwrite: true,
			bundle: false,
			minify: false,
			sourcemap: false,
			logLevel: 'info',
			platform: 'node',
			format: 'cjs',
			target: 'node12',
		});

    // Overwrite the package.json with the type of CommonJS
		await writeFile(
			packageJson,
			JSON.stringify({ ...json, type: 'commonjs' }, null, 2)
		);
	});
}

transpileNodeModules();
```

3. Add a `postinstall` script to the `package.json`:


```json
{
  "scripts": {
    "postinstall": "node scripts/esmtocjs.js"
  }
}
```

Now, every time we run `npm install`, all ESM files inside `node_modules` will be compiled to CommonJS, so they could be imported by Gatsy or any other tool that doesn't yet support ESM.

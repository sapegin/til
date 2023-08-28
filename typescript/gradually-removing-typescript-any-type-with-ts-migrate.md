<!-- 2020-08-20 typescript, codemods, refactoring -->

# Gradually removing TypeScript any type with ts-migrate

TODO

https://github.com/airbnb/ts-migrate

There are two reasons for `any` type in the code:

- _explicit `any`_: the type is explicitly defined as `any`;
- _implicit `any`_: the type is not defined and inferred as `any`.

To disable implicit `any`s we need to enable the `noImplicitAny` [TypeScript compiler option](https://www.typescriptlang.org/docs/handbook/compiler-options.html), which is also included in the `strict` option.

However, on a legacy project that already has many implicit `any`s, it might be tricky to enable `strict` mode. And we want to do it as soon as possible to keep new code from implicit `any`s.

We could disable

1. Install the dependencies:

```
npm install --save-dev ts-migrate-server ts-migrate-plugins
```

2. Enable strict mode in TypeScript config, **tsconfig.json**:

```
{

}
```

3. Write a codemod, **src/any.ts**:

```ts
import path from 'path';
import {
  explicitAnyPlugin,
  tsIgnorePlugin
} from 'ts-migrate-plugins';
import {
  forkTSServer,
  migrate,
  MigrateConfig
} from 'ts-migrate-server';

async function main() {
  // Project root folder
  const rootDir = path.resolve(__dirname, '..');

  // Initialize a TypeScript server
  const server = forkTSServer();
  process.on('exit', () => {
    server.kill();
  });

  // Create new migration config and add plugins with empty options
  const config = new MigrateConfig()
    .addPlugin(explicitAnyPlugin, {})
    .addPlugin(tsIgnorePlugin, {});

  // Tun migration
  const exitCode = await migrate({ rootDir, config, server });

  // Kill server
  server.kill();
  process.exit(exitCode);
}

main();
```

4. Run the codemod:

```
ts-node --compiler-options '{"module": "commonjs"}' src/any.ts
```

**Note:** We need to override compiler options here because our project is using webpack and ECMAScript modules aren’t transpiled, which is required for Node.js.

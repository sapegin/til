# Renaming TypeScript interfaces with ts-morph

[ts-morph](https://ts-morph.com/) is a tool to write _codemods_ for TypeScript — programs that modify TypeScript code.

Here’s how I removed the `I` prefixes of all interfaces in a project: `IDog` becomes `Dog`.

1. Create a file with the codemod, **src/remove-prefixes.ts**:

```ts
import { Project } from 'ts-morph';

const project = new Project({
  tsConfigFilePath: 'tsconfig.json',
});

const sourceFiles = project.getSourceFiles();

sourceFiles.forEach(sourceFile => {
  console.log('👉', sourceFile.getBaseName());

  const interfaces = sourceFile.getInterfaces();
  interfaces.forEach(i => {
    const name = i.getName();
    const nextName = name.replace(/^I([A-Z])/, '$1');
    if (name === nextName) {
      return;
    }

    console.log(name, '->', nextName);
    i.rename(nextName, {
      renameInComments: true,
      renameInStrings: true,
    });
  });

  console.log();
});

project.saveSync();
```

2. Install dependencies:

```
npm install --save-dev ts-morph
```

3. Run the codemod:

```
ts-node --compiler-options '{"module": "commonjs"}' src/remove-prefixes.ts
```

**Note:** We need to override compiler options here because our project is using webpack and ECMAScript modules aren’t transpiled, which is required for Node.js.

## Caveats

This codemod doesn’t do anything with naming conflict. For example, if we already have a `Dog` component or class and we’re importing the `IDog` interface into the same file, we’ll have a naming conflict: both, the component and the interface will be called `Dog`.
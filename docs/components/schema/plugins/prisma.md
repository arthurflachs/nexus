Refer to the [framework Prisma plugin docs](/plugins/prisma#runtime-integration) for API reference and concepts. What follows here are aspects that are particular to the schema plugin only.

## Installation {docsify-ignore}

```cli
npm install nexus-prisma
```

## Usage {docsify-ignore}

1. Import `nexusPrismaPlugin` from `nexus-prisma`
1. Create and configure it if needed (shouldn't be)
1. Pass into `Nexus.makeSchema` `plugins` array

**Example**

```ts
import { nexusPrismaPlugin } from 'nexus-prisma'
import { makeSchema } from '@nexus/schema'
import * as types from './types'

const schema = makeSchema({
  types,
  plugins: [nexusPrismaPlugin()],
})
```

You can find runnable examples in the [repo examples folder](https://github.com/graphql-nexus/nexus-schema-plugin-prisma/tree/master/examples).

## Configuration {docsify-ignore}

Note that, In most cases, you should not need to configure anything.

```ts
type Options = {
  /**
   * nexus-prisma will call this to get a reference to an instance of the Prisma Client.
   * The function is passed the context object. Typically a Prisma Client instance will
   * be available on the context to support your custom resolvers. Therefore the
   * default getter returns `ctx.prisma`.
   */
  prismaClient?: PrismaClientFetcher
  /**
   * Same purpose as for that used in `NexusSchema.makeSchema`. Follows the same rules
   * and permits the same environment variables.
   */
  shouldGenerateArtifacts?: boolean

  inputs?: {
    /**
     * What is the path to the Prisma Client package? By default looks in
     * `node_modules/@prisma/client`. This is needed in order to read your Prisma
     * schema AST and Prisma Client CRUD info from the generated Prisma Client package.
     */
    prismaClient?: string
  }
  outputs?: {
    /**
     * Where should typegen be put on disk? By default emits into `node_modules/@types`.
     */
    typegen?: string
  }
  computedInputs?: GlobalComputedInputs
}
```

## Project Setup {docsify-ignore}

These are tips to help you with a successful project workflow

1. Keep app schema somewhere apart from server so that you can do `ts-node --transpile-only path/to/schema/module` to generate typegen. This will come in handy in certain deployment contexts.

1. Consider using something like the following set of npm scripts. The
   `postinstall` step is helpful for guarding against pruning since the
   generated `@types` packages will be seen as extraneous. We have an idea to
   solve this with [package
   facades](https://github.com/prisma-labs/nexus/issues/253). For yarn users
   though this would still be helpful since yarn rebuilds all packages whenever
   the dependency tree changes in any way
   ([issue](https://github.com/yarnpkg/yarn/issues/4703)). The
   `NODE_ENV=development` is needed to ensure typegen is run even in a context where `NODE_ENV` is set to `production` (like a heroku deploy pipeline, see next point). Also, consider using [cross-env][https://github.com/kentcdodds/cross-env] for better compatibility with different environments, e.g. on Windows.

   ```json
   {
     "scripts": {
       "generate:prisma": "prisma generate",
       "generate:nexus": "cross-env NODE_ENV=development ts-node --transpile-only path/to/schema/module",
       "generate": "npm -s run generate:prisma && npm -s run generate:nexus",
       "postinstall": "npm -s run generate"
     }
   }
   ```

1. In your deployment pipeline you may wish to run a build step. Heroku buildpacks for example call `npm run build` if that script is defined in your `package.json`. If this is your case and you are a TypeScript user consider a build setup as follows. Prior to `tsc` we run artifact generation so that TypeScript will have types for the all the resolver signatures etc. of your app.

   ```json
   {
     "scripts": {
       "build": "npm -s run generate && tsc"
     }
   }
   ```

## Legacy Note {docsify-ignore}

If you are still using `nexus-prisma@0.3` / Prisma 1 you can find the old docs [here](https://github.com/graphql-nexus/schema/blob/8cf2d6b3e22a9dec1f7c23f384bf33b7be5a25cc/docs/database-access-with-prisma.md). Please note that `nexus-prisma@0.3` is only compatible with `nexus` up to `0.12.0-beta.14` ([issue](https://github.com/graphql-nexus/nexus-prisma/issues/520)). We do not currently have plans to update `nexus-prisma@0.3` to become compatible with versions of `nexus` newer than that.

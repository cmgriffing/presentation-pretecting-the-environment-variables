# Solutions

## Options

- T3 Env [https://env.t3.gg/](https://env.t3.gg/)
- geller [https://github.com/cmgriffing/geller](https://github.com/cmgriffing/geller)

## T3 Env (implementation)

```typescript
// src/env.mjs
import { createEnv } from "@t3-oss/env-nextjs";
import { z } from "zod";

export const env = createEnv({
  server: {
    DATABASE_URL: z.string().url(),
    OPEN_AI_API_KEY: z.string().min(1),
  },
  client: {
    NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY: z.string().min(1),
  },
  runtimeEnv: {
    DATABASE_URL: process.env.DATABASE_URL,
    OPEN_AI_API_KEY: process.env.OPEN_AI_API_KEY,
    NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY:
      process.env.NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY,
  },
});
```

<div class="notes">
Inspired by a pattern made famous by Matt Pocock

Total TypeScript is great (especially if your company will foot the bill)

</div>

## T3 Env (usage)

```typescript
// src/app/hello/route.ts
import { env } from "../env.mjs";

export const GET = (req: Request) => {
  const DATABASE_URL = env.DATABASE_URL;
  // use it...
};
```

## T3 Env (thoughts)

- awesome for new projects
- harder to enforce for old projects
  - nothing prevents a wild `process.env` (without an eslint rule)

## geller (core)

- (compile/build)-time
- uses typescript
- supports dotenv

```typescript
geller scan -g ./foo.ts
```

## geller details (process nodes)

```typescript
export function getUsedEnvVarsSync(
  globs: string[],
  { envs, }: { envs?: string[]; }
) {
  const project = new Project({});
  project.addSourceFilesAtPaths(globs);

  const processEnvNodes: Identifier[] = [];

  project.getSourceFiles().forEach((sourceFile, index) => {
    const identifierNodes = sourceFile.getDescendantsOfKind(
      SyntaxKind.Identifier
    );

    //
    const fileProcessNodes = identifierNodes.filter(
      (identifier) => identifier.getText() === "process"
    );

    if (fileProcessNodes.length) {
      const fileProcessEnvNodes = fileProcessNodes
        .map((processIdentifier) =>
          processIdentifier
            .getNextSiblingIfKind(SyntaxKind.DotToken)
            ?.getNextSiblingIfKind(SyntaxKind.Identifier)
        )
        .filter((envIdentifier) => typeof envIdentifier !== "undefined")
        .filter(
          (envIdentifier) => envIdentifier?.getText() === "env"
        ) as Identifier[];

      if (fileProcessEnvNodes.length) {
        processEnvNodes.push(...fileProcessEnvNodes);
      }
    }

    //...
```

## geller details (process nodes)

```typescript
processEnvNodes.forEach((envNode) => {
  const varNode = envNode
    .getParent()
    .getNextSiblingIfKind(SyntaxKind.DotToken)
    ?.getNextSiblingIfKind(SyntaxKind.Identifier);

  if (varNode) {
    // basic: process.env.FOO
    basicEnvVars.push(varNode.getText());
  } else if (
    envNode?.getParent()?.getParent()?.isKind(SyntaxKind.VariableDeclaration)
  ) {
    // destructured: const { FOO } = process.env
    const syntaxList = envNode
      .getParent()
      ?.getParent()
      ?.getFirstDescendantByKind(SyntaxKind.SyntaxList);

    const identifiers = syntaxList
      ?.getDescendantsOfKind(SyntaxKind.Identifier)
      .map((identifier) => identifier.getText())
      .filter((identifier) => !!identifier) as string[];

    if (identifiers.length) {
      destructuredEnvVars.push(...identifiers);
    }
  }

  // ...
```

## Which musical instrument is eco-friendly and yet contains CO2?

## An air guitar.

![](./assets/air-guitar2.gif)

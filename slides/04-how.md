# How

## Tools

- TS
- unist
- ts-morph
- TreeSitter

## TS

Literally "typescript"

```typescript
handlers
  // ... other transformers
  .map((handlerPath) => {
    // parse http handlers using ts
    const fileName = handlerPath.substring(handlerPath.lastIndexOf("/"));
    const sourceText = fs.readFileSync(handlerPath, { encoding: "utf8" });

    return ts.createSourceFile(fileName, sourceText, ts.ScriptTarget.Latest);
  })
  .map((sourceFile) => {
    // map parsed handlers and extract metadata from Route
    ts.forEachChild(sourceFile, (node) => {
      if (ts.isClassDeclaration(node)) {
        const routeMetadata = getRouteMetadata(node, sourceFile);
        if (routeMetadata.length) {
          routeOptions.push(...routeMetadata);
        }
      }
    });
  });
```

## unist

Universal Syntax Tree

```typescript
interface Node {
  type: string
  data: Data?
  position: Position?
}
```

<div class="notes">
lots of utilities (32 at this time)

find, find-after, flatmap, visit, reduce

issue with latest version needing esm support. just use older version if needed

</div>

## unist (cont.)

> unist is not intended to be self-sufficient. Instead, it is expected that other specifications implement unist and extend it to express language specific nodes.

- `hast` for HTML
- `mdast` for Markdown
- `xast` for XML
- `sast` for CSS, SCSS, Less

## ts-morph

[https://ts-morph.com/](https://ts-morph.com/)

Most useful for transformations (codemods)

NOT unist compatible

## TreeSitter

> Dependency-free so that the runtime library (which is written in pure C) can be embedded in any application

Many supported languages

WASM output available\*

[](./assets/tree-sitter-small.png)

<div class="notes">
WASM output does not have language parity with core (some just fail to build)
</div>

## What kind of environment do hobbits live in?

## A hobbitat.

![](./assets/hobbits.gif)

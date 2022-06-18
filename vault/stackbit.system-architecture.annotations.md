---
id: 244wzvzc0dha8wknfu73kfz
title: Annotations System
desc: ''
updated: 1654371199783
created: 1654362100433
---


## Annotations Tree Extractor

Source: `@stackbit-app/src/snippet/processors/HtmlProcessor/annotation-tree-extractor.js`

The tree extractor is the entry point to the annotations system, and is part of the `HtmlProcessor` module. Once the DOM is ready, pass the global `document` object into `extractAnnotationTree`. The results are then fed into the `resolveObjectsWithAnnotationTree` method 

```ts
export interface Sourcemaps {
  [xpath: string]: ElementSourcemap;
}

export interface ElementSourcemap {
  file: string;
  start?: ElementPosition;
  end?: ElementPosition;
}

export interface ElementPosition {
  line: number;
  column: number;
}

export interface AnnotationsTree {
  objectIds: string[];
  children: AnnotationsTreeNode[];
}

export interface AnnotationsTreeNode {
  oid: string | null;
  fp: AnnotationsTreeFieldPath[];
  xpath: string[];
  children: AnnotationsTreeNode[];
}

export interface AnnotationsTreeFieldPath {
  oid: string | null;
  fp: string;
  loc: string;
  hasOnlyTextNodes: boolean;
}

export interface AnnotationError {
  message: string;
  elementXPath: string;
}

export function extractAnnotationTree(document: Document): {
  sourcemaps: Sourcemaps;
  annotationTree: AnnotationsTree;
  errors: AnnotationError[];
};
```

### Example

```js
{
  objectIds: ['obj1', 'obj3'],
  children: [
    {
      oid: "obj1",
      fp: [],
      xpath: ["/html", "node"],
      children: [
        {
          oid: null,
          fp: [
            { oid: null, fp: "author.title", loc: "", hasOnlyTextNodes: true },
          ],
          xpath: ["/html", "node[1]", "inner[1]"],
        },
        {
          oid: null,
          fp: [{ oid: "obj3", fp: "title", loc: "" }],
          xpath: ["/html", "node[1]", "inner[2]"],
        },
      ],
    },
  ],
}
```

### findDescendantElements

```ts
export type Descendant = { node: Element; xpath: string[] };

function findDescendantElements(
  node: Element,
  xpath: string[],
  iteratee: (childNode: Element, xpath: string[]) => boolean,
  options?: { recursive: boolean }
): Descendant[];
```

Helper function that when given a DOM node and an XPath, return a list of DOM descendants nodes along with their absolute XPath.

Elements are only added when the given iteratee function returns true.

## Annotation Tree Parser

Test: `packages/dev-common/__tests__/annotation-tree-parser.test.js`

### resolveObjectsWithAnnotationTree

```ts
function resolveObjectsWithAnnotationTree(
  annotationTree: AnnotationsTree,
  fieldData: any[],
  { resolveAllReferences: boolean }?
): { pathMap: {}; objects; errors: AnnotationError[] };
```

Receive a list of objectIds, and the annotationTree, and resolves the annotations into a pathMap, and returns any additional referenced objects that were used in the annotations. Also returns validation errors to the client.


### parseAnnotationTree

```ts
function parseAnnotationTree(
  annotationTree,
  fieldData,
  { parentOid, parentFp }?
): { pathMap: {}; pathMapForResolving: {}; errors: any[] };
```

Recursion that goes over annotationTree, and parses out annotations using the fieldData to resolve and infer objectIds and partial fieldPaths (.field).
Steps:

1. parse 'data-sb-object-id' annotations and store context in parentOid to append to child annotations if they don't specify objectId explicitly
2. parse 'data-sb-field-path' annotations (using resolveFp to resolve partials (.field), and parent objectIds), and create a pathMap that maps fieldPaths to xPaths.
2.1. determine if any of the annotations is a container annotation and store it in context to pass to descendants.
3. go over child nodes and run the recursion, merging any returned validation errors, and xpath matches.

Return any resolved annotations as a pathMap,
also returns pathMapForResolving, which includes partial fieldPaths for resolving references from any ancestor objectId.

### resolveMatchType


```ts
function resolveMatchType(location): string
```

receives the xpath location, and resolves the matchType for highlighting (text, element, attribute)

### resolveLocation


```ts
function resolveLocation(annotationFp, field): string[];
```

parses the xpath location partial of an annotation ('objectId:field.path#location') and resolves defaults and fallbacks to return the final xpath.

## Annotation Utils

### fieldPathToFieldDataPath

This method translates fieldPath (object.sections[0].title) to fieldDataPath (object.fields.sections.items[0].fields.title) It follows references, and returns both the fieldDataPath from the nearest object that has an ID, but also the fieldDataPath from the original passed object

FieldDataPathsFromRoot contains fieldPaths to current fieldPathStr from every object in the reference chain. example: Oid1.fields.sections.items.1.fields.features.items.0.fields.featureTitle Oid2.fields.features.items.0.fields.featureTitle Oid3.fields.featureTitle all three fieldpaths are required to resolve every object in the reference chain

```typescript
function fieldPathToFieldDataPath(
  objectId,
  fieldPathStr,
  fieldData
): { fieldDataPathsFromRoot: string[]; fieldDataPath: string } | {};
```

### fieldPathToString

This converts a fieldPath array to a string, it puts complex strings inside single quotes '', and uses square brackets [] for number keys.

```typescript
function fieldPathToString(fieldPath: string[]): string;
```

### appendOrArray

Append item to an array in object given a path, or initialize if it doesn't exist.

```typescript
function appendOrArray(object, path, item): void;
```

## Annotation Service

Source: `@stackbit/stackbit-dev/packages/stackbit-dev/src/services/annotation-errors.ts`

### logAnnotationErrors

```ts
export type AnnotationError = {
  message: string;
    elementXPath: string;
}

export type SourceMap = {
  file: string;
    start?: { line: number }
}

export async function logAnnotationErrors(
  errors: any[],
  logger: any,
  baseDir: string,
  sourcemaps?: { [p: string]: any } | undefined
): Promise<void>;
```

Serialize a list of `AnnotationError` objects and send to the provided logger.

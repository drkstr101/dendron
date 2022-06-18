
## Definitions

Item 1
: Definition for Item 1

Item 2
: Definition for Item 2
: Another definition for Item 2, with a [link](http://www.example.com)



## Implementation

### Presets Service

Source: [presets-service.ts](https://github.com/stackbit/stackbit/blob/master/packages/dev-common/src/services/presets-service.ts)

Test: [preset-service.test.js](https://github.com/stackbit/stackbit/blob/master/packages/dev-common/__tests__/presets-service.test.js)

#### API

```ts
export interface PresetDefinition {
  label: string;
  metadata: any;
  thumbnail: string;
}

export interface ReferenceBehavior {
  behavior?: "copyReference" | "duplicateContents";
  nonDuplicatableModels?: string[];
  duplicatableModels?: string[];
}
```

Write the `PresetDefinition` to a presets directory then return the result:

```ts
interface CreatePresetResponse {
  files: string[];
  preset: {
    modelName: any;
    thumbnail: string | undefined;
    data: any;
    label: string;
    metadata: any;
  };
}

export async function createPreset(
  dir: string,
  fieldData: any,
  fieldDataPath: string[],
  preset: PresetDefinition,
  referenceBehavior?: ReferenceBehavior | undefined,
  logger?: any,
  dryRun?: boolean
): Promise<CreatePresetResponse>;
```

Delete a given preset id and return a list of deleted preset ids:

```ts
export async function deletePreset(
  dir: string,
  presetId: string,
  logger?: any
): Promise<string[]>;
```

### Preset Loader

Source: [presets-loader.ts](https://github.com/stackbit/stackbit/blob/44ac86a2dedd4399269e11c5cd048522e0fe8fb7/packages/stackbit-sdk/src/config/presets-loader.ts)

Helper function for loading presets from disk


```ts
export interface PresetsLoaderResult {
  config: Config;
  errors: ConfigPresetsError[];
}

export async function loadPresets(
  dirPath: string,
  config: Config
): Promise<PresetsLoaderResult>;
```

### Existing UI Components

- [src/components/studio/AddEntityModal](https://github.com/stackbit/stackbit-app/blob/1a055df1c0ee409e0ed9f4a80414e33aa084badf/src/components/studio/AddEntityModal)
- [src/components/studio/common/AddEntity](https://github.com/stackbit/stackbit-app/blob/1a055df1c0ee409e0ed9f4a80414e33aa084badf/src/components/studio/common/AddEntity)
- [src/components/studio/CreatePage](https://github.com/stackbit/stackbit-app/blob/1a055df1c0ee409e0ed9f4a80414e33aa084badf/src/components/studio/CreatePage) (unused)
- [src/components/studio/SavePreset](https://github.com/stackbit/stackbit-app/blob/1a055df1c0ee409e0ed9f4a80414e33aa084badf/src/components/studio/SavePreset)
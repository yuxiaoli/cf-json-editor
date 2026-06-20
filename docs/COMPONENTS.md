# Component API Reference

## `App.vue`
The root component of the application.

### State
- `jsonData`: `Ref<any>` - The global JSON data object.
- `rawJson`: `Ref<string>` - The stringified version of `jsonData` for the Ace Editor.
- `isEditMode`: `Ref<boolean>` - Global toggle for view/edit states.
- `jsonError`: `Ref<string | null>` - Holds parsing errors from the raw JSON view.

### Methods
- `handleFileUpload(event: Event)`: Reads a local `.json` file using the FileReader API.
- `exportJson()`: Generates a Blob from `jsonData` and triggers a browser download.
- `applyRawJson()`: Parses `rawJson` and updates `jsonData`.

---

## `JsonRenderer.vue`
A recursive component responsible for rendering a specific node of a JSON tree.

### Props
- `modelValue`: `any` (Required) - The current JSON node value.
- `nodeKey`: `string | number` (Optional) - The key or index of the current node.
- `isEditMode`: `boolean` (Default: `false`) - Determines whether to render inputs or static badges.

### Emits
- `update:modelValue(value: any)` - Fired when the node's value changes.
- `delete()` - Fired when the user deletes this node (object property or array item).
- `rename(oldKey: string, newKey: string)` - Fired when an object key is renamed.

### Internal Logic
- **Type Inference**: Computes `nodeType` (string, number, boolean, null, array, object).
- **Mutations**: 
  - `updateValue(val)`: Emits simple value changes.
  - `changeType(newType)`: Coerces the current value into a new JSON type and emits it.
  - `addChild()`: Adds a new key to an Object or pushes an item to an Array.
  - `updateChildKey()` / `updateChildValue()`: Handles mutations bubbling up from recursive children.

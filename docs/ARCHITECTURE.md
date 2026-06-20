# Architecture Overview

The CF JSON Editor is designed as a standalone, client-side heavy application with optional backend capabilities. The application follows a dual-panel layout pattern and utilizes recursive Vue components for dynamic rendering.

## High-Level Architecture

```text
+-------------------------------------------------------+
|                       App.vue                         |
|  (Global State, File I/O, Mode Toggling, Syncing)     |
|                                                       |
|  +----------------------+  +-----------------------+  |
|  |   Visual Editor      |  |     Raw JSON Editor   |  |
|  |  (JsonRenderer.vue)  |  |     (Ace Editor)      |  |
|  +----------|-----------+  +-----------|-----------+  |
|             |                          |              |
|             v                          v              |
|      v-model updates           String parsed/synced   |
+-------------------------------------------------------+
```

## Core Components

### 1. State Management (`App.vue`)
- Holds the master `jsonData` state.
- Manages the `isEditMode` flag.
- Handles browser File I/O for the **Import** and **Export** features.
- Syncs state changes between the `JsonRenderer` and the Ace Editor instance.

### 2. Recursive Renderer (`JsonRenderer.vue`)
This is the heart of the UI form builder. It takes a JSON payload and dynamically infers its type (`typeof` and `Array.isArray()`).
- **Primitives**: Renders native inputs (text, numbers, boolean switches).
- **Complex Types**: Iterates over Object keys or Array indices, recursively rendering another `JsonRenderer` instance for each child node.
- **Mutations**: Emits `update:modelValue` events upwards, ensuring Vue's one-way data flow is respected while updating deep object trees.

### 3. Raw JSON Editor
- Utilizes `vue3-ace-editor` with JSON syntax highlighting.
- Becomes read-only when `isEditMode` is toggled off.

## Deployment Architecture

Currently, the application is purely static:
- **Build**: Vite compiles the Vue 3 application into static HTML/CSS/JS.
- **Hosting**: GitHub Pages serves the `/dist` output.
- **Backend**: A scaffolded Cloudflare Worker exists in `src/index.ts`. It is currently a placeholder for future features (e.g., saving schemas to a database, sharing JSON links, or validating payloads server-side).

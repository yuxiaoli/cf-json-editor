# CF JSON Editor

A schema-less JSON UI Renderer and JSON-driven Form Builder, built with Vue 3, Vite, Tailwind CSS, and shadcn-vue. 

This project provides a complete local workflow for visualizing, editing, and managing JSON data without requiring predefined schemas or a backend. It features a dual-panel interface with a recursive visual UI builder and a raw Ace editor.

## Features

- **Schema-less Rendering**: Dynamically parses any valid JSON and maps it to interactive UI components.
- **Recursive UI Components**: Supports deeply nested objects and arrays.
- **Two-Way Synchronization**: Real-time sync between the visual UI builder and the raw JSON Ace editor.
- **Edit & View Modes**: Toggle between a read-only data viewer and an interactive data mutation interface.
- **Type Casting & Coercion**: Change JSON node types dynamically (String, Number, Boolean, Null, Object, Array).
- **Import/Export**: Load `.json` files locally via browser APIs and export edited configurations.
- **Ace Editor Integration**: Syntax highlighting and validation for raw JSON manipulation.
- **Cloudflare Ready**: Initialized with Cloudflare Workers configuration for optional backend extensibility.

## Project Structure

- `/frontend` - The Vue 3 SPA containing the JSON UI Renderer.
  - `/src/components/JsonRenderer.vue` - Core recursive rendering engine.
  - `/src/App.vue` - Main application layout and state management.
- `/src` - Cloudflare Worker backend entry points.
- `/docs` - Detailed architectural and component documentation.

## Getting Started

### Frontend Development

1. Navigate to the frontend directory:
   ```bash
   cd frontend
   ```
2. Install dependencies:
   ```bash
   npm install
   ```
3. Start the development server:
   ```bash
   npm run dev
   ```

### Backend (Cloudflare Worker)

1. Install root dependencies:
   ```bash
   npm install
   ```
2. Start the local Wrangler server:
   ```bash
   npx wrangler dev
   ```

## Deployment

The frontend is configured for automatic deployment to GitHub Pages via GitHub Actions (`.github/workflows/pages.yml`). Any push to `main` or `develop` triggers the deployment.

Live Demo: [https://yuxiaoli.github.io/cf-json-editor/](https://yuxiaoli.github.io/cf-json-editor/)

## Tech Stack

- **Framework**: Vue 3 (Composition API)
- **Build Tool**: Vite
- **Styling**: Tailwind CSS
- **UI Components**: shadcn-vue / reka-ui
- **Editor**: Ace Editor (`vue3-ace-editor`)
- **Backend (Optional)**: Cloudflare Workers

<script setup lang="ts">
import { ref, watch } from 'vue'
import { Upload, Download, Code, LayoutTemplate } from 'lucide-vue-next'
import { VAceEditor } from 'vue3-ace-editor'
import 'ace-builds/src-noconflict/mode-json'
import 'ace-builds/src-noconflict/theme-chrome'
import { Button } from '@/components/ui/button'
import { Switch } from '@/components/ui/switch'
import { Label } from '@/components/ui/label'
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/tabs'
import JsonRenderer from '@/components/JsonRenderer.vue'

const jsonData = ref<any>({
  name: "CF JSON Editor",
  version: 1,
  active: true,
  features: ["visual-editing", "raw-json", "import-export"],
  settings: {
    theme: "dark",
    autoSave: null
  }
})

const rawJson = ref(JSON.stringify(jsonData.value, null, 2))
const jsonError = ref('')
const isEditMode = ref(true)
const activeTab = ref('visual') // mobile view tabs
const aceEditorOptions = {
  useWorker: false,
  tabSize: 2,
  fontSize: 14,
  showPrintMargin: false,
}

// Sync Visual Editor -> Raw JSON
watch(jsonData, (newVal) => {
  try {
    rawJson.value = JSON.stringify(newVal, null, 2)
    jsonError.value = ''
  } catch (e: any) {
    jsonError.value = 'Failed to stringify JSON: ' + e.message
  }
}, { deep: true })

// Apply Raw JSON -> Visual Editor
function applyRawJson() {
  try {
    const parsed = JSON.parse(rawJson.value)
    jsonData.value = parsed
    jsonError.value = ''
  } catch (e: any) {
    jsonError.value = 'Invalid JSON: ' + e.message
  }
}

function handleDesktopRawJsonInput() {
  if (!isEditMode.value) {
    return
  }
  applyRawJson()
}

// Import JSON
function importJson() {
  const input = document.createElement('input')
  input.type = 'file'
  input.accept = 'application/json'
  input.onchange = (e: any) => {
    const file = e.target.files[0]
    if (!file) return
    const reader = new FileReader()
    reader.onload = (event) => {
      try {
        const content = event.target?.result as string
        const parsed = JSON.parse(content)
        jsonData.value = parsed
        jsonError.value = ''
      } catch (err: any) {
        alert('Invalid JSON file: ' + err.message)
      }
    }
    reader.readAsText(file)
  }
  input.click()
}

// Export JSON
function exportJson() {
  // Ensure we are exporting the latest valid JSON
  let content = ''
  try {
    // try to use rawJson if it's valid, otherwise fallback to jsonData
    JSON.parse(rawJson.value)
    content = rawJson.value
  } catch {
    content = JSON.stringify(jsonData.value, null, 2)
  }
  
  const blob = new Blob([content], { type: 'application/json' })
  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = 'data.json'
  document.body.appendChild(a)
  a.click()
  document.body.removeChild(a)
  URL.revokeObjectURL(url)
}
</script>

<template>
  <div class="min-h-screen bg-background flex flex-col font-sans">
    
    <!-- Navbar -->
    <header class="border-b border-border bg-card px-6 py-4 flex items-center justify-between sticky top-0 z-10">
      <div class="flex items-center gap-2">
        <div class="bg-primary text-primary-foreground p-1.5 rounded-md">
          <LayoutTemplate class="w-5 h-5" />
        </div>
        <h1 class="text-xl font-bold tracking-tight">CF JSON Editor</h1>
      </div>
      
      <div class="flex items-center gap-6">
        <div class="flex items-center space-x-2">
          <Switch
            id="edit-mode"
            :key="`edit-mode-${String(isEditMode)}`"
            :model-value="isEditMode"
            @update:model-value="(value) => { isEditMode = value === true }"
          />
          <Label htmlFor="edit-mode" class="cursor-pointer">{{ isEditMode ? 'Edit Mode' : 'View Mode' }}</Label>
        </div>
        
        <div class="flex items-center gap-2">
          <Button variant="outline" size="sm" @click="importJson">
            <Upload class="w-4 h-4 mr-2" /> Import
          </Button>
          <Button variant="default" size="sm" @click="exportJson">
            <Download class="w-4 h-4 mr-2" /> Export
          </Button>
        </div>
      </div>
    </header>

    <!-- Main Content -->
    <main class="flex-1 flex overflow-hidden">
      <!-- Desktop: Split View -->
      <div class="hidden md:flex w-full h-[calc(100vh-73px)]">
        
        <!-- Left: Visual Editor -->
        <div class="flex-1 border-r border-border overflow-y-auto p-6 bg-muted/20">
          <div class="max-w-3xl mx-auto bg-card rounded-lg border shadow-sm p-6">
            <h2 class="text-lg font-semibold mb-4 flex items-center gap-2">
              <LayoutTemplate class="w-5 h-5 text-muted-foreground" />
              Visual Editor
            </h2>
            <JsonRenderer 
              v-model="jsonData" 
              :isEditMode="isEditMode" 
              :isRoot="true" 
            />
          </div>
        </div>

        <!-- Right: Raw JSON -->
        <div class="w-1/3 min-w-[400px] flex flex-col bg-card">
          <div class="p-4 border-b border-border flex items-center justify-between bg-muted/40">
            <h2 class="text-sm font-semibold flex items-center gap-2">
              <Code class="w-4 h-4" /> Raw JSON
            </h2>
            <Button size="sm" @click="applyRawJson" :disabled="!isEditMode || !!jsonError">Apply Changes</Button>
          </div>
          <div class="flex-1 p-4 flex flex-col">
            <div class="flex-1 overflow-hidden rounded-md border border-input" :class="{ 'opacity-70': !isEditMode }">
              <VAceEditor
                v-model:value="rawJson"
                lang="json"
                theme="chrome"
                class="h-full w-full font-mono"
                :readonly="!isEditMode"
                :options="aceEditorOptions"
                :print-margin="false"
                :placeholder="isEditMode ? 'Paste JSON here...' : 'Raw JSON editing is disabled in View Mode.'"
                @update:value="handleDesktopRawJsonInput"
              />
            </div>
            <div v-if="jsonError" class="mt-2 text-sm text-destructive bg-destructive/10 p-2 rounded border border-destructive/20">
              {{ jsonError }}
            </div>
          </div>
        </div>
      </div>

      <!-- Mobile: Tabs View -->
      <div class="md:hidden w-full p-4 h-[calc(100vh-73px)] overflow-y-auto">
        <Tabs v-model="activeTab" class="w-full">
          <TabsList class="grid w-full grid-cols-2 mb-4">
            <TabsTrigger value="visual">Visual UI</TabsTrigger>
            <TabsTrigger value="raw">Raw JSON</TabsTrigger>
          </TabsList>
          
          <TabsContent value="visual" class="m-0">
            <div class="bg-card rounded-lg border shadow-sm p-4">
              <JsonRenderer 
                v-model="jsonData" 
                :isEditMode="isEditMode" 
                :isRoot="true" 
              />
            </div>
          </TabsContent>
          
          <TabsContent value="raw" class="m-0 flex flex-col h-[calc(100vh-180px)]">
            <div class="mb-4 flex-1 overflow-hidden rounded-md border border-input" :class="{ 'opacity-70': !isEditMode }">
              <VAceEditor
                v-model:value="rawJson"
                lang="json"
                theme="chrome"
                class="h-full w-full font-mono"
                :readonly="!isEditMode"
                :options="aceEditorOptions"
                :print-margin="false"
                :placeholder="isEditMode ? 'Paste JSON here...' : 'Raw JSON editing is disabled in View Mode.'"
              />
            </div>
            <Button @click="applyRawJson" class="w-full mb-2" :disabled="!isEditMode || !!jsonError">Apply Changes</Button>
            <div v-if="jsonError" class="text-sm text-destructive bg-destructive/10 p-2 rounded border border-destructive/20">
              {{ jsonError }}
            </div>
          </TabsContent>
        </Tabs>
      </div>

    </main>
  </div>
</template>

<script setup lang="ts">
import { computed, ref } from 'vue'
import { Plus, Trash2, Edit2, Check, X } from 'lucide-vue-next'
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import { Badge } from '@/components/ui/badge'
import { Switch } from '@/components/ui/switch'
import { Card, CardContent } from '@/components/ui/card'
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select'

const props = defineProps<{
  modelValue: any
  name?: string
  isEditMode: boolean
  isRoot?: boolean
}>()

const emit = defineEmits<{
  (e: 'update:modelValue', value: any): void
  (e: 'remove'): void
  (e: 'rename', newName: string): void
}>()

const valueType = computed(() => {
  if (props.modelValue === null) return 'null'
  if (Array.isArray(props.modelValue)) return 'array'
  return typeof props.modelValue
})

const isArrayOfPrimitives = computed(() => {
  if (valueType.value !== 'array') return false
  if (props.modelValue.length === 0) return true
  return props.modelValue.every((item: any) => typeof item !== 'object' || item === null)
})

const isEditingName = ref(false)
const editNameInput = ref(props.name || '')
const nameError = ref('')

const booleanModel = computed(() => props.modelValue === true)

function updateBooleanValue(val: unknown) {
  updateValue(val === true)
}

function handleTypeChange(newType: any) {
  if (typeof newType !== 'string') return
  let newValue: any
  switch (newType) {
    case 'string': newValue = ''; break
    case 'number': newValue = 0; break
    case 'boolean': newValue = false; break
    case 'null': newValue = null; break
    case 'array': newValue = []; break
    case 'object': newValue = {}; break
    default: newValue = ''; break
  }
  emit('update:modelValue', newValue)
}

function updateValue(val: any) {
  emit('update:modelValue', val)
}

function updateObjectKey(oldKey: string, newKey: string) {
  if (oldKey === newKey) return
  if (!newKey.trim()) return
  const obj = { ...props.modelValue }
  if (newKey in obj) return // Prevent duplicate
  
  // Preserve order
  const newObj: Record<string, any> = {}
  for (const k in obj) {
    if (k === oldKey) {
      newObj[newKey] = obj[k]
    } else {
      newObj[k] = obj[k]
    }
  }
  emit('update:modelValue', newObj)
}

function removeObjectKey(key: string) {
  const obj = { ...props.modelValue }
  delete obj[key]
  emit('update:modelValue', obj)
}

function addObjectKey() {
  const obj = { ...props.modelValue }
  let newKey = 'newKey'
  let counter = 1
  while (newKey in obj) {
    newKey = `newKey${counter++}`
  }
  obj[newKey] = ''
  emit('update:modelValue', obj)
}

function updateArrayItem(index: number | string, val: any) {
  const arr = [...props.modelValue]
  arr[Number(index)] = val
  emit('update:modelValue', arr)
}

function removeArrayItem(index: number | string) {
  const arr = [...props.modelValue]
  arr.splice(Number(index), 1)
  emit('update:modelValue', arr)
}

function addArrayItem() {
  const arr = [...props.modelValue]
  arr.push('')
  emit('update:modelValue', arr)
}

function startRename() {
  editNameInput.value = props.name || ''
  isEditingName.value = true
  nameError.value = ''
}

function confirmRename() {
  const newName = editNameInput.value.trim()
  if (!newName) {
    nameError.value = 'Key cannot be empty'
    return
  }
  emit('rename', newName)
  isEditingName.value = false
}

function cancelRename() {
  isEditingName.value = false
}

</script>

<template>
  <div class="json-renderer flex flex-col gap-2 relative group w-full" :class="{ 'pl-4 border-l-2 border-border/50': !isRoot }">
    <!-- Header: Key Name + Type Switcher + Actions -->
    <div class="flex items-center gap-2 flex-wrap min-h-[28px]">
      
      <!-- Key Name -->
      <template v-if="name !== undefined">
        <div v-if="isEditMode && isEditingName" class="flex items-center gap-1">
          <Input v-model="editNameInput" class="h-7 w-32 text-sm" @keyup.enter="confirmRename" @keyup.esc="cancelRename" />
          <Button variant="ghost" size="icon" class="h-7 w-7 text-green-600" @click="confirmRename"><Check class="w-4 h-4" /></Button>
          <Button variant="ghost" size="icon" class="h-7 w-7 text-destructive" @click="cancelRename"><X class="w-4 h-4" /></Button>
        </div>
        <div v-else class="flex items-center gap-1">
          <span class="font-semibold text-sm text-primary">{{ name }}</span>
          <Button v-if="isEditMode" variant="ghost" size="icon" class="h-6 w-6 opacity-0 group-hover:opacity-100 transition-opacity" @click="startRename">
            <Edit2 class="w-3 h-3" />
          </Button>
        </div>
        <span class="text-muted-foreground mr-2">:</span>
      </template>

      <!-- Type Indicator / Switcher -->
      <template v-if="isEditMode">
        <Select :modelValue="valueType" @update:modelValue="handleTypeChange">
          <SelectTrigger class="h-7 w-28 text-xs">
            <SelectValue />
          </SelectTrigger>
          <SelectContent>
            <SelectItem value="string">String</SelectItem>
            <SelectItem value="number">Number</SelectItem>
            <SelectItem value="boolean">Boolean</SelectItem>
            <SelectItem value="object">Object</SelectItem>
            <SelectItem value="array">Array</SelectItem>
            <SelectItem value="null">Null</SelectItem>
          </SelectContent>
        </Select>
      </template>
      <Badge v-else variant="outline" class="text-[10px] uppercase">{{ valueType }}</Badge>

      <!-- Remove Self Action -->
      <Button v-if="isEditMode && !isRoot" variant="ghost" size="icon" class="h-7 w-7 text-destructive opacity-0 group-hover:opacity-100 transition-opacity ml-auto" @click="$emit('remove')">
        <Trash2 class="w-4 h-4" />
      </Button>
    </div>

    <div v-if="nameError" class="text-xs text-destructive">{{ nameError }}</div>

    <!-- Values -->
    <div class="w-full mt-1">
      
      <!-- Null -->
      <div v-if="valueType === 'null'">
        <Badge variant="secondary">null</Badge>
      </div>

      <!-- Boolean -->
      <div v-else-if="valueType === 'boolean'">
        <Switch
          v-if="isEditMode"
          :key="`${name ?? 'boolean'}-${String(booleanModel)}`"
          :model-value="booleanModel"
          @update:model-value="updateBooleanValue"
        />
        <Badge v-else :variant="modelValue ? 'default' : 'secondary'" class="text-xs">{{ modelValue ? 'True' : 'False' }}</Badge>
      </div>

      <!-- String -->
      <div v-else-if="valueType === 'string'">
        <Input v-if="isEditMode" :modelValue="modelValue" @update:modelValue="updateValue" class="h-8" />
        <span v-else class="text-sm text-green-600 dark:text-green-400 break-all">"{{ modelValue }}"</span>
      </div>

      <!-- Number -->
      <div v-else-if="valueType === 'number'">
        <Input v-if="isEditMode" type="number" :modelValue="modelValue" @update:modelValue="updateValue($event === '' ? 0 : Number($event))" class="h-8 w-48" />
        <Badge v-else variant="outline" class="text-blue-600 dark:text-blue-400 border-blue-200 dark:border-blue-900 bg-blue-50 dark:bg-blue-950/30">{{ modelValue }}</Badge>
      </div>

      <!-- Object -->
      <Card v-else-if="valueType === 'object'" class="mt-2 w-full border-muted/60 shadow-sm">
        <CardContent class="p-3 flex flex-col gap-3">
          <div v-if="Object.keys(modelValue).length === 0" class="text-xs text-muted-foreground italic">Empty object</div>
          <template v-for="(val, key) in modelValue" :key="key">
            <JsonRenderer
              :modelValue="val"
              :name="String(key)"
              :isEditMode="isEditMode"
              @update:modelValue="(newVal) => { const obj = {...modelValue}; obj[key] = newVal; updateValue(obj) }"
              @remove="removeObjectKey(String(key))"
              @rename="(newKey) => updateObjectKey(String(key), newKey)"
            />
          </template>
          <Button v-if="isEditMode" variant="outline" size="sm" class="w-fit mt-2 h-7 text-xs" @click="addObjectKey">
            <Plus class="w-3 h-3 mr-1" /> Add Property
          </Button>
        </CardContent>
      </Card>

      <!-- Array -->
      <div v-else-if="valueType === 'array'" class="mt-2 w-full">
        <Card class="border-muted/60 shadow-sm">
          <CardContent class="p-3 flex flex-col gap-3">
            <div v-if="modelValue.length === 0" class="text-xs text-muted-foreground italic">Empty array</div>
            
            <!-- View Mode: Array of Primitives -> Dropdown -->
            <div v-else-if="!isEditMode && isArrayOfPrimitives">
              <Select>
                <SelectTrigger class="w-[200px] h-8 text-sm">
                  <SelectValue placeholder="View items..." />
                </SelectTrigger>
                <SelectContent>
                  <SelectItem v-for="(val, index) in modelValue" :key="index" :value="String(index)">
                    {{ val === null ? 'null' : String(val) }}
                  </SelectItem>
                </SelectContent>
              </Select>
            </div>

            <!-- Edit Mode or Array of Objects -> Recursive rendering -->
            <template v-else>
              <div v-for="(val, index) in modelValue" :key="index" class="flex items-start gap-2 relative group/item bg-muted/10 p-2 rounded-md border border-border/40">
                <div class="mt-1.5 min-w-[24px] text-xs text-muted-foreground select-none font-mono">[{{ index }}]</div>
                <div class="flex-1 min-w-0">
                  <JsonRenderer
                    :modelValue="val"
                    :isEditMode="isEditMode"
                    @update:modelValue="(newVal) => updateArrayItem(index, newVal)"
                    @remove="removeArrayItem(index)"
                  />
                </div>
              </div>
            </template>
            
            <Button v-if="isEditMode" variant="outline" size="sm" class="w-fit mt-2 h-7 text-xs" @click="addArrayItem">
              <Plus class="w-3 h-3 mr-1" /> Add Item
            </Button>
          </CardContent>
        </Card>
      </div>

    </div>
  </div>
</template>

<script setup lang="ts">
import { computed, onMounted, ref, watch } from 'vue'

const props = defineProps({
  src: { type: String, default: '' },
  csv: { type: String, default: '' },
  caption: { type: String, default: '' },

  maxRows: { type: Number, default: 0 },
  maxHeight: { type: [String, Number], default: '70vh' },

  zoom: { type: Number, default: 1 },
  cellMaxWidth: { type: String, default: '18ch' },
})

const raw = ref('')
const loading = ref(false)
const error = ref('')

async function load() {
  error.value = ''

  if (props.csv && props.csv.trim()) {
    raw.value = props.csv
    return
  }
  if (!props.src) {
    raw.value = ''
    return
  }

  try {
    loading.value = true
    const res = await fetch(props.src)
    if (!res.ok) throw new Error(`Fetch failed (${res.status})`)
    raw.value = await res.text()
  } catch (e: any) {
    error.value = e?.message ?? String(e)
    raw.value = ''
  } finally {
    loading.value = false
  }
}

onMounted(load)
watch(() => [props.src, props.csv], load)

const table = computed(() => parseCsv(raw.value))
const header = computed(() => table.value[0] ?? [])
const body = computed(() => {
  const rows = table.value.slice(1)
  return props.maxRows > 0 ? rows.slice(0, props.maxRows) : rows
})

function parseCsv(text: string): string[][] {
  if (!text || !text.trim()) return []

  const out: string[][] = []
  let row: string[] = []
  let cell = ''
  let inQuotes = false

  for (let i = 0; i < text.length; i++) {
    const ch = text[i]
    const next = text[i + 1]

    if (ch === '"' && inQuotes && next === '"') { cell += '"'; i++; continue }
    if (ch === '"') { inQuotes = !inQuotes; continue }

    if (!inQuotes && (ch === ',' || ch === '\n' || ch === '\r')) {
      if (ch === '\r' && next === '\n') continue
      row.push(cell); cell = ''
      if (ch === '\n') {
        if (row.some(x => x.trim() !== '')) out.push(row)
        row = []
      }
      continue
    }

    cell += ch
  }

  row.push(cell)
  if (row.some(x => x.trim() !== '')) out.push(row)

  const w = Math.max(...out.map(r => r.length), 0)
  return out.map(r => (r.length < w ? r.concat(Array(w - r.length).fill('')) : r))
}
</script>

<template>
  <div m="0" p="0" :style="{ zoom: String(zoom) }">
    <div v-if="caption" font="mono" text="[10px]" op="60" leading="![1.1]" m="0" mb="[2px]">
      {{ caption }}
    </div>

    <div v-if="error" font="mono" text="[10px] red-4" leading="![1.1]" m="0">
      Error: {{ error }}
    </div>
    <div v-else-if="loading" font="mono" text="[10px]" op="60" leading="![1.1]" m="0">
      Loading…
    </div>

    <div
      v-else
      inline-block
      w="max"
      max-w="full"
      overflow="auto"
      m="0"
      p="0"
      border="~ main/30"
      :style="{ maxHeight: typeof maxHeight === 'number' ? `${maxHeight}px` : maxHeight }"
    >
      <table
        v-if="header.length"
        font="mono"
        text="[11px]"
        leading="![1.1]"
        border="collapse"
        :style="{ tableLayout: 'fixed', width: 'max-content' }"
      >
        <thead>
          <tr>
            <th
            v-for="(h, i) in header"
            :key="i"
            scope="col"
            pos="sticky"
            top="0"
            z="1"
            text="left"
            bg="main/6"
            border="b main/20"
            whitespace="nowrap"
            class="truncate"
            px="!3px"
            py="!1px"
            leading="![1.1]"
            style="font-weight: 700 !important;"
            :style="{ maxWidth: cellMaxWidth }"
            >
            {{ h }}
            </th>
          </tr>
        </thead>

        <tbody>
          <tr v-for="(r, ri) in body" :key="ri">
            <td
              v-for="(c, ci) in r"
              :key="ci"
              text="left"
              border="b main/12"
              whitespace="nowrap"
              class="truncate"
              px="!3px"
              py="!1px"
              leading="![1.1]"
              :style="{ maxWidth: cellMaxWidth }"
              :class="(c ?? '').trim().toUpperCase() === 'NULL' ? 'op-55 italic' : ''"
            >
              {{ c }}
            </td>
          </tr>
        </tbody>
      </table>

      <div v-else font="mono" text="[10px]" op="60" px="!3px" py="!1px" leading="![1.1]" m="0">
        No data
      </div>
    </div>
  </div>
</template>

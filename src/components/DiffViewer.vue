<template>
  <div ref="container" class="diff-viewer" />
</template>

<script setup>
import { ref, onMounted, watch } from 'vue'
import { Diff2HtmlUI } from 'diff2html/lib/ui/js/diff2html-ui-slim'
import 'diff2html/bundles/css/diff2html.min.css'

const props = defineProps({
  patch: { type: String, default: '' },
  outputFormat: { type: String, default: 'side-by-side' },
})

const container = ref(null)

function render() {
  if (!container.value || !props.patch) return
  const ui = new Diff2HtmlUI(container.value, props.patch, {
    drawFileList: true,
    matching: 'lines',
    outputFormat: props.outputFormat,
    highlight: true,
    colorScheme: 'light',
  })
  ui.draw()
  ui.highlightCode()
}

onMounted(render)
watch(() => props.patch, render)
watch(() => props.outputFormat, render)
</script>

<style>
/* Override diff2html to fit our layout */
.diff-viewer .d2h-wrapper { font-size: 0.8rem; }
.diff-viewer .d2h-file-header { background: var(--bg2); border-color: var(--border); }
.diff-viewer .d2h-file-list-wrapper { background: var(--bg); border-color: var(--border); }
</style>

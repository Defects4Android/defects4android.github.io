<template>
  <div v-if="bug" class="bug-page">

    <!-- Nav bar within page -->
    <div class="bug-nav">
      <RouterLink to="/dataset" class="back-link">← Back to Dataset</RouterLink>
      <div class="bug-nav-pager" v-if="prevBug || nextBug">
        <RouterLink v-if="prevBug" :to="`/dataset/${prevBug.bid}`" class="pager-link">← Previous</RouterLink>
        <RouterLink v-if="nextBug" :to="`/dataset/${nextBug.bid}`" class="pager-link">Next →</RouterLink>
      </div>
    </div>

    <!-- Header -->
    <div class="bug-header">
      <div class="bug-id">
        <code>{{ bug.repo_full_name }}@{{ bug.commit_hash.slice(0, 12) }}</code>
      </div>
      <h1 class="bug-title">{{ firstLine(bug.commit_message) }}</h1>
      <div class="bug-meta">
        <span>
          <a :href="`https://github.com/${bug.repo_full_name}`" target="_blank" class="meta-link">
            {{ bug.repo_full_name }}
          </a>
        </span>
        <span>·</span>
        <span>{{ formatDate(bug.authored_date) }}</span>
        <span v-if="bug.issue_refs.length">·</span>
        <span v-for="ref in bug.issue_refs" :key="ref">
          <a :href="`https://github.com/${bug.repo_full_name}/issues/${ref}`" target="_blank" class="meta-link">#{{ ref }}</a>
        </span>
      </div>

      <!-- Category badges -->
      <div class="badge-row">
        <span class="badge" :class="transitionClass(bug.transition)">{{ bug.transition }}</span>
        <span v-if="allF2PTests.length" class="badge badge-orange-solid">FAIL_PASS</span>
        <span class="badge badge-blue">{{ bug.modified_files_count }} files</span>
        <span class="badge badge-green">+{{ bug.insertions }}</span>
        <span class="badge badge-red">−{{ bug.deletions }}</span>
        <span v-if="allF2PTests.length" class="badge badge-purple">
          {{ allF2PTests.length }} F2P tests
        </span>
        <span v-if="bug.counts.pass_to_pass_tests" class="badge badge-gray">
          {{ bug.counts.pass_to_pass_tests }} P2P tests
        </span>
        <a
          :href="`https://github.com/${bug.repo_full_name}/commit/${bug.commit_hash}`"
          target="_blank"
          class="badge badge-outline"
        >View on GitHub ↗</a>
      </div>
    </div>

    <!-- Diff format toggle -->
    <div class="diff-controls">
      <span class="diff-controls-label">Diff format:</span>
      <div class="format-toggle">
        <button :class="{ active: fmt === 'side-by-side' }" @click="fmt = 'side-by-side'">Side-by-side</button>
        <button :class="{ active: fmt === 'line-by-line' }" @click="fmt = 'line-by-line'">Line-by-line</button>
      </div>
    </div>

    <!-- Commit message body -->
    <section class="info-section">
      <div class="info-header" @click="toggleSection('msg')">
        <div class="diff-section-title">
          <span class="section-icon">💬</span>
          Full commit message
        </div>
        <span class="toggle-icon">{{ open.msg ? '▲' : '▼' }}</span>
      </div>
      <div v-show="open.msg" class="commit-body">
        <pre>{{ bug.commit_message }}</pre>
      </div>
    </section>

    <!-- Human Patch -->
    <section v-if="bug.bug_patch" class="diff-section">
      <div class="diff-section-header" @click="toggleSection('bug')">
        <div class="diff-section-title">
          <span class="section-icon">🩹</span>
          Human Patch
          <span class="files-changed">Files changed ({{ bugPatchFileCount }})</span>
        </div>
        <span class="toggle-icon">{{ open.bug ? '▲' : '▼' }}</span>
      </div>
      <div v-show="open.bug" class="diff-body">
        <DiffViewer :patch="bug.bug_patch" :outputFormat="fmt" :key="'bug-' + fmt" />
      </div>
    </section>

    <!-- Test Patch -->
    <section v-if="bug.test_patch" class="diff-section">
      <div class="diff-section-header" @click="toggleSection('test')">
        <div class="diff-section-title">
          <span class="section-icon">🧪</span>
          Test Patch
          <span class="files-changed">Files changed ({{ testPatchFileCount }})</span>
        </div>
        <span class="toggle-icon">{{ open.test ? '▲' : '▼' }}</span>
      </div>
      <div v-show="open.test" class="diff-body">
        <DiffViewer :patch="bug.test_patch" outputFormat="line-by-line" :key="'test-' + fmt" />
      </div>
    </section>

    <!-- Non-code Patch -->
    <section v-if="bug.non_code_patch" class="diff-section">
      <div class="diff-section-header" @click="toggleSection('noncode')">
        <div class="diff-section-title">
          <span class="section-icon">📄</span>
          Resource Files
          <span class="files-changed">Non-code changes</span>
        </div>
        <span class="toggle-icon">{{ open.noncode ? '▲' : '▼' }}</span>
      </div>
      <div v-show="open.noncode" class="diff-body">
        <DiffViewer :patch="bug.non_code_patch" outputFormat="line-by-line" :key="'noncode-' + fmt" />
      </div>
    </section>

    <!-- Fail-to-pass tests -->
    <section v-if="allF2PTests.length" class="info-section">
      <div class="info-header" @click="toggleSection('f2p')">
        <div class="diff-section-title">
          <span class="section-icon">❌→✅</span>
          Fail-to-pass tests
          <span class="files-changed">{{ allF2PTests.length }} tests</span>
        </div>
        <span class="toggle-icon">{{ open.f2p ? '▲' : '▼' }}</span>
      </div>
      <div v-show="open.f2p" class="test-list">
        <div v-for="t in allF2PTests" :key="t" class="test-item test-f2p">{{ t }}</div>
      </div>
    </section>

    <!-- Pass-to-pass tests -->
    <section v-if="bug.pass_to_pass_tests.length" class="info-section">
      <div class="info-header" @click="toggleSection('p2p')">
        <div class="diff-section-title">
          <span class="section-icon">✅</span>
          Pass-to-pass tests
          <span class="files-changed">{{ bug.counts.pass_to_pass_tests }} total (showing {{ bug.pass_to_pass_tests.length }})</span>
        </div>
        <span class="toggle-icon">{{ open.p2p ? '▲' : '▼' }}</span>
      </div>
      <div v-show="open.p2p" class="test-list">
        <div v-for="t in bug.pass_to_pass_tests" :key="t" class="test-item test-p2p">{{ t }}</div>
        <div v-if="bug.counts.pass_to_pass_tests > bug.pass_to_pass_tests.length" class="test-item test-more">
          … and {{ bug.counts.pass_to_pass_tests - bug.pass_to_pass_tests.length }} more
        </div>
      </div>
    </section>

  </div>

  <div v-else class="loading">
    <span v-if="notFound">Bug not found. <RouterLink to="/dataset">← Back</RouterLink></span>
    <span v-else>Loading…</span>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'
import { useRoute } from 'vue-router'
import DiffViewer from '../components/DiffViewer.vue'

const route = useRoute()
const allBugs = ref([])
const bug = ref(null)
const notFound = ref(false)
const fmt = ref('side-by-side')
const open = ref({ bug: true, test: true, noncode: false, f2p: true, p2p: false, msg: false })

function toggleSection(key) {
  open.value[key] = !open.value[key]
}

onMounted(async () => {
  const res = await fetch('/bugs.json')
  allBugs.value = await res.json()
  loadBug()
})

watch(() => route.params.bid, loadBug)

function loadBug() {
  const found = allBugs.value.find(b => b.bid === route.params.bid)
  if (found) {
    bug.value = found
    notFound.value = false
    open.value = { bug: true, test: true, noncode: false, f2p: true, p2p: false, msg: false }
  } else if (allBugs.value.length) {
    notFound.value = true
  }
}

const prevBug = computed(() => {
  if (!bug.value) return null
  const idx = allBugs.value.findIndex(b => b.bid === bug.value.bid)
  return idx > 0 ? allBugs.value[idx - 1] : null
})
const nextBug = computed(() => {
  if (!bug.value) return null
  const idx = allBugs.value.findIndex(b => b.bid === bug.value.bid)
  return idx < allBugs.value.length - 1 ? allBugs.value[idx + 1] : null
})

const allF2PTests = computed(() => {
  if (!bug.value) return []
  const a = bug.value.fail_to_pass_tests ?? []
  const b = bug.value.parent_test_patch_fail_to_pass_tests ?? []
  return [...new Set([...a, ...b])]
})

const bugPatchFileCount = computed(() =>
  bug.value ? countDiffFiles(bug.value.bug_patch) : 0
)
const testPatchFileCount = computed(() =>
  bug.value ? countDiffFiles(bug.value.test_patch) : 0
)

function countDiffFiles(patch) {
  if (!patch) return 0
  return (patch.match(/^diff --git/gm) || []).length
}

function firstLine(msg) {
  return (msg || '').split('\n')[0]
}

function formatDate(iso) {
  if (!iso) return ''
  return new Date(iso).toLocaleDateString('en-US', { year: 'numeric', month: 'short', day: 'numeric' })
}

function transitionClass(t) {
  if (t === 'PASS_PASS') return 'badge-green-solid'
  if (t === 'FAIL_PASS') return 'badge-orange-solid'
  return 'badge-gray-solid'
}
</script>

<style scoped>
.bug-page {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 24px 80px;
}
.loading {
  text-align: center;
  padding: 80px;
  color: var(--text2);
}

/* Nav */
.bug-nav {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px 0;
  border-bottom: 1px solid var(--border);
  margin-bottom: 28px;
}
.back-link { color: var(--text2); font-size: 0.875rem; font-weight: 500; }
.back-link:hover { color: var(--android-dark); }
.bug-nav-pager { display: flex; gap: 12px; }
.pager-link { color: var(--android-dark); font-size: 0.875rem; font-weight: 500; }
.pager-link:hover { text-decoration: underline; }

/* Header */
.bug-header { margin-bottom: 24px; }
.bug-id {
  margin-bottom: 10px;
}
.bug-id code {
  font-size: 0.82rem;
  background: var(--bg2);
  padding: 3px 8px;
  border-radius: 5px;
  color: var(--text2);
  border: 1px solid var(--border);
}
.bug-title {
  font-size: clamp(1.1rem, 2.5vw, 1.5rem);
  font-weight: 700;
  letter-spacing: -0.02em;
  line-height: 1.3;
  margin-bottom: 10px;
}
.bug-meta {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  align-items: center;
  font-size: 0.85rem;
  color: var(--text2);
  margin-bottom: 14px;
}
.meta-link { color: var(--android-dark); font-weight: 500; }
.meta-link:hover { text-decoration: underline; }

/* Badges */
.badge-row { display: flex; flex-wrap: wrap; gap: 6px; }
.badge {
  display: inline-block;
  padding: 3px 10px;
  border-radius: 100px;
  font-size: 0.75rem;
  font-weight: 700;
}
.badge-green-solid { background: #16a34a; color: #fff; }
.badge-orange-solid { background: #ea580c; color: #fff; }
.badge-gray-solid { background: #6b7280; color: #fff; }
.badge-blue { background: #dbeafe; color: #1d4ed8; }
.badge-green { background: #dcfce7; color: #15803d; }
.badge-red { background: #fee2e2; color: #dc2626; }
.badge-purple { background: #ede9fe; color: #7c3aed; }
.badge-gray { background: var(--bg3); color: var(--text2); }
.badge-outline {
  background: transparent;
  border: 1px solid var(--border);
  color: var(--text2);
}
.badge-outline:hover { border-color: var(--android); color: var(--android-dark); }

/* Diff controls */
.diff-controls {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 20px;
}
.diff-controls-label { font-size: 0.85rem; color: var(--text2); font-weight: 500; }
.format-toggle {
  display: flex;
  border: 1px solid var(--border);
  border-radius: var(--radius);
  overflow: hidden;
}
.format-toggle button {
  padding: 5px 12px;
  font-size: 0.8rem;
  background: #fff;
  border: none;
  color: var(--text2);
  transition: all .15s;
}
.format-toggle button.active { background: var(--android); color: #fff; font-weight: 600; }

/* Diff sections */
.diff-section, .info-section {
  border: 1px solid var(--border);
  border-radius: 10px;
  overflow: hidden;
  margin-bottom: 16px;
}
.diff-section-header, .info-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 16px;
  background: var(--bg2);
  cursor: pointer;
  user-select: none;
  border-bottom: 1px solid var(--border);
}
.diff-section-header:hover, .info-header:hover { background: var(--bg3); }
.diff-section-title {
  display: flex;
  align-items: center;
  gap: 8px;
  font-weight: 600;
  font-size: 0.9rem;
}
.section-icon { font-size: 1rem; }
.files-changed {
  font-size: 0.78rem;
  font-weight: 400;
  color: var(--text3);
  background: var(--bg3);
  padding: 1px 8px;
  border-radius: 100px;
}
.toggle-icon { font-size: 0.7rem; color: var(--text3); }
.diff-body { overflow-x: auto; }

/* Test lists */
.test-list { padding: 12px 16px; display: flex; flex-direction: column; gap: 4px; }
.test-item {
  font-size: 0.8rem;
  padding: 5px 10px;
  border-radius: 5px;
  font-family: 'JetBrains Mono', monospace;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
.test-f2p { background: #fef3c7; color: #92400e; }
.test-p2p { background: #dcfce7; color: #15803d; }
.test-more { color: var(--text3); font-style: italic; background: none; }

/* Commit message */
.commit-body { padding: 16px; }
.commit-body pre {
  font-size: 0.85rem;
  color: var(--text2);
  white-space: pre-wrap;
  word-break: break-word;
  line-height: 1.6;
}
</style>

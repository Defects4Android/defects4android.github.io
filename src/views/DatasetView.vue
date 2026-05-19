<template>
  <div class="page">
    <!-- Sidebar -->
    <aside class="sidebar">
      <div class="sidebar-section">
        <div class="sidebar-label">Search</div>
        <input
          v-model="search"
          class="search-input"
          placeholder="Repo or commit…"
          type="search"
        />
      </div>

      <div class="sidebar-section">
        <div class="sidebar-label">Edit Scope</div>
        <label v-for="s in ALL_EDIT_SCOPES" :key="s" class="checkbox-row">
          <input type="checkbox" v-model="selectedEditScopes" :value="s" />
          <span>{{ scopeLabel(s) }}</span>
          <span class="filter-count">{{ editScopeCount(s) }}</span>
        </label>
      </div>

      <div class="sidebar-section">
        <div class="sidebar-label">Language</div>
        <label v-for="lang in ALL_LANGUAGES" :key="lang" class="checkbox-row">
          <input type="checkbox" v-model="selectedLanguages" :value="lang" />
          <span>{{ lang }}</span>
          <span class="filter-count">{{ languageCount(lang) }}</span>
        </label>
      </div>

      <div class="sidebar-section">
        <div class="sidebar-label">Has F2P Tests</div>
        <label class="checkbox-row">
          <input type="checkbox" v-model="onlyF2P" />
          <span>Only fail-to-pass</span>
        </label>
      </div>

      <button class="reset-btn" @click="resetFilters">Clear filters</button>
    </aside>

    <!-- Main content -->
    <div class="main">
      <div class="main-header">
        <h1 class="main-title">
          Defects4Android is showing
          <span class="count-badge">{{ filtered.length }} bug-fix{{ filtered.length !== 1 ? 'es' : '' }}</span>
        </h1>
      </div>

      <!-- Table -->
      <div class="table-wrap">
        <table class="bug-table">
          <thead>
            <tr>
              <th @click="sortBy('bid')" class="sortable">Bug ID <SortIcon :col="'bid'" :sort="sort" /></th>
              <th @click="sortBy('repo_full_name')" class="sortable">Repository <SortIcon :col="'repo_full_name'" :sort="sort" /></th>
              <th @click="sortBy('edit_scope')" class="sortable">Edit Scope <SortIcon :col="'edit_scope'" :sort="sort" /></th>
              <th @click="sortBy('f2p')" class="sortable"># F2P <SortIcon :col="'f2p'" :sort="sort" /></th>
              <th @click="sortBy('modified_files_count')" class="sortable">Files <SortIcon :col="'modified_files_count'" :sort="sort" /></th>
              <th @click="sortBy('insertions')" class="sortable">+Lines <SortIcon :col="'insertions'" :sort="sort" /></th>
              <th @click="sortBy('deletions')" class="sortable">−Lines <SortIcon :col="'deletions'" :sort="sort" /></th>
              <th @click="sortBy('hunks')" class="sortable">Hunks <SortIcon :col="'hunks'" :sort="sort" /></th>
            </tr>
          </thead>
          <tbody>
            <tr
              v-for="bug in sorted"
              :key="bug.bid"
              class="bug-row"
              @click="go(bug.bid)"
            >
              <td class="td-bid">
                <code>{{ bug.repo_full_name }}@{{ bug.commit_hash.slice(0, 12) }}</code>
              </td>
              <td class="td-repo">
                <a :href="`https://github.com/${bug.repo_full_name}`" target="_blank" @click.stop>
                  {{ bug.repo_full_name }}
                </a>
              </td>
              <td>
                <span v-if="bug.edit_scope" class="chip" :class="scopeChipClass(bug.edit_scope)">{{ scopeLabel(bug.edit_scope) }}</span>
                <span v-else class="na">—</span>
              </td>
              <td class="num">{{ totalF2P(bug) }}</td>
              <td class="num">{{ bug.modified_files_count }}</td>
              <td class="num added">+{{ bug.insertions }}</td>
              <td class="num removed">−{{ bug.deletions }}</td>
              <td class="num">{{ hunkCount(bug.bug_patch) }}</td>
            </tr>
            <tr v-if="sorted.length === 0">
              <td colspan="8" class="empty">No results match your filters.</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, defineComponent, h } from 'vue'
import { useRouter } from 'vue-router'

const SortIcon = defineComponent({
  props: ['col', 'sort'],
  render(props) {
    const active = props.sort?.col === props.col
    const asc = active && props.sort?.asc
    return h('span', { class: 'sort-icon' + (active ? ' active' : '') }, active ? (asc ? ' ↑' : ' ↓') : ' ↕')
  },
})

const ALL_EDIT_SCOPES = ['single_function', 'single_class', 'single_file', 'multi_file_same_component', 'multi_component']
const ALL_LANGUAGES = ['Java', 'Kotlin', 'Dart']

const router = useRouter()
const bugs = ref([])
const search = ref('')
const selectedEditScopes = ref([...ALL_EDIT_SCOPES])
const selectedLanguages = ref([...ALL_LANGUAGES])
const onlyF2P = ref(false)
const sort = ref({ col: 'bid', asc: true })

onMounted(async () => {
  const res = await fetch('/bugs.json')
  bugs.value = await res.json()
})

function totalF2P(bug) {
  const a = bug.fail_to_pass_tests ?? []
  const b = bug.parent_test_patch_fail_to_pass_tests ?? []
  return new Set([...a, ...b]).size
}

function editScopeCount(s) {
  return bugs.value.filter(b => b.edit_scope === s).length
}

function scopeLabel(s) {
  const labels = {
    single_function: 'Single Function',
    single_class: 'Single Class',
    single_file: 'Single File',
    multi_file_same_component: 'Multi-File Same Component',
    multi_component: 'Multi-Component',
  }
  return labels[s] ?? s
}

function scopeChipClass(s) {
  const classes = {
    single_function: 'chip-blue',
    single_class: 'chip-purple',
    single_file: 'chip-green',
    multi_file_same_component: 'chip-orange',
    multi_component: 'chip-red',
  }
  return classes[s] ?? 'chip-gray'
}

function hunkCount(patch) {
  if (!patch) return 0
  return (patch.match(/^@@/gm) || []).length
}

function languageCount(lang) {
  return bugs.value.filter(b => b.language === lang).length
}

const filtered = computed(() => {
  let list = bugs.value
  if (search.value.trim()) {
    const q = search.value.toLowerCase()
    list = list.filter(b =>
      b.repo_full_name.toLowerCase().includes(q) ||
      b.commit_hash.includes(q) ||
      b.commit_message.toLowerCase().includes(q)
    )
  }
  if (selectedEditScopes.value.length < ALL_EDIT_SCOPES.length) {
    list = list.filter(b => !b.edit_scope || selectedEditScopes.value.includes(b.edit_scope))
  }
  if (selectedLanguages.value.length < ALL_LANGUAGES.length) {
    list = list.filter(b =>
      !b.language || selectedLanguages.value.includes(b.language)
    )
  }
  if (onlyF2P.value) {
    list = list.filter(b => totalF2P(b) > 0)
  }
  return list
})

const sorted = computed(() => {
  const list = [...filtered.value]
  const { col, asc } = sort.value
  list.sort((a, b) => {
    let av = col === 'f2p' ? totalF2P(a)
           : col === 'hunks' ? hunkCount(a.bug_patch)
           : a[col]
    let bv = col === 'f2p' ? totalF2P(b)
           : col === 'hunks' ? hunkCount(b.bug_patch)
           : b[col]
    if (typeof av === 'string') av = av.toLowerCase()
    if (typeof bv === 'string') bv = bv.toLowerCase()
    if (av < bv) return asc ? -1 : 1
    if (av > bv) return asc ? 1 : -1
    return 0
  })
  return list
})

function sortBy(col) {
  if (sort.value.col === col) {
    sort.value = { col, asc: !sort.value.asc }
  } else {
    sort.value = { col, asc: true }
  }
}

function go(bid) {
  router.push(`/dataset/${bid}`)
}

function resetFilters() {
  search.value = ''
  selectedEditScopes.value = [...ALL_EDIT_SCOPES]
  selectedLanguages.value = [...ALL_LANGUAGES]
  onlyF2P.value = false
}
</script>

<style scoped>
.page {
  display: flex;
  min-height: calc(100vh - 60px);
}

/* Sidebar */
.sidebar {
  width: 220px;
  flex-shrink: 0;
  border-right: 1px solid var(--border);
  padding: 24px 16px;
  background: var(--bg2);
  display: flex;
  flex-direction: column;
  gap: 28px;
}
.sidebar-section { display: flex; flex-direction: column; gap: 10px; }
.sidebar-label {
  font-size: 0.72rem;
  font-weight: 700;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: var(--text3);
}
.search-input {
  width: 100%;
  padding: 7px 10px;
  border: 1px solid var(--border);
  border-radius: var(--radius);
  font-size: 0.85rem;
  background: #fff;
  color: var(--text);
  outline: none;
  transition: border-color .15s;
}
.search-input:focus { border-color: var(--android); }
.checkbox-row {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 0.85rem;
  cursor: pointer;
  color: var(--text2);
}
.checkbox-row input { accent-color: var(--android); }
.filter-count {
  margin-left: auto;
  font-size: 0.72rem;
  color: var(--text3);
  background: var(--bg3);
  padding: 1px 6px;
  border-radius: 100px;
  font-variant-numeric: tabular-nums;
}
.reset-btn {
  margin-top: auto;
  padding: 7px 12px;
  border-radius: var(--radius);
  border: 1px solid var(--border);
  background: #fff;
  font-size: 0.82rem;
  color: var(--text2);
  transition: all .15s;
}
.reset-btn:hover { background: var(--bg3); color: var(--text); }

/* Main */
.main {
  flex: 1;
  min-width: 0;
  padding: 28px 24px;
  overflow-x: auto;
}
.main-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  flex-wrap: wrap;
  gap: 12px;
  margin-bottom: 20px;
}
.main-title {
  font-size: 1.15rem;
  font-weight: 700;
  color: var(--text);
}
.count-badge {
  display: inline-block;
  padding: 2px 10px;
  border-radius: 100px;
  background: var(--android);
  color: #fff;
  font-size: 0.9rem;
  font-weight: 700;
}
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

/* Table */
.table-wrap {
  overflow-x: auto;
  border: 1px solid var(--border);
  border-radius: 10px;
  box-shadow: var(--shadow);
}
.bug-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.875rem;
}
.bug-table thead tr {
  background: var(--bg2);
  border-bottom: 1px solid var(--border);
}
.bug-table th {
  padding: 10px 14px;
  text-align: left;
  font-size: 0.78rem;
  font-weight: 700;
  color: var(--text2);
  white-space: nowrap;
}
.bug-table th.sortable { cursor: pointer; user-select: none; }
.bug-table th.sortable:hover { color: var(--text); }
.sort-icon { opacity: .4; font-size: 0.7rem; }
.sort-icon.active { opacity: 1; color: var(--android-dark); }

.bug-row {
  border-bottom: 1px solid var(--bg2);
  cursor: pointer;
  transition: background .12s;
}
.bug-row:hover { background: #f0fdf6; }
.bug-row:last-child { border-bottom: none; }

.bug-table td { padding: 10px 14px; vertical-align: middle; }
.td-bid code {
  font-size: 0.8rem;
  background: var(--bg2);
  padding: 2px 6px;
  border-radius: 4px;
  color: #1d4ed8;
}
.td-repo a { color: var(--android-dark); font-weight: 500; }
.td-repo a:hover { text-decoration: underline; }

.num { text-align: right; font-variant-numeric: tabular-nums; color: var(--text2); }
.added { color: #16a34a; font-weight: 600; }
.removed { color: #dc2626; font-weight: 600; }

.empty { text-align: center; color: var(--text3); padding: 40px; }

/* Chips */
.chip {
  display: inline-block;
  padding: 2px 9px;
  border-radius: 100px;
  font-size: 0.75rem;
  font-weight: 700;
  letter-spacing: 0.02em;
}
.chip-green { background: #dcfce7; color: #15803d; }
.chip-orange { background: #ffedd5; color: #c2410c; }
.chip-blue { background: #dbeafe; color: #1d4ed8; }
.chip-purple { background: #ede9fe; color: #7c3aed; }
.chip-red { background: #fee2e2; color: #dc2626; }
.chip-gray { background: var(--bg3); color: var(--text2); }
.na { color: var(--text3); }
</style>

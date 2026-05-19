<template>
  <main>

    <!-- HERO -->
    <section class="hero">
      <div class="hero-inner">
        <div class="hero-badge">
          <span class="badge-dot"></span>
          Android Bug-Fix Research Dataset
        </div>
        <h1>
          <span class="brand">Defects4Android</span>
        </h1>
        <p class="hero-sub">
          A curated benchmark of real-world bug fixes from open-source Android applications,
          with reproducible test-suite transitions for automated program repair research.
        </p>
        <div class="hero-ctas">
          <RouterLink to="/dataset" class="btn btn-primary">Browse Dataset</RouterLink>
          <a href="https://huggingface.co/Defects4Android" target="_blank" class="btn btn-outline">
            <HFIcon /> HuggingFace
          </a>
          <a href="https://arxiv.org" target="_blank" class="btn btn-outline">
            <PaperIcon /> Paper
          </a>
          <a href="https://github.com/Defects4Android" target="_blank" class="btn btn-ghost">
            <GitHubIcon /> GitHub
          </a>
        </div>

        <!-- stats row -->
        <div class="stats-row">
          <div v-for="s in stats" :key="s.label" class="stat-card">
            <span class="stat-num">{{ s.value }}</span>
            <span class="stat-label">{{ s.label }}</span>
          </div>
        </div>
      </div>

      <!-- decorative grid bg -->
      <div class="hero-grid" aria-hidden="true"></div>
    </section>

    <!-- WHAT IS IT -->
    <section class="section alt">
      <div class="container">
        <SectionLabel>About</SectionLabel>
        <h2>Built for Android APR research</h2>
        <p class="section-sub">
          Defects4Android is a systematic collection of bug-fixing commits from popular
          open-source Android applications. Each entry is validated with a Docker harness
          that confirms the test-suite transitions from failing to passing, making the
          dataset directly usable for benchmarking automated program repair tools.
        </p>

        <div class="feature-grid">
          <div v-for="f in features" :key="f.title" class="feature-card">
            <div class="feature-icon">{{ f.icon }}</div>
            <h3>{{ f.title }}</h3>
            <p>{{ f.desc }}</p>
          </div>
        </div>
      </div>
    </section>

    <!-- SCHEMA -->
    <section class="section">
      <div class="container">
        <SectionLabel>Schema</SectionLabel>
        <h2>Dataset fields at a glance</h2>
        <p class="section-sub">
          Each instance captures the full context of a bug fix — patches, tests,
          and harness-validated outcomes.
        </p>

        <div class="schema-grid">
          <div v-for="group in schema" :key="group.group" class="schema-group">
            <div class="schema-group-title">{{ group.group }}</div>
            <div v-for="field in group.fields" :key="field.name" class="schema-row">
              <code class="field-name">{{ field.name }}</code>
              <span class="field-type">{{ field.type }}</span>
              <span class="field-desc">{{ field.desc }}</span>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- QUICK START -->
    <section class="section alt">
      <div class="container narrow">
        <SectionLabel>Quick Start</SectionLabel>
        <h2>Load it in three lines</h2>
        <p class="section-sub">Available on HuggingFace Datasets — no download required.</p>

        <div class="code-block">
          <div class="code-header">
            <span class="code-lang">Python</span>
            <button class="copy-btn" @click="copy(pythonSnippet)" :class="{ copied: justCopied }">
              {{ justCopied ? '✓ Copied' : 'Copy' }}
            </button>
          </div>
          <pre><code>{{ pythonSnippet }}</code></pre>
        </div>
      </div>
    </section>

    <!-- CITATION -->
    <section class="section">
      <div class="container narrow">
        <SectionLabel>Citation</SectionLabel>
        <h2>Cite our work</h2>
        <div class="code-block">
          <div class="code-header">
            <span class="code-lang">BibTeX</span>
            <button class="copy-btn" @click="copy(bibtex, true)" :class="{ copied: justCopiedBib }">
              {{ justCopiedBib ? '✓ Copied' : 'Copy' }}
            </button>
          </div>
          <pre><code>{{ bibtex }}</code></pre>
        </div>
      </div>
    </section>

    <!-- FOOTER -->
    <footer class="footer">
      <div class="container">
        <div class="footer-inner">
          <span class="footer-logo"><img src="/favicon.svg" style="width:18px;height:18px;vertical-align:middle;margin-right:6px;" alt="" />Defects4Android</span>
          <span class="footer-copy">
            Released under CC-BY-4.0 ·
            <a href="https://github.com/Defects4Android" target="_blank">GitHub</a> ·
            <a href="https://huggingface.co/Defects4Android" target="_blank">HuggingFace</a> ·
            <a href="https://github.com/orgs/Defects4Android/packages" target="_blank">Docker Images</a>
          </span>
        </div>
      </div>
    </footer>

  </main>
</template>

<script setup>
import { ref, defineComponent, h } from 'vue'

// Inline micro-components
const SectionLabel = defineComponent({ render: () => h('div', { class: 'section-label' }, []) })
const SectionLabel2 = { template: '<div class="section-label"><slot/></div>' }

const HFIcon = { template: '<svg width="16" height="16" viewBox="0 0 24 24" fill="currentColor"><circle cx="12" cy="12" r="10"/></svg>' }
const PaperIcon = { template: '<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M14 2H6a2 2 0 00-2 2v16a2 2 0 002 2h12a2 2 0 002-2V8z"/><polyline points="14 2 14 8 20 8"/></svg>' }
const GitHubIcon = { template: '<svg width="16" height="16" viewBox="0 0 24 24" fill="currentColor"><path d="M12 2C6.477 2 2 6.477 2 12c0 4.418 2.865 8.166 6.839 9.489.5.092.682-.217.682-.482 0-.237-.009-.868-.013-1.703-2.782.604-3.369-1.342-3.369-1.342-.454-1.154-1.11-1.462-1.11-1.462-.908-.62.069-.608.069-.608 1.003.07 1.531 1.03 1.531 1.03.892 1.529 2.341 1.087 2.91.832.092-.647.35-1.088.636-1.338-2.22-.253-4.555-1.11-4.555-4.943 0-1.091.39-1.984 1.029-2.683-.103-.253-.446-1.27.098-2.647 0 0 .84-.269 2.75 1.025A9.578 9.578 0 0112 6.836a9.59 9.59 0 012.504.337c1.909-1.294 2.747-1.025 2.747-1.025.546 1.377.202 2.394.1 2.647.64.699 1.028 1.592 1.028 2.683 0 3.842-2.339 4.687-4.566 4.935.359.309.678.919.678 1.852 0 1.336-.012 2.415-.012 2.741 0 .267.18.578.688.48C19.138 20.163 22 16.418 22 12c0-5.523-4.477-10-10-10z"/></svg>' }

const stats = [
  { value: '5+', label: 'Bug Fixes' },
  { value: '10+', label: 'Android Apps' },
  { value: '3', label: 'Patch Types' },
  { value: '100%', label: 'Harness Validated' },
]

const features = [
  {
    icon: '🔬',
    title: 'Harness Validated',
    desc: 'Every bug is confirmed by a Docker-based test harness that runs the test suite before and after the fix, ensuring reproducible pass/fail transitions.',
  },
  {
    icon: '🩹',
    title: 'Three Patch Types',
    desc: 'Each instance includes the human bug patch, test patch, and non-code resource patch — giving full context for repair and analysis.',
  },
  {
    icon: '📱',
    title: 'Android-Focused',
    desc: 'Sourced from popular open-source Android apps written in Kotlin, Java, and Flutter/Dart, covering UI, networking, database, and system-level bugs.',
  },
  {
    icon: '🔗',
    title: 'Full Provenance',
    desc: 'Every entry links back to the original GitHub commit with parent and child hashes, issue refs, and author metadata.',
  },
  {
    icon: '📊',
    title: 'Rich Metadata',
    desc: 'Fail-to-pass and pass-to-pass test counts, file-level change stats, matched bug keywords, and transition type classification.',
  },
  {
    icon: '🤗',
    title: 'HuggingFace Ready',
    desc: 'Published on HuggingFace Datasets — load with a single line, no data wrangling required.',
  },
]

const schema = [
  {
    group: 'Identity',
    fields: [
      { name: 'bid', type: 'string', desc: 'Unique bug identifier (repo + commit)' },
      { name: 'repo_full_name', type: 'string', desc: 'GitHub owner/repo slug' },
      { name: 'commit_hash', type: 'string', desc: 'SHA-1 of the fixing commit' },
      { name: 'parent_hash', type: 'string', desc: 'SHA-1 of the buggy parent commit' },
    ],
  },
  {
    group: 'Patches',
    fields: [
      { name: 'bug_patch', type: 'string', desc: 'Unified diff of source-code changes' },
      { name: 'test_patch', type: 'string', desc: 'Unified diff of test-file changes' },
      { name: 'non_code_patch', type: 'string', desc: 'Unified diff of resource/config changes' },
    ],
  },
  {
    group: 'Harness',
    fields: [
      { name: 'transition', type: 'string', desc: 'PASS_PASS | FAIL_PASS | UNKNOWN' },
      { name: 'fail_to_pass_tests', type: 'string[]', desc: 'Tests that go from fail → pass' },
      { name: 'pass_to_pass_tests', type: 'string[]', desc: 'Tests that remain passing' },
    ],
  },
  {
    group: 'Metadata',
    fields: [
      { name: 'author_name', type: 'string', desc: 'Commit author' },
      { name: 'authored_date', type: 'string', desc: 'ISO 8601 authoring timestamp' },
      { name: 'insertions', type: 'int', desc: 'Lines added in bug patch' },
      { name: 'deletions', type: 'int', desc: 'Lines removed in bug patch' },
      { name: 'issue_refs', type: 'string[]', desc: 'Referenced GitHub issue numbers' },
    ],
  },
]

const pythonSnippet = `from datasets import load_dataset

ds = load_dataset("Defects4Android/defects4android")
print(ds["test"][0]["bug_patch"])  # unified diff`

const bibtex = `@misc{fakorede2026defects4android,
  title        = {Defects4Android: A Reproducible Benchmark for Android Fault Localization and Repair},
  author       = {Fakorede, Moshood A. and Upadhyay, Krishna and Farooq, Umar},
  year         = {2026},
  institution  = {Department of Computer Science, Louisiana State University},
  address      = {USA},
  howpublished = {\\url{https://github.com/Defects4Android/defects4android}},
  note         = {Benchmark dataset and reproducibility harness for Android fault localization and repair}
}`

const justCopied = ref(false)
const justCopiedBib = ref(false)

function copy(text, bib = false) {
  navigator.clipboard.writeText(text)
  if (bib) {
    justCopiedBib.value = true
    setTimeout(() => (justCopiedBib.value = false), 2000)
  } else {
    justCopied.value = true
    setTimeout(() => (justCopied.value = false), 2000)
  }
}
</script>

<style scoped>
/* ─── HERO ─── */
.hero {
  position: relative;
  overflow: hidden;
  padding: 100px 24px 80px;
  text-align: center;
  background: linear-gradient(180deg, #f0fdf6 0%, #ffffff 100%);
}
.hero-grid {
  position: absolute;
  inset: 0;
  background-image:
    linear-gradient(var(--border) 1px, transparent 1px),
    linear-gradient(90deg, var(--border) 1px, transparent 1px);
  background-size: 40px 40px;
  opacity: 0.45;
  pointer-events: none;
  mask-image: radial-gradient(ellipse 80% 80% at 50% 0%, black 40%, transparent 100%);
}
.hero-inner {
  position: relative;
  max-width: 760px;
  margin: 0 auto;
}
.hero-badge {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 5px 14px;
  border-radius: 100px;
  border: 1px solid #bbf7d0;
  background: #f0fdf4;
  color: #15803d;
  font-size: 0.8rem;
  font-weight: 600;
  letter-spacing: 0.02em;
  margin-bottom: 24px;
}
.badge-dot {
  width: 7px; height: 7px;
  border-radius: 50%;
  background: var(--android);
  flex-shrink: 0;
}
h1 {
  font-size: clamp(2.6rem, 6vw, 4rem);
  font-weight: 800;
  letter-spacing: -0.04em;
  line-height: 1.05;
  margin-bottom: 20px;
}
.brand {
  background: linear-gradient(135deg, #00875a 0%, #3DDC84 60%, #00c853 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}
.hero-sub {
  font-size: 1.1rem;
  color: var(--text2);
  max-width: 580px;
  margin: 0 auto 36px;
  line-height: 1.7;
}
.hero-ctas {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  justify-content: center;
  margin-bottom: 56px;
}

/* Buttons */
.btn {
  display: inline-flex;
  align-items: center;
  gap: 7px;
  padding: 10px 20px;
  border-radius: var(--radius);
  font-size: 0.9rem;
  font-weight: 600;
  transition: all .15s;
  border: 1px solid transparent;
}
.btn-primary { background: var(--android); color: #fff; }
.btn-primary:hover { background: var(--android-dark); }
.btn-outline {
  background: #fff;
  color: var(--text);
  border-color: var(--border);
  box-shadow: var(--shadow);
}
.btn-outline:hover { border-color: #9ca3af; background: var(--bg2); }
.btn-ghost { color: var(--text2); }
.btn-ghost:hover { color: var(--text); background: var(--bg2); }

/* Stats */
.stats-row {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  justify-content: center;
}
.stat-card {
  flex: 1;
  min-width: 110px;
  max-width: 160px;
  padding: 18px 12px;
  border-radius: 12px;
  background: #fff;
  border: 1px solid var(--border);
  box-shadow: var(--shadow);
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
}
.stat-num {
  font-size: 1.8rem;
  font-weight: 800;
  color: var(--android-dark);
  letter-spacing: -0.03em;
}
.stat-label {
  font-size: 0.78rem;
  color: var(--text2);
  font-weight: 500;
  text-align: center;
}

/* ─── SECTIONS ─── */
.section { padding: 80px 24px; }
.section.alt { background: var(--bg2); }
.container { max-width: 1100px; margin: 0 auto; }
.container.narrow { max-width: 760px; margin: 0 auto; }
.section-label {
  display: inline-block;
  font-size: 0.75rem;
  font-weight: 700;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: var(--android-dark);
  margin-bottom: 12px;
}
h2 {
  font-size: clamp(1.6rem, 3vw, 2.2rem);
  font-weight: 800;
  letter-spacing: -0.03em;
  margin-bottom: 12px;
}
.section-sub {
  color: var(--text2);
  font-size: 1rem;
  line-height: 1.7;
  max-width: 620px;
  margin-bottom: 48px;
}

/* Feature grid */
.feature-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 16px;
}
.feature-card {
  padding: 24px;
  border-radius: 12px;
  background: #fff;
  border: 1px solid var(--border);
  box-shadow: var(--shadow);
}
.feature-icon { font-size: 1.8rem; margin-bottom: 12px; }
.feature-card h3 { font-size: 1rem; font-weight: 700; margin-bottom: 8px; }
.feature-card p { font-size: 0.875rem; color: var(--text2); line-height: 1.6; }

/* Schema */
.schema-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(340px, 1fr));
  gap: 16px;
}
.schema-group {
  border: 1px solid var(--border);
  border-radius: 12px;
  overflow: hidden;
  background: #fff;
}
.schema-group-title {
  padding: 10px 16px;
  background: var(--bg2);
  font-size: 0.78rem;
  font-weight: 700;
  letter-spacing: 0.06em;
  text-transform: uppercase;
  color: var(--text2);
  border-bottom: 1px solid var(--border);
}
.schema-row {
  display: grid;
  grid-template-columns: 140px 60px 1fr;
  gap: 8px;
  align-items: start;
  padding: 8px 16px;
  font-size: 0.82rem;
  border-bottom: 1px solid var(--bg2);
}
.schema-row:last-child { border-bottom: none; }
.field-name { color: #1d4ed8; font-weight: 600; white-space: nowrap; }
.field-type { color: #7c3aed; font-size: 0.75rem; padding-top: 1px; }
.field-desc { color: var(--text2); line-height: 1.4; }

/* Code block */
.code-block {
  border-radius: 12px;
  border: 1px solid var(--border);
  overflow: hidden;
  background: #0f1117;
  box-shadow: var(--shadow-md);
}
.code-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 16px;
  background: #1a1d27;
  border-bottom: 1px solid #2d3142;
}
.code-lang { font-size: 0.78rem; font-weight: 600; color: #6b7280; text-transform: uppercase; letter-spacing: 0.08em; }
.copy-btn {
  font-size: 0.78rem;
  color: #9ca3af;
  background: none;
  border: 1px solid #374151;
  border-radius: 6px;
  padding: 3px 10px;
  transition: all .15s;
}
.copy-btn:hover { color: #fff; border-color: #6b7280; }
.copy-btn.copied { color: var(--android); border-color: var(--android); }
pre { padding: 20px; overflow-x: auto; }
pre code { color: #e2e8f0; font-size: 0.85rem; line-height: 1.7; }

/* Footer */
.footer {
  border-top: 1px solid var(--border);
  padding: 28px 24px;
}
.footer-inner {
  max-width: 1100px;
  margin: 0 auto;
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 12px;
}
.footer-logo { font-weight: 700; font-size: 0.95rem; }
.footer-copy { font-size: 0.82rem; color: var(--text2); }
.footer-copy a { color: var(--android-dark); text-decoration: underline; }
</style>

<template>
  <nav class="nav">
    <div class="nav-inner">
      <RouterLink to="/" class="logo">
        <img src="/favicon.svg" class="logo-icon" alt="" />
        <span class="logo-text">Defects4Android</span>
      </RouterLink>
      <div class="nav-links">
        <RouterLink to="/" class="nav-link" active-class="active" exact>Home</RouterLink>
        <RouterLink to="/dataset" class="nav-link" active-class="active">Dataset</RouterLink>
        <a href="https://arxiv.org" target="_blank" class="nav-link">Paper</a>
        <a href="https://github.com/orgs/Defects4Android/packages" target="_blank" class="nav-link">Docker Images</a>
        <a href="https://huggingface.co/Defects4Android" target="_blank" class="nav-btn">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="currentColor"><path d="M12 2C6.477 2 2 6.477 2 12s4.477 10 10 10 10-4.477 10-10S17.523 2 12 2z"/></svg>
          HuggingFace
        </a>
      </div>
    </div>
  </nav>
</template>

<style scoped>
.nav {
  position: sticky;
  top: 0;
  z-index: 100;
  background: rgba(255,255,255,0.95);
  backdrop-filter: blur(10px);
  border-bottom: 1px solid var(--border);
}
.nav-inner {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 24px;
  height: 60px;
  display: flex;
  align-items: center;
  justify-content: space-between;
}
.logo {
  display: flex;
  align-items: center;
  gap: 8px;
  font-weight: 700;
  font-size: 1.05rem;
  color: var(--text);
}
.logo-icon { width: 26px; height: 26px; display: block; }
.logo-text { letter-spacing: -0.02em; }
.nav-links {
  display: flex;
  align-items: center;
  gap: 4px;
}
.nav-link {
  padding: 6px 12px;
  border-radius: var(--radius);
  color: var(--text2);
  font-size: 0.875rem;
  font-weight: 500;
  transition: color .15s, background .15s;
}
.nav-link:hover, .nav-link.active { color: var(--text); background: var(--bg2); }
.nav-btn {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 6px 14px;
  margin-left: 8px;
  border-radius: var(--radius);
  background: var(--android);
  color: #fff;
  font-size: 0.875rem;
  font-weight: 600;
  transition: background .15s;
}
.nav-btn:hover { background: var(--android-dark); }
</style>

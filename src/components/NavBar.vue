<template>
  <nav class="nav">
    <div class="nav-inner">
      <RouterLink to="/" class="logo">
        <img src="/favicon.svg" class="logo-icon" alt="" />
        <span class="logo-text">Defects4Android</span>
      </RouterLink>

      <!-- Hamburger button (mobile only) -->
      <button class="hamburger" @click="open = !open" :aria-expanded="open" aria-label="Toggle menu">
        <span class="bar" :class="{ rotated: open }"></span>
        <span class="bar" :class="{ hidden: open }"></span>
        <span class="bar" :class="{ rotatedNeg: open }"></span>
      </button>

      <!-- Links -->
      <div class="nav-links" :class="{ 'nav-links--open': open }">
        <RouterLink to="/" class="nav-link" active-class="active" exact @click="open = false">Home</RouterLink>
        <RouterLink to="/dataset" class="nav-link" active-class="active" @click="open = false">Dataset</RouterLink>
        <span class="nav-link nav-link--disabled">Paper <span class="soon">soon</span></span>
        <a href="https://github.com/orgs/Defects4Android/packages" target="_blank" class="nav-link" @click="open = false">Docker Images</a>
        <a href="https://huggingface.co/Defects4Android" target="_blank" class="nav-btn" @click="open = false">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="currentColor"><path d="M12 2C6.477 2 2 6.477 2 12s4.477 10 10 10 10-4.477 10-10S17.523 2 12 2z"/></svg>
          HuggingFace
        </a>
      </div>
    </div>
  </nav>
</template>

<script setup>
import { ref } from 'vue'
const open = ref(false)
</script>

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
  flex-wrap: wrap;
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

/* Desktop nav */
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
.nav-link--disabled { opacity: 0.45; cursor: default; pointer-events: none; }
.soon {
  font-size: 0.65rem;
  font-weight: 700;
  letter-spacing: 0.04em;
  text-transform: uppercase;
  background: var(--bg3);
  color: var(--text3);
  padding: 1px 5px;
  border-radius: 4px;
  margin-left: 3px;
  vertical-align: middle;
}
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

/* Hamburger */
.hamburger {
  display: none;
  flex-direction: column;
  justify-content: center;
  gap: 5px;
  width: 36px;
  height: 36px;
  background: none;
  border: none;
  padding: 4px;
  cursor: pointer;
  border-radius: var(--radius);
}
.hamburger:hover { background: var(--bg2); }
.bar {
  display: block;
  width: 100%;
  height: 2px;
  background: var(--text);
  border-radius: 2px;
  transition: transform .2s, opacity .2s;
  transform-origin: center;
}
.bar.rotated    { transform: translateY(7px) rotate(45deg); }
.bar.rotatedNeg { transform: translateY(-7px) rotate(-45deg); }
.bar.hidden     { opacity: 0; }

/* Mobile */
@media (max-width: 640px) {
  .hamburger { display: flex; }
  .nav-inner { height: auto; padding: 0 16px; }

  .nav-links {
    display: none;
    width: 100%;
    flex-direction: column;
    align-items: stretch;
    padding: 8px 0 12px;
    gap: 2px;
    border-top: 1px solid var(--border);
  }
  .nav-links--open { display: flex; }

  .nav-link {
    padding: 10px 12px;
    font-size: 0.95rem;
  }
  .nav-btn {
    margin-left: 0;
    margin-top: 6px;
    justify-content: center;
    padding: 10px 14px;
    font-size: 0.95rem;
  }
}
</style>

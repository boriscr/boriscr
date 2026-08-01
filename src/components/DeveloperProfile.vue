<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import SplineCharacter from './SplineCharacter.vue'

const name = 'Boris'
const username = 'kredensir'
const title = 'Full Stack Developer'
const quote = 'El código es como el humor. Cuando tienes que explicarlo, es malo.'
const quoteAuthor = 'Cory House'
const about = [
  'Aprendiendo constantemente sobre Arquitectura de Software y Optimización',
  'Resolviendo problemas complejos con soluciones simples.'
]
const skills = [
  { name: 'HTML', icon: 'html' },
  { name: 'CSS', icon: 'css' },
  { name: 'Bootstrap', icon: 'bootstrap' },
  { name: 'JavaScript', icon: 'js' },
  { name: 'PHP', icon: 'php' },
  { name: 'Laravel', icon: 'laravel' },
  { name: 'MySQL', icon: 'mysql' },
  { name: 'GitHub', icon: 'github' },
  { name: 'VS Code', icon: 'vscode' },
  { name: 'Git', icon: 'git' },
  { name: 'Flutter', icon: 'flutter' },
  { name: 'Vue', icon: 'vue' },
  { name: 'Python', icon: 'python' },
  { name: 'Firebase', icon: 'firebase' }
]
const socialLinks = [
  { name: 'LinkedIn', url: 'https://linkedin.com/in/kredensir', icon: 'linkedin' },
  { name: 'Facebook', url: 'https://facebook.com/robin.boris.92', icon: 'facebook' },
  { name: 'Instagram', url: 'https://instagram.com/boris_kredensir', icon: 'instagram' },
  { name: 'GitHub', url: 'https://github.com/kredensir', icon: 'github' }
]

const heroRef = ref(null)
const tiltX = ref(0)
const tiltY = ref(0)
const mouseX = ref(0)
const mouseY = ref(0)
const scrollY = ref(0)
const visibleSections = ref(new Set())
const isTouchDevice = ref(false)
const gyroBeta = ref(0)
const gyroGamma = ref(0)



function handleMouseMove(e) {
  mouseX.value = (e.clientX / window.innerWidth) * 2 - 1
  mouseY.value = (e.clientY / window.innerHeight) * 2 - 1
  if (!heroRef.value) return
  const rect = heroRef.value.getBoundingClientRect()
  const x = e.clientX - rect.left
  const y = e.clientY - rect.top
  const cx = rect.width / 2
  const cy = rect.height / 2
  tiltX.value = ((y - cy) / cy) * -12
  tiltY.value = ((x - cx) / cx) * 12
}

function handleMouseLeave() {
  tiltX.value = 0
  tiltY.value = 0
}

function handleTouchMove(e) {
  const touch = e.touches[0]
  mouseX.value = (touch.clientX / window.innerWidth) * 2 - 1
  mouseY.value = (touch.clientY / window.innerHeight) * 2 - 1
  if (!heroRef.value) return
  const rect = heroRef.value.getBoundingClientRect()
  const x = touch.clientX - rect.left
  const y = touch.clientY - rect.top
  const cx = rect.width / 2
  const cy = rect.height / 2
  tiltX.value = ((y - cy) / cy) * -10
  tiltY.value = ((x - cx) / cx) * 10
}

function handleTouchEnd() {
  tiltX.value = 0
  tiltY.value = 0
}

function handleScroll() {
  scrollY.value = window.scrollY
}

function handleOrientation(e) {
  if (!isTouchDevice.value) return
  gyroBeta.value = (e.beta || 0) // -180 to 180 (tilt front/back)
  gyroGamma.value = (e.gamma || 0) // -90 to 90 (tilt left/right)
  const betaNorm = Math.max(-1, Math.min(1, (gyroBeta.value - 45) / 45))
  const gammaNorm = Math.max(-1, Math.min(1, gyroGamma.value / 45))
  mouseX.value = gammaNorm
  mouseY.value = -betaNorm
  if (heroRef.value) {
    tiltX.value = betaNorm * 10
    tiltY.value = gammaNorm * 10
  }
}

let observer = null

onMounted(() => {
  isTouchDevice.value = 'ontouchstart' in window || navigator.maxTouchPoints > 0

  observer = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        visibleSections.value.add(entry.target.dataset.section)
      }
    })
  }, { threshold: 0.15 })
  document.querySelectorAll('[data-section]').forEach(el => observer.observe(el))
  window.addEventListener('scroll', handleScroll)

  if (isTouchDevice.value && window.DeviceOrientationEvent) {
    if (typeof DeviceOrientationEvent.requestPermission === 'function') {
      DeviceOrientationEvent.requestPermission().then(state => {
        if (state === 'granted') {
          window.addEventListener('deviceorientation', handleOrientation)
        }
      }).catch(() => {})
    } else {
      window.addEventListener('deviceorientation', handleOrientation)
    }
  }
})

onUnmounted(() => {
  if (observer) observer.disconnect()
  window.removeEventListener('scroll', handleScroll)
  window.removeEventListener('deviceorientation', handleOrientation)
})
</script>

<template>
  <div
    class="profile-wrapper"
    @mousemove="handleMouseMove"
    @mouseleave="handleMouseLeave"
    @touchmove="handleTouchMove"
    @touchend="handleTouchEnd"
  >
    <!-- Floating 3D Orbs Background -->
    <div class="orbs-container" aria-hidden="true">
      <div
        class="orb orb-1"
        :style="{
          transform: `translate(${mouseX * 30}px, ${mouseY * 30}px) translateZ(${scrollY * 0.1}px)`
        }"
      />
      <div
        class="orb orb-2"
        :style="{
          transform: `translate(${mouseX * -20}px, ${mouseY * -20}px) translateZ(${scrollY * -0.08}px)`
        }"
      />
      <div
        class="orb orb-3"
        :style="{
          transform: `translate(${mouseX * 15}px, ${mouseY * -25}px)`
        }"
      />
    </div>

    <!-- Hero Section -->
    <section class="hero" data-section="hero">
      <div class="hero-bg" />
      <div class="hero-content">
        <div class="hero-layout">
          <div class="hero-character-column">
            <div class="character-container">
              <SplineCharacter />
            </div>
          </div>
          <div class="hero-card-column">
            <div
              ref="heroRef"
              class="hero-card"
              :style="{
                transform: `perspective(1000px) rotateX(${tiltX}deg) rotateY(${tiltY}deg) scale3d(1.02, 1.02, 1.02)`
              }"
            >
              <div class="hero-card-inner">
                <div class="avatar-frame">
                  <div class="avatar-glow" />
                  <div class="avatar-initials">{{ name[0] }}</div>
                </div>
                <h1 class="hero-name">{{ name }}</h1>
                <p class="hero-username">@{{ username }}</p>
                <div class="hero-title-wrapper">
                  <span class="hero-title">{{ title }}</span>
                </div>
                <p class="hero-quote">
                  <q>{{ quote }}</q>
                  <span class="quote-author">— {{ quoteAuthor }}</span>
                </p>
              </div>
            </div>
          </div>
        </div>
        <div class="scroll-indicator">
          <div class="scroll-mouse">
            <div class="scroll-wheel" />
          </div>
          <span>Desplázate</span>
        </div>
      </div>
    </section>

    <!-- About Section -->
    <section
      class="section about-section"
      data-section="about"
      :class="{ visible: visibleSections.has('about') }"
    >
      <div class="section-bg" />
      <div class="section-content">
        <h2 class="section-title">
          <span class="title-line" />
          Sobre Mí
          <span class="title-line" />
        </h2>
        <div class="about-cards">
          <div
            v-for="(item, i) in about"
            :key="i"
            class="about-card"
            :style="{ transitionDelay: `${i * 0.2}s` }"
          >
            <div class="about-card-icon">{{ ['🌱', '⚡'][i] }}</div>
            <p>{{ item }}</p>
          </div>
        </div>
      </div>
    </section>

    <!-- Skills Section -->
    <section
      class="section skills-section"
      data-section="skills"
      :class="{ visible: visibleSections.has('skills') }"
    >
      <div class="section-content">
        <h2 class="section-title">
          <span class="title-line" />
          Tecnologías
          <span class="title-line" />
        </h2>
        <p class="section-subtitle">Herramientas que uso para dar vida a mis ideas</p>
        <div class="skills-cloud">
          <div
            v-for="(skill, i) in skills"
            :key="skill.name"
            class="skill-item"
            :style="{
              transitionDelay: `${i * 0.05}s`,
              '--rotation': `${(i % 5 - 2) * 3}deg`
            }"
          >
            <div class="skill-inner">
              <img
                :src="`https://skillicons.dev/icons?i=${skill.icon}`"
                :alt="skill.name"
                class="skill-icon"
                loading="lazy"
              />
              <span class="skill-name">{{ skill.name }}</span>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- Stats Section -->
    <section
      class="section stats-section"
      data-section="stats"
      :class="{ visible: visibleSections.has('stats') }"
    >
      <div class="section-content">
        <h2 class="section-title">
          <span class="title-line" />
          Estadísticas
          <span class="title-line" />
        </h2>
        <div class="stats-grid">
          <div class="stat-card">
            <div class="stat-frame">
              <img
                src="https://github-readme-stats-sigma-five.vercel.app/api?username=kredensir&show_icons=true&theme=vision_friendly_dark&border_radius=12&bg_color=0d111700"
                alt="GitHub Stats"
                loading="lazy"
              />
            </div>
          </div>
          <div class="stat-card">
            <div class="stat-frame">
              <img
                src="https://github-readme-stats-sigma-five.vercel.app/api/top-langs/?username=kredensir&layout=compact&theme=vision_friendly_dark&border_radius=12&bg_color=0d111700"
                alt="Top Languages"
                loading="lazy"
              />
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- Projects Section -->
    <section
      class="section projects-section"
      data-section="projects"
      :class="{ visible: visibleSections.has('projects') }"
    >
      <div class="section-content">
        <h2 class="section-title">
          <span class="title-line" />
          Mis Proyectos
          <span class="title-line" />
        </h2>
        <div class="project-list">
          <a
            class="project-card"
            href="https://play.google.com/store/apps/details?id=com.boris.profit"
            target="_blank"
            rel="noopener noreferrer"
          >
            <img alt="Logo" class="project-logo" src="https://cdn.jsdelivr.net/gh/boriscr/Icons/v1-2026/1-profitpro.webp" />
            <div class="project-info">
              <span class="project-name">ProfitPro</span>
              <span class="project-desc">Plazo Fijo/FCI</span>
              <span class="project-desc">(Sin Anuncios)</span>
            </div>
            <div class="view-button">Ver</div>
          </a>
          <a
            class="project-card"
            href="https://play.google.com/store/apps/details?id=com.kredensir.vivajujuy"
            target="_blank"
            rel="noopener noreferrer"
          >
            <img alt="Logo" class="project-logo" src="https://cdn.jsdelivr.net/gh/boriscr/Icons/v1-2026/1-viva-jujuy.webp" />
            <div class="project-info">
              <span class="project-name">Viva Jujuy</span>
              <span class="project-desc">Radios locales en vivo.</span>
            </div>
            <div class="view-button">Ver</div>
          </a>
          <a
            class="project-card"
            href="#"
            target="_blank"
            rel="noopener noreferrer"
          >
            <img alt="Logo" class="project-logo" src="https://cdn.jsdelivr.net/gh/boriscr/Icons/v1-2026/1-viva-tube.webp" />
            <div class="project-info">
              <span class="project-name">Viva Tube</span>
              <span class="project-desc">Streaming sin interrupciones.</span>
            </div>
            <div class="view-button btn-soon">Pronto</div>
          </a>
          <a
            class="project-card"
            href="https://play.google.com/store/apps/details?id=com.kredensir.ritmomax"
            target="_blank"
            rel="noopener noreferrer"
          >
            <img alt="Logo" class="project-logo" src="https://cdn.jsdelivr.net/gh/boriscr/Icons/v1-2026/1-ritmoMax.webp" />
            <div class="project-info">
              <span class="project-name">RitmoMax</span>
              <span class="project-desc">Reproductor de música</span>
              <span class="project-desc">(Pendiente de Correcciones)</span>
            </div>
            <div class="view-button">Ver</div>
          </a>
          <a
            class="project-card"
            href="https://play.google.com/store/apps/details?id=com.kredensir.argento"
            target="_blank"
            rel="noopener noreferrer"
          >
            <img alt="Logo" class="project-logo" src="https://cdn.jsdelivr.net/gh/boriscr/Icons/v1-2026/1-argento.webp" />
            <div class="project-info">
              <span class="project-name">Argento</span>
              <span class="project-desc">Radios de Argentina.</span>
            </div>
            <div class="view-button">Ver</div>
          </a>
          <a
            class="project-card"
            href="#"
            target="_blank"
            rel="noopener noreferrer"
          >
            <img alt="Logo" class="project-logo" src="https://cdn.jsdelivr.net/gh/boriscr/Icons/v1-2026/1-IptvX.webp" />
            <div class="project-info">
              <span class="project-name">IptvX</span>
              <span class="project-desc">Reproductor Iptv / Xtream.</span>
            </div>
            <div class="view-button btn-soon">Pronto</div>
          </a>
        </div>
      </div>
    </section>

    <!-- Social Section -->
    <section
      class="section social-section"
      data-section="social"
      :class="{ visible: visibleSections.has('social') }"
    >
      <div class="section-content">
        <h2 class="section-title">
          <span class="title-line" />
          Conectemos
          <span class="title-line" />
        </h2>
        <div class="social-grid">
          <a
            v-for="link in socialLinks"
            :key="link.name"
            :href="link.url"
            target="_blank"
            rel="noopener noreferrer"
            class="social-link"
            :style="{ '--shadow-color': `var(--${link.icon})` }"
          >
            <div class="social-icon-wrapper">
              <svg class="social-svg" viewBox="0 0 24 24" fill="currentColor">
                <path v-if="link.icon === 'linkedin'" d="M20.447 20.452h-3.554v-5.569c0-1.328-.027-3.037-1.852-3.037-1.853 0-2.136 1.445-2.136 2.939v5.667H9.351V9h3.414v1.561h.046c.477-.9 1.637-1.85 3.37-1.85 3.601 0 4.267 2.37 4.267 5.455v6.286zM5.337 7.433a2.062 2.062 0 0 1-2.063-2.065 2.064 2.064 0 1 1 2.063 2.065zm1.782 13.019H3.555V9h3.564v11.452zM22.225 0H1.771C.792 0 0 .774 0 1.729v20.542C0 23.227.792 24 1.771 24h20.451C23.2 24 24 23.227 24 22.271V1.729C24 .774 23.2 0 22.222 0h.003z"/>
                <path v-if="link.icon === 'facebook'" d="M24 12.073c0-6.627-5.373-12-12-12s-12 5.373-12 12c0 5.99 4.388 10.954 10.125 11.854v-8.385H7.078v-3.47h3.047V9.43c0-3.007 1.792-4.669 4.533-4.669 1.312 0 2.686.235 2.686.235v2.953H15.83c-1.491 0-1.956.925-1.956 1.874v2.25h3.328l-.532 3.47h-2.796v8.385C19.612 23.027 24 18.062 24 12.073z"/>
                <path v-if="link.icon === 'instagram'" d="M12 0C8.74 0 8.333.015 7.053.072 5.775.132 4.905.333 4.14.63c-.789.306-1.459.717-2.126 1.384S.935 3.35.63 4.14C.333 4.905.131 5.775.072 7.053.012 8.333 0 8.74 0 12s.015 3.667.072 4.947c.06 1.277.261 2.148.558 2.913.306.788.717 1.459 1.384 2.126.667.666 1.336 1.079 2.126 1.384.766.296 1.636.499 2.913.558C8.333 23.988 8.74 24 12 24s3.667-.015 4.947-.072c1.277-.06 2.148-.262 2.913-.558.788-.306 1.459-.718 2.126-1.384.666-.667 1.079-1.335 1.384-2.126.296-.765.499-1.636.558-2.913.06-1.28.072-1.687.072-4.947s-.015-3.667-.072-4.947c-.06-1.277-.262-2.149-.558-2.913-.306-.789-.718-1.459-1.384-2.126C21.319 1.347 20.651.935 19.86.63c-.765-.297-1.636-.499-2.913-.558C15.667.012 15.26 0 12 0zm0 2.16c3.203 0 3.585.016 4.85.071 1.17.055 1.805.249 2.227.415.562.217.96.477 1.382.896.419.42.679.819.896 1.381.164.422.36 1.057.413 2.227.057 1.266.07 1.646.07 4.85s-.015 3.585-.074 4.85c-.061 1.17-.256 1.805-.421 2.227-.224.562-.479.96-.899 1.382-.419.419-.824.679-1.38.896-.42.164-1.065.36-2.235.413-1.274.057-1.649.07-4.859.07-3.211 0-3.586-.015-4.859-.074-1.171-.061-1.816-.256-2.236-.421-.569-.224-.96-.479-1.379-.899-.421-.419-.69-.824-.9-1.38-.165-.42-.359-1.065-.42-2.235-.045-1.26-.061-1.649-.061-4.844 0-3.196.016-3.586.061-4.861.061-1.17.255-1.814.42-2.234.21-.57.479-.96.9-1.381.419-.419.81-.689 1.379-.898.42-.166 1.051-.361 2.221-.421 1.275-.045 1.65-.06 4.859-.06l.045.03zm0 3.678a6.162 6.162 0 1 0 0 12.324 6.162 6.162 0 1 0 0-12.324zM12 16c-2.21 0-4-1.79-4-4s1.79-4 4-4 4 1.79 4 4-1.79 4-4 4zm7.846-10.405a1.441 1.441 0 1 1-2.882 0 1.441 1.441 0 0 1 2.882 0z"/>
                <path v-if="link.icon === 'github'" d="M12 .297c-6.63 0-12 5.373-12 12 0 5.303 3.438 9.8 8.205 11.385.6.113.82-.258.82-.577 0-.285-.01-1.04-.015-2.04-3.338.724-4.042-1.61-4.042-1.61C4.422 18.07 3.633 17.7 3.633 17.7c-1.087-.744.084-.729.084-.729 1.205.084 1.838 1.236 1.838 1.236 1.07 1.835 2.809 1.305 3.495.998.108-.776.417-1.305.76-1.605-2.665-.3-5.466-1.332-5.466-5.93 0-1.31.465-2.38 1.235-3.22-.135-.303-.54-1.523.105-3.176 0 0 1.005-.322 3.3 1.23.96-.267 1.98-.399 3-.405 1.02.006 2.04.138 3 .405 2.28-1.552 3.285-1.23 3.285-1.23.645 1.653.24 2.873.12 3.176.765.84 1.23 1.91 1.23 3.22 0 4.61-2.805 5.625-5.475 5.92.42.36.81 1.096.81 2.22 0 1.606-.015 2.896-.015 3.286 0 .315.21.69.825.57C20.565 22.092 24 17.592 24 12.297c0-6.627-5.373-12-12-12"/>
              </svg>
            </div>
            <span class="social-name">{{ link.name }}</span>
          </a>
        </div>
        <a
          href="https://kredensir.blogspot.com/2026/01/projects.html"
          target="_blank"
          rel="noopener noreferrer"
          class="project-button"
        >
          <span>Ver Proyectos</span>
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
            <line x1="5" y1="12" x2="19" y2="12"/>
            <polyline points="12 5 19 12 12 19"/>
          </svg>
        </a>
      </div>
    </section>

    <!-- Footer -->
    <footer class="footer">
      <img
        src="https://capsule-render.vercel.app/api?type=waving&color=gradient&height=120&section=footer"
        alt=""
        class="footer-wave"
      />
      <div class="footer-content">
        <p>Hecho con 💜 por {{ name }} &copy; {{ new Date().getFullYear() }}</p>
        <div class="footer-badge">
          <img src="https://komarev.com/ghpvc/?username=kredensir&label=VISITAS&color=0078d4&style=flat-square" alt="Profile views" />
        </div>
      </div>
    </footer>
  </div>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800&display=swap');

*, *::before, *::after {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

:root {
  --bg-primary: #0d0806;
  --bg-secondary: #1a100a;
  --bg-card: rgba(255, 245, 230, 0.03);
  --text-primary: #fff5e6;
  --text-secondary: #b8a89a;
  --accent: #ff6b35;
  --accent-2: #ffd43b;
  --accent-3: #ff4757;
  --sunset-1: #ff6b35;
  --sunset-2: #ffb347;
  --sunset-3: #ffd43b;
  --sunset-4: #ff4757;
  --linkedin: #0a66c2;
  --facebook: #1877f2;
  --instagram: #e4405f;
  --github: #f0f0f0;
  --shadow: rgba(255, 107, 53, 0.25);
}

html {
  scroll-behavior: smooth;
}

body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
  background: var(--bg-primary);
  background-image:
    radial-gradient(ellipse at 30% 20%, rgba(255, 107, 53, 0.06) 0%, transparent 50%),
    radial-gradient(ellipse at 70% 80%, rgba(255, 71, 87, 0.04) 0%, transparent 50%),
    radial-gradient(ellipse at 50% 50%, rgba(255, 180, 71, 0.03) 0%, transparent 50%);
  color: var(--text-primary);
  overflow-x: hidden;
  line-height: 1.6;
}

.profile-wrapper {
  position: relative;
  min-height: 100vh;
  overflow: hidden;
  touch-action: pan-y pinch-zoom;
}

/* ── Floating Orbs ── */
.orbs-container {
  position: fixed;
  inset: 0;
  pointer-events: none;
  z-index: 0;
  perspective: 800px;
}

.orb {
  position: absolute;
  border-radius: 50%;
  filter: blur(80px);
  will-change: transform;
  transition: transform 0.1s ease-out;
}

.orb-1 {
  width: 600px;
  height: 600px;
  background: radial-gradient(circle, rgba(255, 107, 53, 0.15), transparent 70%);
  top: -200px;
  right: -100px;
  animation: orbFloat 20s ease-in-out infinite;
}

.orb-2 {
  width: 500px;
  height: 500px;
  background: radial-gradient(circle, rgba(255, 71, 87, 0.12), transparent 70%);
  bottom: -150px;
  left: -150px;
  animation: orbFloat 25s ease-in-out infinite reverse;
}

.orb-3 {
  width: 400px;
  height: 400px;
  background: radial-gradient(circle, rgba(255, 180, 71, 0.1), transparent 70%);
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  animation: orbFloat 18s ease-in-out infinite 3s;
}

@keyframes orbFloat {
  0%, 100% { transform: translate(0, 0) scale(1); }
  33% { transform: translate(30px, -40px) scale(1.1); }
  66% { transform: translate(-20px, 30px) scale(0.95); }
}

/* ── Sections ── */
.section {
  position: relative;
  z-index: 1;
  padding: 100px 24px;
  opacity: 0;
  transform: translateY(60px) perspective(800px) rotateX(5deg) scale(0.97);
  filter: blur(4px);
  transition: all 0.9s cubic-bezier(0.16, 1, 0.3, 1);
}

.section.visible {
  opacity: 1;
  transform: translateY(0) perspective(800px) rotateX(0) scale(1);
  filter: blur(0);
}

.section-content {
  max-width: 1100px;
  margin: 0 auto;
  position: relative;
  z-index: 1;
}

.section-title {
  font-size: clamp(1.75rem, 4vw, 2.5rem);
  font-weight: 700;
  text-align: center;
  margin-bottom: 16px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 16px;
  color: var(--text-primary);
  transition: all 0.8s ease;
}

.section.visible .section-title {
  text-shadow: 0 0 40px rgba(255, 180, 71, 0.15);
}

.title-line {
  display: inline-block;
  width: 40px;
  height: 2px;
  background: linear-gradient(90deg, transparent, var(--sunset-2), transparent);
  border-radius: 2px;
}

.section-subtitle {
  text-align: center;
  color: var(--text-secondary);
  font-size: 1.05rem;
  margin-bottom: 48px;
}

/* ── Hero ── */
.hero {
  position: relative;
  z-index: 1;
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 24px;
  overflow-x: hidden;
}

.hero-bg {
  position: absolute;
  inset: 0;
  background:
    radial-gradient(ellipse at 20% 40%, rgba(255, 107, 53, 0.1) 0%, transparent 50%),
    radial-gradient(ellipse at 80% 60%, rgba(255, 71, 87, 0.06) 0%, transparent 50%),
    radial-gradient(ellipse at 50% 90%, rgba(255, 180, 71, 0.08) 0%, transparent 40%);
}

.hero-content {
  position: relative;
  z-index: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 48px;
  width: 100%;
}

.hero-layout {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 40px;
  width: 100%;
  max-width: 1100px;
}

.hero-character-column {
  flex: 0 0 380px;
  height: 450px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.character-container {
  width: 100%;
  height: 100%;
  overflow: hidden;
}

.hero-card-column {
  flex: 0 1 520px;
}

.hero-card {
  perspective: 1000px;
  transform-style: preserve-3d;
  transition: transform 0.15s ease-out;
  width: 100%;
}

.hero-card:hover .hero-card-inner {
  border-color: rgba(255, 180, 71, 0.2);
  box-shadow:
    0 20px 60px rgba(0, 0, 0, 0.3),
    0 0 80px rgba(255, 107, 53, 0.08),
    inset 0 1px 0 rgba(255, 245, 230, 0.08);
}

.hero-card-inner {
  background: linear-gradient(
    135deg,
    rgba(255, 245, 230, 0.06) 0%,
    rgba(255, 107, 53, 0.03) 50%,
    rgba(255, 245, 230, 0.04) 100%
  );
  backdrop-filter: blur(24px);
  border: 1px solid rgba(255, 180, 71, 0.12);
  border-radius: 32px;
  padding: 48px 40px;
  text-align: center;
  transform-style: preserve-3d;
  box-shadow:
    0 20px 60px rgba(0, 0, 0, 0.3),
    0 0 80px rgba(255, 107, 53, 0.05),
    inset 0 1px 0 rgba(255, 245, 230, 0.08);
}

.avatar-frame {
  width: 120px;
  height: 120px;
  border-radius: 50%;
  margin: 0 auto 24px;
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  transform: translateZ(40px);
}

.avatar-glow {
  position: absolute;
  inset: -4px;
  border-radius: 50%;
  background: conic-gradient(from 0deg, var(--sunset-1), var(--sunset-2), var(--sunset-3), var(--sunset-4), var(--sunset-1));
  animation: spinGlow 4s linear infinite;
  filter: blur(4px);
}

@keyframes spinGlow {
  to { transform: rotate(360deg); }
}

.avatar-initials {
  width: 112px;
  height: 112px;
  border-radius: 50%;
  background: linear-gradient(135deg, var(--bg-primary), var(--bg-secondary));
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 3rem;
  font-weight: 800;
  color: var(--accent);
  position: relative;
  z-index: 1;
  border: 2px solid rgba(255, 255, 255, 0.1);
}

.hero-name {
  font-size: clamp(2rem, 5vw, 3rem);
  font-weight: 800;
  background: linear-gradient(135deg, var(--text-primary) 0%, var(--sunset-2) 50%, var(--sunset-1) 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  transform: translateZ(30px);
}

.hero-username {
  color: var(--text-secondary);
  font-size: 1rem;
  margin-bottom: 16px;
  transform: translateZ(20px);
}

.hero-title-wrapper {
  display: inline-block;
  padding: 6px 20px;
  background: linear-gradient(135deg, rgba(255, 107, 53, 0.2), rgba(255, 212, 59, 0.15));
  border: 1px solid rgba(255, 107, 53, 0.3);
  border-radius: 100px;
  margin-bottom: 24px;
  transform: translateZ(25px);
}

.hero-title {
  font-size: 0.95rem;
  font-weight: 600;
  color: var(--sunset-2);
  letter-spacing: 0.5px;
}

.hero-quote {
  color: var(--text-secondary);
  font-size: 0.95rem;
  font-style: italic;
  line-height: 1.7;
  transform: translateZ(15px);
}

@keyframes borderGlow {
  0%, 100% { border-color: rgba(255, 180, 71, 0.12); }
  50% { border-color: rgba(255, 107, 53, 0.25); }
}

.quote-author {
  display: block;
  margin-top: 8px;
  font-size: 0.8rem;
  color: var(--text-secondary);
  opacity: 0.7;
}

/* ── Scroll Indicator ── */
.scroll-indicator {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  color: var(--text-secondary);
  font-size: 0.75rem;
  letter-spacing: 2px;
  text-transform: uppercase;
  animation: bounceDown 2s ease-in-out infinite;
}

.scroll-mouse {
  width: 26px;
  height: 40px;
  border: 2px solid var(--text-secondary);
  border-radius: 13px;
  display: flex;
  justify-content: center;
  padding-top: 8px;
}

.scroll-wheel {
  width: 3px;
  height: 8px;
  background: var(--accent);
  border-radius: 2px;
  animation: scrollWheel 1.5s ease-in-out infinite;
}

@keyframes scrollWheel {
  0% { transform: translateY(0); opacity: 1; }
  100% { transform: translateY(12px); opacity: 0; }
}

@keyframes bounceDown {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(6px); }
}

/* ── About ── */
.about-section {
  background: linear-gradient(180deg, transparent, var(--bg-secondary) 50%, transparent);
}

.about-cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 24px;
  max-width: 800px;
  margin: 0 auto;
}

.about-card {
  background: var(--bg-card);
  border: 1px solid rgba(255, 255, 255, 0.06);
  border-radius: 20px;
  padding: 32px;
  text-align: center;
  transition: all 0.6s cubic-bezier(0.34, 1.56, 0.64, 1);
  transition-delay: inherit;
  opacity: 0;
  transform: translateY(40px) scale(0.92);
  will-change: transform, opacity;
}

.section.visible .about-card {
  opacity: 1;
  transform: translateY(0) scale(1);
}

.about-card:hover {
  transform: translateY(-6px);
  border-color: rgba(255, 107, 53, 0.3);
  box-shadow: 0 12px 40px rgba(255, 107, 53, 0.12);
  background: rgba(255, 107, 53, 0.05);
}

.about-card-icon {
  font-size: 2.5rem;
  margin-bottom: 16px;
}

.about-card p {
  color: var(--text-secondary);
  line-height: 1.7;
}

/* ── Skills ── */
.skills-section {
  background: linear-gradient(180deg, transparent, rgba(255, 107, 53, 0.04), transparent);
}

.skills-cloud {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 12px;
  max-width: 800px;
  margin: 0 auto;
}

.skill-item {
  perspective: 600px;
  transform-style: preserve-3d;
  transition: all 0.5s cubic-bezier(0.34, 1.56, 0.64, 1);
  transition-delay: inherit;
  opacity: 0;
  transform: translateY(30px) scale(0.85) rotateY(15deg);
  will-change: transform, opacity;
}

.section.visible .skill-item {
  opacity: 1;
  transform: translateY(0) scale(1) rotateY(0);
}

.skill-inner {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px 18px;
  background: var(--bg-card);
  border: 1px solid rgba(255, 255, 255, 0.06);
  border-radius: 12px;
  transition: all 0.3s ease;
  cursor: default;
}

.skill-item:hover .skill-inner {
  transform:
    perspective(600px)
    rotateX(calc(var(--rotation) * -1))
    rotateY(calc(var(--rotation) * 1.5))
    translateZ(12px);
  border-color: rgba(255, 180, 71, 0.4);
  background: rgba(255, 107, 53, 0.08);
  box-shadow:
    0 8px 30px rgba(255, 107, 53, 0.15),
    inset 0 1px 0 rgba(255, 107, 53, 0.15);
}

.skill-icon {
  width: 24px;
  height: 24px;
  object-fit: contain;
}

.skill-name {
  font-size: 0.85rem;
  font-weight: 500;
  color: var(--text-primary);
}

/* ── Stats ── */
.stats-section {
  background: linear-gradient(180deg, transparent, var(--bg-secondary) 50%, transparent);
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 24px;
  max-width: 900px;
  margin: 0 auto;
}

.stat-card {
  perspective: 800px;
  transition: all 0.7s cubic-bezier(0.34, 1.56, 0.64, 1);
  opacity: 0;
  transform: translateY(50px) rotateY(15deg) scale(0.95);
  will-change: transform, opacity;
}

.section.visible .stat-card {
  opacity: 1;
  transform: translateY(0) rotateY(0) scale(1);
}

.section.visible .stat-card:nth-child(2) {
  transition-delay: 0.2s;
}

.stat-frame {
  background: var(--bg-card);
  border: 1px solid rgba(255, 255, 255, 0.06);
  border-radius: 20px;
  padding: 20px;
  overflow: hidden;
  transition: all 0.4s ease;
}

.stat-card:hover .stat-frame {
  transform: perspective(800px) rotateY(-4deg) translateZ(10px);
  border-color: rgba(255, 180, 71, 0.3);
  box-shadow: 0 16px 48px rgba(255, 107, 53, 0.12);
}

.stat-frame img {
  width: 100%;
  height: auto;
  display: block;
}

/* ── Projects ── */
.projects-section {
  background: linear-gradient(180deg, transparent, rgba(255, 180, 71, 0.04), transparent);
}

.project-list {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 16px;
  max-width: 920px;
  margin: 0 auto;
}

.project-card {
  display: flex;
  align-items: center;
  gap: 14px;
  padding: 14px 18px;
  background: var(--bg-card);
  border: 1px solid rgba(255, 255, 255, 0.06);
  border-radius: 16px;
  text-decoration: none;
  transition: all 0.5s cubic-bezier(0.34, 1.56, 0.64, 1);
  opacity: 0;
  transform: translateX(-30px) scale(0.95);
  will-change: transform, opacity;
}

.section.visible .project-card {
  opacity: 1;
  transform: translateX(0) scale(1);
}

.section.visible .project-card:nth-child(1) { transition-delay: 0s; }
.section.visible .project-card:nth-child(2) { transition-delay: 0.06s; }
.section.visible .project-card:nth-child(3) { transition-delay: 0.12s; }
.section.visible .project-card:nth-child(4) { transition-delay: 0.18s; }
.section.visible .project-card:nth-child(5) { transition-delay: 0.24s; }
.section.visible .project-card:nth-child(6) { transition-delay: 0.3s; }

.project-card:hover {
  transform: translateY(-4px) scale(1.02);
  border-color: rgba(255, 180, 71, 0.3);
  background: rgba(255, 107, 53, 0.05);
  box-shadow: 0 8px 30px rgba(255, 107, 53, 0.12);
}

.project-logo {
  width: 48px;
  height: 48px;
  border-radius: 12px;
  object-fit: cover;
  flex-shrink: 0;
}

.project-info {
  flex: 1;
  min-width: 0;
}

.project-name {
  display: block;
  font-size: 0.95rem;
  font-weight: 600;
  color: var(--text-primary);
  margin-bottom: 2px;
}

.project-desc {
  display: block;
  font-size: 0.78rem;
  color: var(--text-secondary);
  line-height: 1.3;
}

.view-button {
  padding: 6px 16px;
  background: linear-gradient(135deg, var(--sunset-1), #e85d1a);
  border-radius: 8px;
  color: #fff;
  font-size: 0.8rem;
  font-weight: 600;
  white-space: nowrap;
  transition: all 0.3s ease;
  box-shadow: 0 4px 12px rgba(255, 107, 53, 0.3);
}

.view-button.btn-soon {
  background: rgba(255, 255, 255, 0.08);
  color: var(--text-secondary);
  box-shadow: none;
  cursor: default;
}

.project-card:hover .view-button:not(.btn-soon) {
  transform: scale(1.05);
  box-shadow: 0 6px 20px rgba(255, 107, 53, 0.4);
}

/* ── Social ── */
.social-section {
  background: linear-gradient(180deg, transparent, rgba(255, 71, 87, 0.04), transparent);
}

.social-grid {
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
  gap: 16px;
  margin-bottom: 40px;
}

.social-link {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 12px 24px;
  background: var(--bg-card);
  border: 1px solid rgba(255, 255, 255, 0.06);
  border-radius: 14px;
  text-decoration: none;
  transition: all 0.5s cubic-bezier(0.34, 1.56, 0.64, 1);
  color: var(--text-primary);
  opacity: 0;
  transform: translateY(30px) scale(0.9);
  will-change: transform, opacity;
}

.section.visible .social-link {
  opacity: 1;
  transform: translateY(0) scale(1);
}

.section.visible .social-link:nth-child(1) { transition-delay: 0s; }
.section.visible .social-link:nth-child(2) { transition-delay: 0.1s; }
.section.visible .social-link:nth-child(3) { transition-delay: 0.2s; }
.section.visible .social-link:nth-child(4) { transition-delay: 0.3s; }

.social-link:hover {
  transform: translateY(-4px) scale(1.03);
  border-color: var(--shadow-color);
  background: rgba(255, 255, 255, 0.06);
  box-shadow: 0 8px 30px color-mix(in srgb, var(--shadow-color) 25%, transparent);
}

.social-icon-wrapper {
  width: 28px;
  height: 28px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.social-svg {
  width: 100%;
  height: 100%;
  transition: transform 0.3s ease;
}

.social-link:hover .social-svg {
  transform: scale(1.15);
}

.social-name {
  font-size: 0.9rem;
  font-weight: 500;
}

/* ── Project Button ── */
.project-button {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  padding: 16px 36px;
  background: linear-gradient(135deg, var(--sunset-1), #d94f0a);
  border: none;
  border-radius: 14px;
  color: white;
  font-size: 1rem;
  font-weight: 600;
  text-decoration: none;
  cursor: pointer;
  transition: all 0.3s ease;
  margin: 0 auto;
  display: flex;
  width: fit-content;
  box-shadow: 0 8px 30px rgba(255, 107, 53, 0.3);
}

.project-button:hover {
  transform: translateY(-3px) scale(1.02);
  box-shadow: 0 12px 40px rgba(255, 107, 53, 0.4);
}

.project-button:active {
  transform: translateY(0) scale(0.98);
}

.project-button svg {
  transition: transform 0.3s ease;
}

.project-button:hover svg {
  transform: translateX(4px);
}

/* ── Footer ── */
.footer {
  position: relative;
  z-index: 1;
  text-align: center;
}

.footer-wave {
  width: 100%;
  display: block;
  pointer-events: none;
}

.footer-content {
  padding: 24px;
  background: var(--bg-primary);
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
}

.footer-content p {
  color: var(--text-secondary);
  font-size: 0.85rem;
}

.footer-badge img {
  height: 20px;
}

/* ── Responsive ── */
@media (max-width: 1024px) {
  .hero-layout {
    flex-direction: column-reverse;
    gap: 20px;
  }

  .hero-character-column {
    flex: 0 0 auto;
    width: 100%;
    max-width: 320px;
    height: 350px;
    margin: 0 auto;
  }

  .hero-card-column {
    flex: 0 0 auto;
    width: 100%;
    max-width: 520px;
    margin: 0 auto;
  }
}

@media (max-width: 768px) {
  .hero-card-inner {
    padding: 32px 24px;
  }

  .avatar-frame {
    width: 90px;
    height: 90px;
  }

  .avatar-initials {
    width: 84px;
    height: 84px;
    font-size: 2.2rem;
  }

  .hero-character-column {
    max-width: 280px;
    height: 300px;
  }

  .section {
    padding: 64px 20px;
  }

  .about-cards {
    grid-template-columns: 1fr;
  }

  .stats-grid {
    grid-template-columns: 1fr;
  }

  .stat-card {
    min-width: 0;
  }

  .social-grid {
    flex-direction: column;
    align-items: center;
  }

  .social-link {
    width: 100%;
    justify-content: center;
    max-width: 320px;
  }

  .skills-cloud {
    gap: 8px;
  }

  .skill-inner {
    padding: 8px 14px;
  }

  .skill-icon {
    width: 20px;
    height: 20px;
  }

  .skill-name {
    font-size: 0.8rem;
  }

  .title-line {
    width: 24px;
  }
}

@media (max-width: 480px) {
  .hero-card-inner {
    padding: 24px 16px;
    border-radius: 24px;
  }

  .hero-title-wrapper {
    padding: 5px 14px;
  }

  .hero-quote {
    font-size: 0.85rem;
  }

  .hero-character-column {
    max-width: 100%;
    height: 460px;
  }

  .section {
    padding: 48px 16px;
  }

  .project-button {
    padding: 14px 28px;
    font-size: 0.9rem;
  }
}

@media (min-width: 1200px) {
  .hero-character-column {
    flex: 0 0 440px;
    height: 500px;
  }

  .hero-card-column {
    flex: 0 1 600px;
  }

  .hero-card {
    max-width: 600px;
  }

  .hero-card-inner {
    padding: 60px 48px;
  }

  .about-cards {
    max-width: 900px;
  }
}
</style>

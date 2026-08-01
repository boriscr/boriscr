<script setup>
import { ref, onMounted } from 'vue'

const loaded = ref(false)

onMounted(() => {
  if (customElements.get('spline-viewer')) {
    loaded.value = true
    return
  }
  const script = document.createElement('script')
  script.type = 'module'
  script.src = 'https://unpkg.com/@splinetool/viewer@1.12.98/build/spline-viewer.js'
  script.onload = () => { loaded.value = true }
  document.head.appendChild(script)
})
</script>

<template>
  <div class="spline-wrapper">
    <spline-viewer
      v-if="loaded"
      url="https://prod.spline.design/WiXL5nNNBVUJaoYs/scene.splinecode"
    />
    <div v-else class="spline-loading">
      <div class="loading-spinner" />
    </div>
  </div>
</template>

<style scoped>
.spline-wrapper {
  width: 100%;
  height: 100%;
  min-height: 400px;
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
}

spline-viewer {
  width: 100%;
  height: 100%;
  display: block;
  max-width: 100%;
  max-height: 100%;
}

.spline-loading {
  width: 100%;
  height: 100%;
  min-height: 400px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.loading-spinner {
  width: 32px;
  height: 32px;
  border: 3px solid rgba(255, 107, 53, 0.15);
  border-top-color: var(--accent, #ff6b35);
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}
</style>

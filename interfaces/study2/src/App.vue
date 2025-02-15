<script setup>
import { ref, onMounted } from 'vue'
import conditions from '../public/conditions.json'
import Model from './components/Model.vue'
import Chat from './components/Chat.vue'
import Modal from './components/Modal.vue'

// State management for application conditions and UI states
const condition = ref(null)          // Current selected experimental condition
const error = ref(null)              // Error state for handling failures
const isTalking = ref(false)         // Tracks if the AI agent is currently speaking
const isVideoLoaded = ref(false)     // Background video loading state
const isModelLoaded = ref(false)     // 3D model loading state
const showModal = ref(true)          // Controls visibility of the introduction modal
const interventionStarted = ref(false) // Tracks if the intervention session has begun

// Event Handlers
const handleModalEnter = () => {
  showModal.value = false
  interventionStarted.value = true
}

const handleVideoLoad = () => {
  isVideoLoaded.value = true
}

const handleModelLoad = () => {
  isModelLoaded.value = true
}

// Lifecycle hook to initialize condition from URL parameters
onMounted(() => {
  const urlParams = new URLSearchParams(window.location.search)
  const groupId = urlParams.get('group')

  if (groupId) {
    condition.value = conditions[groupId]
  }
})

// Condition selection handler with URL state management
const selectCondition = (groupId) => {
  // Update URL with selected condition
  const url = new URL(window.location)
  url.searchParams.set('group', groupId)
  window.history.pushState({}, '', url)

  // Update application state
  condition.value = conditions[groupId]
  // Reset loading states when a new condition is selected
  isVideoLoaded.value = false
  isModelLoaded.value = false
}

const handleTalkingState = (talking) => {
  isTalking.value = talking
}
</script>

<template>
  <div v-if="showModal" class="modal-container">
    <Modal :show="showModal" @enter="handleModalEnter" :interventionDescription="condition?.description || ''" :isModelLoaded="isModelLoaded" />
  </div>
  <template v-if="!showModal">
    <div class="app-container">
      <!-- Show video for both treatment and control_personal -->
      <video v-if="condition?.id?.startsWith('treatment_') || condition?.id?.startsWith('control_personal_')" 
        autoplay muted loop playsinline class="background-video"
        @loadeddata="handleVideoLoad">
        <source src="@/assets/bg_loop.mp4" type="video/mp4">
      </video>

      <div v-if="condition" class="main-container">
        <!-- Left side - Chat section -->
        <div class="left-panel">
          <Chat v-if="interventionStarted" @talking-state-change="handleTalkingState" :condition="condition"
            :ready="isVideoLoaded && isModelLoaded" />
        </div>

        <!-- Right side - 3D Model -->
        <div class="right-panel" :class="{ 'no-negative-margin': condition?.id?.startsWith('control_scientific') }">
          <Model v-if="interventionStarted && condition.model" 
            :condition="condition" 
            :is-talking="isTalking"
            :loading="!isVideoLoaded || !isModelLoaded"
            :is-model-loaded="isModelLoaded" 
            @model-loaded="handleModelLoad" />
        </div>
      </div>

      <div v-else-if="error">
        {{ error }}
      </div>

      <div v-else class="condition-selector">
        <h2 class="condition-selector-title">Select a Condition</h2>
        <div class="condition-buttons">
          <button v-for="(value, key) in conditions" :key="key" @click="selectCondition(key)" class="condition-button">
            <div class="condition-name">{{ value.id || key }}</div>
            <div class="condition-description">{{ value.description || 'No description available' }}</div>
          </button>
        </div>
      </div>
    </div>
  </template>
</template>

<style>
/* Global Styles */
body {
  margin: 0;
  /* Reset margin */
  background: #000000;
  /* Set body background to black */
}

#app {
  width: 100%;
  height: 100%;
  position: relative;
  font-family: "Space Grotesk", -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Arial, sans-serif;
  background: #000000 !important;
  color: #ffffff;
}
</style>

<style scoped>
/* Layout Containers */
.app-container {
  position: relative;
  width: 100%;
  height: 100%;
}

/* Modal Styling */
.modal-container {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 100vh;
}

/* Background Video Styling */
.background-video {
  position: fixed;
  right: 0;
  bottom: 0;
  min-width: 100%;
  min-height: 100%;
  width: auto;
  height: auto;
  z-index: -1;
  -o-object-fit: cover;
  object-fit: cover;
  -webkit-transform: scale(1.01);
  transform: scale(1.01);
}

.main-container {
  display: flex;
  flex-direction: row;
  height: 100vh;
  width: 100vw;
  position: fixed;
  top: 0;
  left: 0;

  &>*+* {
    margin-left: 1rem;
  }
}

.left-panel {
  width: 70%;
  height: 100%;
  padding: 2rem;
  padding-right: 0px !important;
  overflow-y: auto;
}

.right-panel {
  width: calc(30% + 80px);
  height: 100%;
  padding: 0px !important;
  margin: 0px !important;
  margin-left: -80px !important;
}

.right-panel.no-negative-margin {
  width: 30%;
  margin-left: 0px !important;
}

/* Update media query for screens smaller than MD (768px) */
@media (max-width: 768px) {
  .main-container {
    &>*+* {
      margin-left: 0.5rem;
    }
  }

  .left-panel,
  .right-panel {
    width: 65%;
    height: 100%;
  }

  .right-panel {
    width: 35%;
  }

  .left-panel {
    padding: 0.5rem;
  }
}

/* Condition Selector Styling */
.condition-selector {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100vh;
  padding: 1rem;
  color: #333;
  width: 100vw;
}

.condition-selector h2 {
  font-size: 32px;
  font-weight: 500;
  letter-spacing: -0.02em;
  margin-bottom: 2rem;
  color: #333;
}

.condition-buttons {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1rem;
  width: 100%;
  max-width: 100vw;
  margin: 1rem auto;
  padding: 15px;
  background: rgba(255, 255, 255);
  border-radius: 20px;
}

.condition-button {
  padding: 15px;
  border: 2px solid rgba(0, 0, 0, 0.2);
  border-radius: 30px;
  background: rgba(255, 255, 255, 0.2);
  cursor: pointer;
  color: #333;
  transition: all 0.3s ease;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  text-align: left;
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.condition-button:hover {
  background: rgba(255, 255, 255, 0.3);
  -webkit-transform: translateY(-2px);
  transform: translateY(-2px);
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.15);
  color: #333;
}

.condition-name {
  font-size: 18px;
  font-weight: 500;
  color: #333;
}

.condition-description {
  font-size: 14px;
  opacity: 0.8;
  line-height: 1.4;
  font-weight: 400;
  color: #333;
}

/* Update media queries for responsive behavior */
@media (max-width: 1024px) {
  .condition-buttons {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 480px) {
  .condition-buttons {
    grid-template-columns: 1fr;
    gap: 0.75rem;
    padding: 10px;
  }

  .condition-selector h2 {
    font-size: 24px;
    margin-bottom: 1rem;
  }

  .condition-name {
    font-size: 16px;
  }

  .condition-description {
    font-size: 13px;
  }
}
</style>
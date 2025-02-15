<script setup>
import { ref, watch, onMounted, onUnmounted } from 'vue'
import '@google/model-viewer'

const props = defineProps({
  condition: {
    type: Object,
    required: true
  },
  isTalking: {
    type: Boolean,
    default: false
  },
  loading: {
    type: Boolean,
    default: true
  },
  isModelLoaded: {
    type: Boolean,
    default: false
  }
})

const modelViewer = ref(null)
const currentAnimations = ref([])
const targetAnimations = ref(null)
let currentAnimationHandler = null
let animationTimer = null
let rotationTimer = null

const talkingIndex = ref(0)
const idleIndex = ref(0)

const ANIMATION_DURATION = props.condition.model.animations?.switch || 1000  // Switch animation every 5 seconds while talking
const ROTATION_SPEED = 0.5 // Degrees per frame
let currentRotation = props.condition.model.defaultRotation || 0 // Initialize with default from props if available
const MAX_ROTATION = -35 
const MIN_ROTATION = -10 
let targetRotation = currentRotation // Initialize target to match current

// Calculate camera distance based on model scale
const CAMERA_DISTANCE = `${120 * (props.condition.model.scale || 1)}%`
const ORBIT_DISTANCE = CAMERA_DISTANCE

const cleanupCurrentAnimation = () => {
  if (currentAnimationHandler) {
    modelViewer.value?.removeEventListener('finished', currentAnimationHandler)
    currentAnimationHandler = null
  }
  if (animationTimer) {
    clearTimeout(animationTimer)
    animationTimer = null
  }
  if (rotationTimer) {
    cancelAnimationFrame(rotationTimer)
    rotationTimer = null
  }
}

// Helper to play animations with crossfade
const playAnimations = (animationNames, force = false) => {
  if (!modelViewer.value || !animationNames?.length) return

  // Clear any existing timer
  if (animationTimer) {
    clearTimeout(animationTimer)
    animationTimer = null
  }

  const availableAnims = modelViewer.value.availableAnimations
  const validAnims = animationNames.filter(name => availableAnims.includes(name))
  
  if (validAnims.length === 0) return

  // Special case for treatment_jellyfish: only play the first animation and loop it
  if (props.condition.id === 'treatment_jellyfish') {
    currentAnimations.value = [validAnims[0]]; // Set to the first valid animation
    targetAnimations.value = currentAnimations.value;

    // Play the first animation indefinitely
    modelViewer.value.animationName = currentAnimations.value[0];
    modelViewer.value.play({
      repetitions: Infinity, // Loop indefinitely
      crossfadeDuration: 0 // No crossfade for looping
    });
    return;
  }

  // Don't restart if already playing these animations unless forced
  if (!force && 
      validAnims.length === currentAnimations.value.length && 
      validAnims.every(anim => currentAnimations.value.includes(anim))) {
    return
  }

  // Clean up any existing animation handler
  cleanupCurrentAnimation()

  currentAnimations.value = validAnims
  targetAnimations.value = validAnims
  
  let currentIndex = 0
  
  const playNext = () => {
    if (!modelViewer.value) return

    const currentAnim = currentAnimations.value[currentIndex]
    const crossfadeDuration = props.condition.model.animations.crossfade || 800

    // Start the next animation with crossfade before the current one ends
    const playNextAnimation = () => {
      if (!modelViewer.value) return
      
      modelViewer.value.animationName = currentAnim
      modelViewer.value.play({
        repetitions: 1,
        crossfadeDuration: crossfadeDuration
      })

      currentIndex = (currentIndex + 1) % currentAnimations.value.length

      // Schedule the next animation
      if (props.isTalking) {
        // Start the next animation slightly before the current one ends
        // This ensures there's always an overlap for smooth crossfading
        const animationDuration = ANIMATION_DURATION - (crossfadeDuration / 2)
        animationTimer = setTimeout(playNext, Math.max(animationDuration, crossfadeDuration))
      }
    }

    // Add a small delay before starting the next animation to ensure proper crossfading
    setTimeout(playNextAnimation, crossfadeDuration / 4)
  }

  currentAnimationHandler = () => {
    if (modelViewer.value && !props.isTalking) {
      playNext()
    }
  }

  modelViewer.value.addEventListener('finished', currentAnimationHandler)
  playNext()
}

const updateRotation = () => {
  if (!modelViewer.value || !props.isTalking) return
  
  // Smoothly interpolate towards target rotation
  const diff = targetRotation - currentRotation
  if (Math.abs(diff) > 0.1) {
    currentRotation += Math.sign(diff) * ROTATION_SPEED
  } else {
    // Set new random target between -35 and -15
    targetRotation = -MIN_ROTATION - Math.random() * (MAX_ROTATION - MIN_ROTATION)
  }
  
  // Ensure modelViewer is not null before setting cameraOrbit
  if (modelViewer.value) {
    modelViewer.value.cameraOrbit = `${currentRotation}deg 90deg ${ORBIT_DISTANCE}`
  }
  rotationTimer = requestAnimationFrame(updateRotation)
}

// Add periodic rotation change while talking
const startPeriodicRotation = () => {
  const changeRotation = () => {
    if (props.isTalking) {
      // Set new random target between -35 and -15
      targetRotation = -MIN_ROTATION - Math.random() * (MAX_ROTATION - MIN_ROTATION)
      animationTimer = setTimeout(changeRotation, ANIMATION_DURATION)
    }
  }
  changeRotation()
}

// Watch for talking state changes
watch(() => props.isTalking, (newValue) => {
  if (!modelViewer.value) return
  
  const animations = props.condition.model.animations
  if (!animations) return

  // Add condition to check for treatment_jellyfish
  if (props.condition.id === 'treatment_jellyfish') {
    // Keep idle all times, do not change to talking
    cleanupCurrentAnimation()
    playAnimations(animations.idle, true)
    return
  }

  if (newValue) {
    // Clean up existing animations before starting talking animations
    cleanupCurrentAnimation()
    playAnimations(animations.talking)
    targetRotation = Math.random() * (MAX_ROTATION * 2) - MAX_ROTATION
    updateRotation()
    startPeriodicRotation()
  } else {
    // When stopping talking, let the current animation finish
    // before transitioning to idle
    if (animationTimer) {
      const remainingTime = ANIMATION_DURATION
      setTimeout(() => {
        cleanupCurrentAnimation()
        playAnimations(animations.idle, true)
        // Keep the current rotation instead of resetting
        if (modelViewer.value) {
          modelViewer.value.cameraOrbit = `${currentRotation}deg 90deg ${ORBIT_DISTANCE}`
        }
      }, remainingTime)
    } else {
      // If no animation is currently playing, switch to idle immediately
      cleanupCurrentAnimation()
      playAnimations(animations.idle, true)
      if (modelViewer.value) {
        modelViewer.value.cameraOrbit = `${currentRotation}deg 90deg ${ORBIT_DISTANCE}`
      }
    }
  }
})

const emit = defineEmits(['model-loaded'])

const handleModelLoad = () => {
  console.log('Model loaded')
  emit('model-loaded')
}

// Play idle animations when model loads
onMounted(() => {
  if (!modelViewer.value) return
  
  modelViewer.value.addEventListener('load', handleModelLoad)
  modelViewer.value.minimumRenderScale = 0.25

  talkingIndex.value = 0
  idleIndex.value = 0
})

onUnmounted(() => {
  cleanupCurrentAnimation()
  if (modelViewer.value) {
    modelViewer.value.removeEventListener('load', handleModelLoad)
  }
})

// Helper functions to check condition types
const isTreatmentCondition = () => {
  return props.condition?.id?.startsWith('treatment_')
}

const isControlPersonal = () => {
  return props.condition?.id?.startsWith('control_personal_')
}

// Simplified loading logic
const shouldShowLoader = () => {
  // Only show loader for treatment and control_personal conditions
  if (isTreatmentCondition() || isControlPersonal()) {
    return props.loading
  }
  // No loader for any other conditions (including control_scientific)
  return false
}

</script>

<template>
  <div class="model-container">
    <model-viewer
      v-if="['gltf', 'glb'].includes(condition.model.type)"
      ref="modelViewer"
      :animation-crossfade-duration="condition.model.animations.crossfade || 800"
      :timeScale="condition.model.animations.speed || 1.5"
      :src="`/models/${condition.name}.${condition.model.type}`"
      alt="3D model"
      autoplay
      :field-of-view="35"
      :camera-orbit="`${currentRotation}deg 90deg ${CAMERA_DISTANCE}`"
      :min-camera-orbit="`auto auto ${CAMERA_DISTANCE}`"
      :max-camera-orbit="`auto auto ${CAMERA_DISTANCE}`"
      :camera-target="`${condition.model.translate?.x || 0}m ${condition.model.translate?.y || 0}m 0m`"
      :minimum-render-scale="0.25"
      @load="handleModelLoad"
    >
    </model-viewer>

    <!-- Show loader only for treatment and control_personal conditions -->
    <div v-if="shouldShowLoader()" class="loader-backdrop">
      <div class="loader">
        <h1>Loading</h1>
        <div class="spinner"></div>
      </div>
    </div>
    
    <!-- Show image if type is image -->
    <img class="model-image" v-else-if="['jpg', 'png'].includes(condition.model.type)"
      :src="`/models/${condition.name}.${condition.model.type}`">
  </div>
</template>

<style scoped>
.loader {
  position: absolute;
  top: 50%;
  left: 50%;
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
  transform: translate(-50%, -50%);
  font-size: 18px;
  color: #000000;
}

.spinner {
  border: 4px solid rgba(0, 0, 0, 0.3);
  border-top: 4px solid #000;
  border-radius: 50%;
  width: 30px;
  height: 30px;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.model-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  margin-left: 80px;
  
}

.model-container {
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}

model-viewer {
  width: 100%;
  height: 100%;
  background-color: transparent;
}

img {
  max-width: 100%;
  max-height: 100%;
  object-fit: contain;
}

@media (max-width: 768px) {
  .model-container {
    padding-left: 0;
  }

  .model-image {
    margin-left: 40px;
  }
}

.loader-backdrop {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: white;
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 999;
}
</style>


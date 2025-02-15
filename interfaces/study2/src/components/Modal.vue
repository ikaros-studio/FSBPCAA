<template>
    <div v-if="show" class="modal-overlay">
        <!-- Modal backdrop for overlay effect -->

        <!-- Main modal container -->
        <div class="modal-content">
            <h2>Welcome</h2>
            <div class="modal-body">
                <p>
                    You are about to participate in a messaging intervention that explores climate change and plastic pollution.
                </p>
                <p class="modal-subtitle">
                    Please take your time to read and engage with the content that will be presented.
                </p>
                <!-- Loading indicator while processing -->
                <div v-if="loading" class="loader">Loading...</div>
            </div>
            <!-- Interactive button with loading state management -->
            <button @click="handleEnter" class="enter-button" :disabled="loading">
                <span v-if="loading" class="spinner"></span>
                <span v-else>Enter</span>
            </button>
        </div>
    </div>
</template>

<script setup>
// Import necessary Vue composition API functions
import { defineProps, defineEmits, ref } from 'vue'

// Component props definition
const props = defineProps({
    show: {
        type: Boolean,
        required: true
    },
    interventionDescription: {
        type: String,
        required: true
    }
})

// Event emission setup
const emit = defineEmits(['enter'])
// Loading state management
const loading = ref(false)

// Handler for enter button click
// Sets loading state, emits enter event, and resets loading after delay
const handleEnter = () => {
    loading.value = true
    emit('enter')
    setTimeout(() => {
        loading.value = false
    }, 5000)
}
</script>

<style scoped>
/* Loading indicator styling */
.loader {
    margin: 20px 0;
    font-size: 18px;
    color: #000000;
}

/* Modal backdrop styling */
.modal-backdrop {
    display: none;
}

/* Main modal container styling */
.modal-content {
    max-width: 600px;
    margin: auto;
    padding: 30px;
    border-radius: 20px;
    font-family: "Space Grotesk", -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Arial, sans-serif;
    text-align: center;
    background: #ffffff;
}

/* Modal header styling */
.modal-content h2 {
    font-size: 32px;
    font-weight: 500;
    color: #000000;
    margin-bottom: 20px;
    letter-spacing: -0.02em;
}

/* Modal body container styling */
.modal-body {
    margin-bottom: 30px;
}

/* Modal text content styling */
.modal-body p {
    font-size: 18px;
    line-height: 1.6;
    color: #000000;
    margin-bottom: 15px;
}

/* Subtitle emphasis styling */
.modal-subtitle {
    font-style: italic;
    opacity: 0.8;
    color: #000000;
}

/* Enter button styling */
.enter-button {
    width: 100%;
    padding: 15px 20px;
    color: #000000;
    border: 2px solid rgba(0, 0, 0, 0.2);
    border-radius: 30px;
    background: rgba(0, 0, 0, 0.05);
    font-family: "Space Grotesk", -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Arial, sans-serif;
    font-size: 18px;
    cursor: pointer;
    transition: all 0.3s ease;
}

/* Enter button hover state */
.enter-button:hover {
    background: rgba(0, 0, 0, 0.1);
    color: #000000;
}

/* Loading spinner animation styling */
.spinner {
    border: 2px solid rgba(0, 0, 0, 0.2);
    border-top: 2px solid #000000;
    border-radius: 50%;
    width: 20px;
    height: 20px;
    animation: spin 1s linear infinite;
    display: inline-block;
    margin-right: 10px;
}

/* Spinner rotation animation keyframes */
@keyframes spin {
    0% { transform: rotate(0deg); }
    100% { transform: rotate(360deg); }
}
</style> 
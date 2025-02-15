<template>
    <div v-if="show" class="modal-overlay">
        <!-- Backdrop -->

        <!-- Modal -->
        <div class="modal-content">
            <h2>Welcome</h2>
            <div class="modal-body">
                <p>
                    You are about to participate in a messaging intervention that explores climate change and plastic pollution.
                </p>
                <p class="modal-subtitle">
                    Please take your time to read and engage with the content that will be presented.
                </p>
                <div v-if="loading" class="loader">Loading...</div>
            </div>
            <button @click="handleEnter" class="enter-button" :disabled="loading">
                <span v-if="loading" class="spinner"></span>
                <span v-else>Enter</span>
            </button>
        </div>
    </div>
</template>

<script setup>
import { defineProps, defineEmits, ref } from 'vue'

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

const emit = defineEmits(['enter'])
const loading = ref(false)

const handleEnter = () => {
    loading.value = true
    emit('enter')
    setTimeout(() => {
        loading.value = false
    }, 5000)
}
</script>

<style scoped>
.loader {
    margin: 20px 0;
    font-size: 18px;
    color: #000000;
}

.modal-backdrop {
    display: none;
}

.modal-content {
    max-width: 600px;
    margin: auto;
    padding: 30px;
    border-radius: 20px;
    font-family: "Space Grotesk", -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Arial, sans-serif;
    text-align: center;
    background: #ffffff;
}

.modal-content h2 {
    font-size: 32px;
    font-weight: 500;
    color: #000000;
    margin-bottom: 20px;
    letter-spacing: -0.02em;
}

.modal-body {
    margin-bottom: 30px;
}

.modal-body p {
    font-size: 18px;
    line-height: 1.6;
    color: #000000;
    margin-bottom: 15px;
}

.modal-subtitle {
    font-style: italic;
    opacity: 0.8;
    color: #000000;
}

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

.enter-button:hover {
    background: rgba(0, 0, 0, 0.1);
    color: #000000;
}

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

@keyframes spin {
    0% { transform: rotate(0deg); }
    100% { transform: rotate(360deg); }
}

</style> 
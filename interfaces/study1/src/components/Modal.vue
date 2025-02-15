<template>
    <!-- Modal container with full screen overlay -->
    <div v-if="show" class="h-screen flex flex-col justify-center items-center px-[30px] bg-white w-full">
        <!-- Content container with border and padding -->
        <div class="w-full max-w-[800px] border border-black bg-transparent p-[75px_50px] box-border">
            <!-- Modal title -->
            <h2 class="text-3xl font-normal mb-8 text-black">Welcome</h2>
            <div class="space-y-6">
                <!-- Dynamic welcome message based on group type -->
                <p class="font-normal text-base sm:text-lg md:text-xl text-black">
                    You are about {{ group?.startsWith('text_') ? 'to read a text snippet.' : 'to start a chat that you can interact with. Please note that your data is fully anonymized and used for research purposes only.' }}
                </p>
                <!-- Instructions for user engagement -->
                <p class="font-normal text-base sm:text-lg md:text-xl text-black italic opacity-80">
                    Please take your time to read and engage with the content.
                </p>
            </div>
            <!-- Action button to proceed -->
            <button 
                @click="$emit('enter')" 
                class="w-full mt-8 px-4 py-4 text-base sm:text-lg md:text-xl bg-black text-white font-normal hover:bg-opacity-90 transition-colors"
            >
                Enter
            </button>
        </div>
    </div>
</template>

<script setup>
// Importing Vue composition API utilities
import { defineProps, defineEmits } from 'vue'

// Component props definition
defineProps({
    // Controls modal visibility
    show: {
        type: Boolean,
        required: true
    },
    // Determines the type of content to be shown (text or chat)
    group: {
        type: String,
        default: ''
    }
})

// Event emission for modal entry
defineEmits(['enter'])
</script>

<style scoped>
/* Full viewport height utility class */
.h-screen {
    height: 100vh;
}

/* Animation for fade-in effect */
@keyframes fadeIn {
    from {
        opacity: 0;
        transform: translateY(20px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}

/* Apply fade-in animation to full-width elements */
.w-full {
    animation: fadeIn 0.5s forwards;
}
</style>

<template>
    <div v-if="show" class="h-screen flex flex-col justify-center items-center px-[30px] bg-white w-full">
        <div class="w-full max-w-[800px] border border-black bg-transparent p-[75px_50px] box-border">
            <h2 class="text-3xl font-normal mb-8 text-black">Welcome</h2>
            <div class="space-y-6">
                <p class="font-normal text-base sm:text-lg md:text-xl text-black">
                    You are about {{ group?.startsWith('text_') ? 'to read a text snippet.' : 'to start a chat that you can interact with. Please note that your data is fully anonymized and used for research purposes only.' }}
                </p>
                <p class="font-normal text-base sm:text-lg md:text-xl text-black italic opacity-80">
                    Please take your time to read and engage with the content.
                </p>
            </div>
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
import { defineProps, defineEmits } from 'vue'

defineProps({
    show: {
        type: Boolean,
        required: true
    },
    group: {
        type: String,
        default: ''
    }
})

defineEmits(['enter'])
</script>

<style scoped>
.h-screen {
    height: 100vh;
}

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

.w-full {
    animation: fadeIn 0.5s forwards;
}
</style>

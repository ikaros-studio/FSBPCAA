<template>
    <div class="h-screen flex flex-col justify-center items-center px-[30px] bg-white w-full">
        <div class="w-full max-w-[800px] border border-black bg-transparent p-[75px_50px] box-border flex flex-col">
            <p class="text-base sm:text-lg md:text-xl font-normal text-[#212121]">{{ group.content }}</p>
        </div>
    </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const props = defineProps({
    group: {
        type: Object,
        required: true
    }
})


</script>
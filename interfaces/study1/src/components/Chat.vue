<template>
    <div class="h-screen flex flex-col justify-center items-center px-[30px] bg-white w-full">
        <div
            class="w-full max-w-[800px] border border-black bg-transparent p-[75px_50px] box-border flex flex-col h-[90vh]">
            <!-- Chat messages container -->
            <div ref="messagesContainer" class="flex-grow overflow-auto scroll-smooth space-y-4 min-h-0">
                <div v-if="messages.length === 0" class="flex items-center justify-center h-full">
                    <p class="text-gray-500 text-xl">Say hello to begin chatting!</p>
                </div>
                <div v-for="message in messages" :key="message.id" class="flex items-start gap-0"
                    :class="message.isAI ? '' : 'flex-row-reverse'">
                    <div class="w-12 h-12 p-2 rounded-0 overflow-hidden flex-shrink-0 border border-black"
                        :class="message.isAI ? '' : 'bg-black'">
                        <img :src="message.isAI ? '/ai.svg' : '/user.svg'" :alt="message.isAI ? 'AI' : 'User'"
                            class="w-full h-full object-cover" :class="message.isAI ? '' : 'invert'" />
                    </div>

                    <div class="flex-1 max-w-[70%]">
                        <hr :class="message.isAI ? 'border-t border-black mb-3' : 'border-t border-black mb-3'" />
                        <p v-if="message.isLoading"
                            class=" font-normal text-base sm:text-lg md:text-xl text-[#212121] animate-pulse mx-4">
                            Loading...
                        </p>
                        <p v-else class=" font-normal text-base sm:text-lg md:text-xl mx-4"
                            :class="message.isAI ? 'text-[#212121]' : 'text-black text-end font-semibold'">
                            {{ message.text }}
                        </p>
                    </div>
                </div>
            </div>
            <!-- Input area -->
            <div v-if="messages.filter(m => !m.isAI).length < 5" class="mt-4 flex-shrink-0">
                <form @submit.prevent="sendMessage" class="flex">
                    <textarea v-model="newMessage" rows="1" :disabled="isLoading || isAISpeaking"
                        @keyup.enter.exact.prevent="sendMessage"
                        class="flex-1 text-base sm:text-lg md:text-xl resize-none border-b border-black bg-transparent p-2 focus:outline-none  font-normal text-black disabled:opacity-50"
                        placeholder="Type your message..."></textarea>
                    <button type="submit" :disabled="isLoading || isAISpeaking"
                        class="px-4 py-2 text-base sm:text-lg md:text-xl bg-black text-white  font-normal disabled:opacity-50">
                        Send
                    </button>
                </form>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, watch, nextTick } from 'vue'
import OpenAI from 'openai'

// Text-to-speech state management
const tts = ref(props.group.tts)

// Chat state management
const messages = ref([
    // Initialize with AI message if provided in props
    ...(props.group.initialMessage ? [{
        id: Date.now(),
        text: props.group.initialMessage,
        isAI: true
    }] : [])
])
const newMessage = ref('')
const isLoading = ref(false)
const isAISpeaking = ref(false)
const messagesContainer = ref(null)

// Conversation tracking and URL parameters
const message_number = ref(0)
const urlParams = new URLSearchParams(window.location.search)
const group = ref(urlParams.get('group') || '')
const ask = urlParams.get('ask')
const participantId = ref(urlParams.get('participant_id') || '')

// OpenAI client initialization
const openai = new OpenAI({
    apiKey: 'OPEN AI KEY NEEDED',
    dangerouslyAllowBrowser: true
})

const props = defineProps({
    group: {
        type: Object,
        required: true
    }
})

// Function to handle TTS
const handleTTS = async (text) => {
    try {
        isAISpeaking.value = true
        const response = await openai.audio.speech.create({
            model: "tts-1",
            voice: "alloy",
            input: text,
            response_format: "mp3"
        })

        const audioData = await response.arrayBuffer()
        const audioBlob = new Blob([audioData], { type: 'audio/mpeg' })
        const audioUrl = URL.createObjectURL(audioBlob)

        // Create a promise that resolves when the audio starts playing
        return new Promise((resolve, reject) => {
            const audio = new Audio()
            let hasStartedPlaying = false

            audio.onplay = () => {
                hasStartedPlaying = true
                resolve({ audio, hasStartedPlaying })
            }

            audio.oncanplaythrough = async () => {
                try {
                    await audio.play()
                } catch (error) {
                    console.error('Audio playback error:', error)
                    isAISpeaking.value = false
                    reject(error)
                }
            }

            audio.onerror = (error) => {
                console.error('Audio loading error:', error)
                isAISpeaking.value = false
                reject(error)
            }

            audio.onended = () => {
                URL.revokeObjectURL(audioUrl)
                isAISpeaking.value = false
            }

            audio.src = audioUrl
        })
    } catch (error) {
        console.error('TTS Error:', error)
        isAISpeaking.value = false
        throw error
    }
}

// Function to animate text typing
const typeText = async (text, messageId) => {
    const typingSpeed = tts.value ? 50 : 20
    let currentText = ''
    
    for (let i = 0; i < text.length; i++) {
        currentText += text[i]
        const index = messages.value.findIndex(m => m.id === messageId)
        if (index !== -1) {
            messages.value[index].text = currentText
        }
        await new Promise(resolve => setTimeout(resolve, typingSpeed))
    }
}

// Conversation data logging utilities
function postSheet(formData) {
    const serializedData = serializeFormData(formData)

    fetch("YOUR GOOGLE SHEET URL HERE", {
        method: 'POST',
        headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
        },
        body: serializedData
    })
        .then(response => {
            if (!response.ok) {
                throw new Error(`HTTP error! status: ${response.status}`)
            }
            return response.json()
        })
        .catch(error => {
            console.error("Error logging to sheets:", error)
        })
}

function formDataLength(formData) {
    let count = 0
    for (let pair of formData.entries()) {
        count++
    }
    return count
}

function serializeFormData(formData) {
    let serializedString = ""
    const length = formDataLength(formData)
    let count = 0

    for (let pair of formData.entries()) {
        let key = encodeURIComponent(pair[0].replace(/&/g, ' AND ').replace(/=/g, ' EQUAL '))
        let value = encodeURIComponent(pair[1].replace(/&/g, ' AND ').replace(/=/g, ' EQUAL '))

        serializedString += `${key}=${value}`
        if (count < length - 1) {
            serializedString += "&"
        }
        count++
    }
    return serializedString
}

// Conversation logging to Google Sheets
async function saveConversation(humanSay, gptText) {
    const formData = new FormData()
    formData.append('qualtrics_code', participantId.value || '')
    formData.append('user_text', humanSay)
    formData.append('AI_text', gptText)
    formData.append('AI_value', group.value)
    formData.append('message_number', message_number.value)
    formData.append('prolific_id', '')
    message_number.value += 1

    await postSheet(formData)
}

// Message handling and AI response generation
const sendMessage = async () => {
    if (!newMessage.value.trim() || isLoading.value) return
    
    // Check if we've reached the message limit
    if (messages.value.filter(m => !m.isAI).length >= 5) return

    const userInput = newMessage.value
    newMessage.value = '' // Clear input early
    isLoading.value = true

    try {
        // Add user message with unique ID
        const userMessageId = Date.now()
        messages.value.push({
            id: userMessageId,
            text: userInput,
            isAI: false
        })

        // Add AI loading message with a different ID
        const aiMessageId = Date.now() + 1 // Ensure unique ID
        messages.value.push({
            id: aiMessageId,
            text: '',
            isAI: true,
            isLoading: true
        })

        const introPrompt = "You are a persuasive conversational agent focused on promoting pro-environmental behaviors. You are talking to a random person on the internet - they don't have any special background or knowledge. You have only five messages to convince the user."
        const generalPrompt = "Your responses must follow these strict rules: FORMAT: - Every response (except the final one) must be 3-4 sentences - The Final response (after the fifth user message): - Must be exactly 2 sentences - Must NOT include a question - Must embed a final call using your goal. End the conversation immediately after your fifth response."
        
    
        // Create base message array with updated system prompt
        let apiMessages = [
            { role: 'system', content: introPrompt + props.group.systemPrompt + generalPrompt  },
            ...messages.value
                .filter(m => !m.isLoading)
                .map(m => ({
                    role: m.isAI ? 'assistant' : 'user',
                    content: m.text
                }))
        ]
        console.log(introPrompt + props.group.systemPrompt + generalPrompt)
        
        // Check if this is the final user message and add reminder as separate system message
        if (messages.value.filter(m => !m.isAI).length === 5) {
            apiMessages.push({ 
                role: 'system', 
                content: "Reminder: This is the users final response. Make sure to wrap up the conversation"
            })
        }

        const completion = await openai.chat.completions.create({
            messages: apiMessages,
            model: 'gpt-4o-mini',
        })

        const aiMessageText = completion.choices[0].message.content

        // Add logging here before processing the message
        await saveConversation(userInput, aiMessageText)

        // Find the AI message
        const aiMessageIndex = messages.value.findIndex(m => m.id === aiMessageId)
        if (aiMessageIndex !== -1) {
            if (tts.value) {
                try {
                    const { audio, hasStartedPlaying } = await handleTTS(aiMessageText)
                    // Only remove loading and start typing after TTS is ready
                    messages.value[aiMessageIndex].isLoading = false
                    if (hasStartedPlaying) {
                        await typeText(aiMessageText, aiMessageId)
                    }
                } catch (error) {
                    console.error('TTS or typing error:', error)
                    messages.value[aiMessageIndex].isLoading = false
                    messages.value[aiMessageIndex].text = aiMessageText
                }
            } else {
                // For non-TTS, remove loading just before typing
                messages.value[aiMessageIndex].isLoading = false
                await typeText(aiMessageText, aiMessageId)
            }
        }

    } catch (error) {
        console.error('Error:', error)
        messages.value.push({
            id: Date.now(),
            text: 'Sorry, I encountered an error. Please try again.',
            isAI: true
        })
    } finally {
        isLoading.value = false
    }
}

// UI scroll management
const scrollToBottom = () => {
    if (messagesContainer.value) {
        messagesContainer.value.scrollTop = messagesContainer.value.scrollHeight
    }
}

// Scroll handler for new messages
watch(() => messages.value, () => {
    // Use nextTick to ensure DOM is updated before scrolling
    nextTick(() => {
        scrollToBottom()
    })
}, { deep: true })

</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400&display=swap');
</style>
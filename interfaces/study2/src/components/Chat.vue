<template>
    <div class="chat-container">
        <div class="chat-section">
            <div v-if="!ready && condition.conversationMode !== 'static'" class="message seahorse">
                Loading...
            </div>
            <template v-else>
                <div v-if="props.condition.conversationMode == 'chat'" class="chat-header">
                    <h1>Chat with the {{ props.condition.name }}</h1>
                </div>
                <div v-if="props.condition.conversationMode == 'chat'" class="chat-messages" ref="chatMessages">
                    <div v-for="(message, index) in messages" :key="index"
                        :class="['message', message.isUser ? 'user' : 'seahorse', message.isTyping ? 'typing' : '']">
                        <template v-if="message.isTyping">
                            <div class="typing-dot"></div>
                            <div class="typing-dot"></div>
                            <div class="typing-dot"></div>
                        </template>
                        <template v-else>
                            {{ message.text }}
                        </template>
                    </div>
                </div>
                <div v-else class="static-content">
                    {{ props.condition.conversationContent }}
                </div>
                <div class="input-section"
                    v-if="props.condition.conversationMode == 'chat' && welcomeComplete && getUserMessageCount() < 3">
                    <button v-if="!chatStarted" class="talk-button" @click="startChat">
                        Talk to the {{ props.condition.name }}
                    </button>
                    <input v-else type="text" id="message-input" v-model="newMessage" @keyup.enter="handleSendMessage"
                        :disabled="isProcessing" placeholder="Type your message...">
                </div>
            </template>
        </div>
    </div>
</template>

<script setup>
import { ref, onMounted, nextTick, onUnmounted, watch } from 'vue'

const props = defineProps({
    condition: {
        type: Object,
        required: true
    },
    ready: {
        type: Boolean,
        default: false
    }
})

const emit = defineEmits(['talkingStateChange'])

const messages = ref([])
const welcomeMessage = ref(null)
const newMessage = ref('')
const chatMessages = ref(null)
const currentAudio = ref(null)
const isPlaying = ref(false)
const audioQueue = ref([])
const introComplete = ref(false)
const chatStarted = ref(false)
const welcomeComplete = ref(false)
const isProcessing = ref(false)

// Add these new refs for logging
const message_number = ref(0)
const urlParams = new URLSearchParams(window.location.search)
const group = ref(urlParams.get('group') || '')
const participantId = ref(urlParams.get('participant_id') || '')

const addMessage = async (text, isUser = false, isWelcome = false) => {
    if (isWelcome) {
        welcomeMessage.value = { text, isUser }
    } else {
        messages.value.push({ text, isUser })
    }
    await nextTick()
    if (chatMessages.value) {
        chatMessages.value.scrollTop = chatMessages.value.scrollHeight
    }
}

const addTypingMessage = async () => {
    messages.value.push({
        text: '',
        isUser: false,
        isTyping: true
    })
    await nextTick()
    if (chatMessages.value) {
        chatMessages.value.scrollTop = chatMessages.value.scrollHeight
    }
}

const cleanupAudio = () => {
    if (currentAudio.value) {
        currentAudio.value.pause()
        URL.revokeObjectURL(currentAudio.value.src)
        currentAudio.value = null
        isPlaying.value = false
    }
}

const playAudio = async (audioBlob) => {
    cleanupAudio()

    const audioUrl = URL.createObjectURL(audioBlob)
    currentAudio.value = new Audio(audioUrl)

    currentAudio.value.addEventListener('ended', () => {
        cleanupAudio()
        emit('talkingStateChange', false)
    })

    currentAudio.value.addEventListener('error', (e) => {
        console.error('Audio playback error:', e)
        cleanupAudio()
        emit('talkingStateChange', false)
    })

    try {
        await currentAudio.value.play()
        isPlaying.value = true
    } catch (error) {
        console.error('Failed to play audio:', error)
        cleanupAudio()
        emit('talkingStateChange', false)
    }
}

// Add this function to save conversations to Google Sheets
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

// Add helper functions for Google Sheets logging
function postSheet(formData) {

    console.log(formData)
    const serializedData = serializeFormData(formData)

    fetch("https://script.google.com/macros/s/AKfycbwN45pgOyilVoYYKeheFSFzlAEknyS1lfp87zeYwcPKfcIDkSfJfQnZgLt92_fH8klp/exec", {
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

const handleSendMessage = async () => {
    const text = newMessage.value.trim()
    if (text && !isProcessing.value) {
        isProcessing.value = true
        addMessage(text, true)
        newMessage.value = ''

        try {
            // Modified history building logic to exclude welcome message and its parts
            const welcomeText = props.condition.conversationContent || 'Welcome! I am your seahorse companion. Feel free to chat with me!'
            const history = [
                { role: 'system', content: props.condition.systemPrompt },
                // Only include messages that aren't typing indicators and aren't part of the welcome message
                ...messages.value
                    .filter(msg => !msg.isTyping && msg.text !== welcomeText && !welcomeText.includes(msg.text))
                    .map(msg => ({
                        role: msg.isUser ? 'user' : 'assistant',
                        content: msg.text
                    }))
            ]

            const response = await fetch('https://api.openai.com/v1/chat/completions', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': `OPEN AI KEY NEEDED`
                },
                body: JSON.stringify({
                    model: 'gpt-4o-mini',
                    messages: history
                })
            })

            const data = await response.json()
            if (data.choices && data.choices[0]) {
                const fullResponse = data.choices[0].message.content

                // Log the conversation to Google Sheets
                await saveConversation(text, fullResponse)

                const sentences = fullResponse.match(/[^.!?]+[.!?]+/g) || [fullResponse]

                for (const sentence of sentences) {
                    try {
                        addTypingMessage()
                        emit('talkingStateChange', false)

                        const ttsResponse = await fetch('https://api.openai.com/v1/audio/speech', {
                            method: 'POST',
                            headers: {
                                'Content-Type': 'application/json',
                                'Authorization': `OPEN AI KEY NEEDED`
                            },
                            body: JSON.stringify({
                                model: 'tts-1',
                                input: sentence.trim(),
                                voice: props.condition.voice.voice,
                            })
                        })

                        if (!ttsResponse.ok) {
                            throw new Error(`TTS API error: ${ttsResponse.status}`)
                        }

                        const audioBlob = await ttsResponse.blob()
                        emit('talkingStateChange', true)

                        // Play audio and type text simultaneously
                        const audioPromise = playAudio(audioBlob)
                        let displayedText = ''

                        // Clear typing animation before starting to type
                        messages.value[messages.value.length - 1] = { text: '', isUser: false }

                        for (const char of sentence) {
                            displayedText += char
                            messages.value[messages.value.length - 1].text = displayedText
                            if (chatMessages.value) {
                                chatMessages.value.scrollTop = chatMessages.value.scrollHeight
                            }
                            await new Promise(resolve => setTimeout(resolve, 60))
                        }

                        // Wait for audio to finish before proceeding to next sentence
                        await audioPromise
                    } catch (ttsError) {
                        console.error('TTS Error:', ttsError)
                        emit('talkingStateChange', false)
                    }
                }
            }
        } catch (error) {
            console.error('Error:', error)
            addMessage('Sorry, I encountered an error. Please try again later.')
            emit('talkingStateChange', false)
        } finally {
            isProcessing.value = false
        }
    }
}

const startChat = () => {
    chatStarted.value = true
}

const playTextWithTTS = async (text) => {
    const sentences = text.match(/[^.!?]+[.!?]+/g) || [text]

    for (const sentence of sentences) {
        try {
            addTypingMessage()
            emit('talkingStateChange', false)

            // Pre-fetch and prepare the audio before starting the typing animation
            const ttsResponse = await fetch('https://api.openai.com/v1/audio/speech', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': `OPEN AI KEY NEEDED`
                },
                body: JSON.stringify({
                    model: 'tts-1',
                    input: sentence.trim(),
                    voice: props.condition.voice.voice,
                })
            })

            if (!ttsResponse.ok) {
                throw new Error(`TTS API error: ${ttsResponse.status}`)
            }

            const audioBlob = await ttsResponse.blob()

            // Start playing audio immediately
            emit('talkingStateChange', true)
            await playAudio(audioBlob)

            // Clear typing animation and start typing
            messages.value[messages.value.length - 1] = { text: '', isUser: false }
            let displayedText = ''

            for (const char of sentence) {
                displayedText += char
                messages.value[messages.value.length - 1].text = displayedText
                if (chatMessages.value) {
                    chatMessages.value.scrollTop = chatMessages.value.scrollHeight
                }
                await new Promise(resolve => setTimeout(resolve, 60))
            }

        } catch (ttsError) {
            console.error('TTS Error:', ttsError)
            emit('talkingStateChange', false)
        }
    }
    welcomeComplete.value = true
}

// Modify the watch for the ready prop
watch(() => props.ready, async (isReady) => {
    if (isReady && props.condition.conversationMode !== 'static') {
        try {
            await new Promise(resolve => setTimeout(resolve, 500));

            const welcomeText = props.condition.conversationContent || 'Welcome! I am your seahorse companion. Feel free to chat with me!'
            await playTextWithTTS(welcomeText)
            addMessage(welcomeText, false, true)
            introComplete.value = true
        } catch (error) {
            console.error('Error playing welcome message:', error)
            addMessage(props.condition.conversationContent || 'Welcome! I am your seahorse companion. Feel free to chat with me!', false, true)
            introComplete.value = true
        }
    }
})

// Modify the onMounted hook
onMounted(() => {
    if (props.condition.conversationMode == 'static') {
        addMessage(props.condition.conversationContent)
    }
})

onUnmounted(() => {
    cleanupAudio()
})

// Add this function to count user messages
const getUserMessageCount = () => {
    return messages.value.filter(message => message.isUser).length
}
</script>

<style scoped>
.chat-container {
    display: -webkit-box;
    display: -ms-flexbox;
    display: flex;
    height: 90vh;
    margin: 0vh 0;
    padding: 10px;
    padding-right: 0px !important;
    max-width: 100%;
}

.chat-section {
    flex: 1;
    display: flex;
    flex-direction: column;
    background-color: rgb(255, 255, 255);
    /* Fallback for older browsers */
    background-color: rgba(255, 255, 255, 0.7);
    /* Modern browsers */
    border-radius: 20px;
    padding: 20px;
    font-family: "Space Grotesk", -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Arial, sans-serif;
    border: 1px solid #FFFFFF;
    box-shadow: 0 8px 32px 0 rgba(31, 38, 135, 0.37);
}

.chat-header {
    text-align: center;
    color: #333333;
    padding: 10px;
    margin-bottom: 20px;
    border-bottom: 2px solid #FFFFFF;
}

.chat-header h1 {
    font-size: 32px;
    font-weight: 500;
    letter-spacing: -0.02em;
}

.chat-messages {
    -webkit-box-flex: 1;
    -ms-flex: 1;
    flex: 1;
    overflow-y: auto;
    padding: 10px;
    display: -webkit-box;
    display: -ms-flexbox;
    display: flex;
    -webkit-box-orient: vertical;
    -webkit-box-direction: normal;
    -ms-flex-direction: column;
    flex-direction: column;
    gap: 15px;

    &>*+* {
        margin-top: 15px;
    }

    scrollbar-width: thin;
    scrollbar-color: rgba(255, 255, 255, 0.3) rgba(255, 255, 255, 0.1);
    -ms-overflow-style: -ms-autohiding-scrollbar;
}

.chat-messages::-webkit-scrollbar {
    width: 6px;
}

.chat-messages::-webkit-scrollbar-track {
    background: rgba(255, 255, 255, 0.1);
}

.chat-messages::-webkit-scrollbar-thumb {
    background: rgba(255, 255, 255, 0.3);
    border-radius: 3px;
}

.message {
    max-width: 80%;
    padding: 15px 20px;
    border-radius: 25px;
    margin: 5px 0;
    opacity: 0;
    transform: translateY(20px);
    animation: fadeIn 0.5s forwards;
    font-size: 18px;
    background: #FFFFFF;
    line-height: 1.4;
    color: #333;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
    border: 1px solid rgba(0, 0, 0, 0.05);
}

.message.user {
    align-self: flex-start;
    border-bottom-left-radius: 5px;
}

.message.seahorse {
    align-self: flex-end;
    border-bottom-right-radius: 5px;
}

.input-section {
    display: -webkit-box;
    display: -ms-flexbox;
    display: flex;
    gap: 10px;
    margin-left: 5px;
    margin-right: 5px;
    padding-top: 20px;
    padding-left: 0;
    padding-right: 0;
    margin-top: auto;
}

#message-input {
    width: 100%;
    padding: 15px 20px;
    color: #333333;
    background: white;
    border: 2px solid rgba(0, 0, 0, 0.1);
    border-radius: 30px;
    outline: none;
    font-family: "Space Grotesk", -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Arial, sans-serif;
    font-size: 18px;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
}

#message-input::-webkit-input-placeholder {
    color: #999999;
    -webkit-font-smoothing: antialiased;
}

#message-input::-moz-placeholder {
    color: #999999;
    opacity: 1;
}

#message-input:-ms-input-placeholder {
    color: #999999;
}

#message-input::-ms-input-placeholder {
    color: #999999;
}

#message-input::placeholder {
    color: #999999;
}

#message-input:disabled {
    opacity: 0.7;
    cursor: not-allowed;
}

.talk-button {
    width: 100%;
    padding: 15px 20px;
    margin: 0;
    color: #333333;
    background: white;
    border: 2px solid rgba(0, 0, 0, 0.1);
    border-radius: 30px;
    outline: none;
    font-family: "Space Grotesk", -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Arial, sans-serif;
    font-size: 18px;
    cursor: pointer;
    transition: all 0.3s ease;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
}

.talk-button:hover {
    transform: translateY(-1px);
    box-shadow: 0 6px 16px rgba(0, 0, 0, 0.12);
}

@media not all and (min-resolution:.001dpcm) {
    @supports (-webkit-appearance:none) {}
}

@-webkit-keyframes fadeIn {
    from {
        opacity: 0;
        -webkit-transform: translateY(20px);
        transform: translateY(20px);
    }

    to {
        opacity: 1;
        -webkit-transform: translateY(0);
        transform: translateY(0);
    }
}

@keyframes fadeIn {
    from {
        opacity: 0;
        -webkit-transform: translateY(20px);
        transform: translateY(20px);
    }

    to {
        opacity: 1;
        -webkit-transform: translateY(0);
        transform: translateY(0);
    }
}

@media (max-width: 768px) {
    .chat-container {
        margin: 0;
        height: 100vh;
        padding: 5px;
    }

    .chat-section {
        padding: 10px;
    }

    .message {
        max-width: 100%;
        padding: 10px 15px;
        font-size: 14px;
    }

    .chat-header h1 {
        font-size: 24px;
    }

    #message-input {
        padding: 10px 15px;
        font-size: 14px;
    }

    .talk-button {
        padding: 10px 15px;
        font-size: 14px;
    }

    .static-content {
        padding: 10px;
        font-size: 14px !important;
        line-height: 1.4;
    }
}

.static-content {
    flex: 1;
    padding: 20px;
    color: #333333;
    font-size: 18px;
    line-height: 1.6;
    overflow-y: auto;
}

.audio-controls {
    display: flex;
    gap: 10px;
    justify-content: center;
    margin-top: 10px;
}

.audio-controls button {
    padding: 8px 16px;
    border-radius: 20px;
    background: rgba(255, 255, 255, 0.2);
    border: 1px solid rgba(255, 255, 255, 0.4);
    color: white;
    cursor: pointer;
    transition: all 0.3s ease;
}

.audio-controls button:disabled {
    opacity: 0.5;
    cursor: not-allowed;
}

.audio-controls button:hover:not(:disabled) {
    background: rgba(255, 255, 255, 0.3);
}

.message.typing {
    display: flex;
    align-items: center;
    gap: 4px;
}

.typing-dot {
    width: 8px;
    height: 8px;
    background: #333333;
    border-radius: 50%;
    animation: typingAnimation 1s infinite;
}

.typing-dot:nth-child(2) {
    animation-delay: 0.2s;
}

.typing-dot:nth-child(3) {
    animation-delay: 0.4s;
}

@keyframes typingAnimation {

    0%,
    100% {
        transform: translateY(0);
        opacity: 0.3;
    }

    50% {
        transform: translateY(-4px);
        opacity: 1;
    }
}

&::-webkit-scrollbar {
    width: 6px;
}

&::-webkit-scrollbar-track {
    background: rgba(255, 255, 255, 0.1);
}

&::-webkit-scrollbar-thumb {
    background: rgba(255, 255, 255, 0.3);
    border-radius: 3px;
}

scrollbar-width: thin;
scrollbar-color: rgba(255, 255, 255, 0.3) rgba(255, 255, 255, 0.1);
</style>

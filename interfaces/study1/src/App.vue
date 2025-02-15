<script setup>
import { ref, onMounted } from 'vue';
import Chat from './components/Chat.vue';
import Text from './components/Text.vue';
import Modal from './components/Modal.vue';
import conditions from './assets/conditions.js'

const currentComponent = ref(null);
const componentProps = ref(null);
const showModal = ref(true);
const hasEnteredApp = ref(false);
const urlParams = new URLSearchParams(window.location.search);

onMounted(() => {
  const group = urlParams.get('group');
  
  if (group && conditions[group]) {
    componentProps.value = conditions[group];
    
    if (group.startsWith('chat_')) {
      currentComponent.value = Chat;
    } else if (group.startsWith('text_')) {
      currentComponent.value = Text;
    }
  } else {
    console.log(`No condition found for group: ${group}`);
  }
});

const handleEnterApp = () => {
  showModal.value = false;
  hasEnteredApp.value = true;
};

const getConditionGroups = () => {
  return Object.keys(conditions).reduce((acc, key) => {
    const type = key.startsWith('chat_') ? 'Chat' : 
                 key.startsWith('text_') ? 'Text' : 'Other';
    acc[type] = acc[type] || [];
    acc[type].push({
      key,
      name: conditions[key].name || key
    });
    return acc;
  }, {});
};
</script>

<template>
  <Modal 
    v-if="!hasEnteredApp" 
    :show="showModal"
    @enter="handleEnterApp"
    :group="urlParams.get('group')"
  />
  
  <div v-else class="min-h-screen p-4">
    <main class="wrapper">
      <component 
        :is="currentComponent" 
        v-if="currentComponent && componentProps"
        :group="componentProps" 
      />
      
      <div v-else class="max-w-md mx-auto mt-8">
        <h2 class="text-xl font-bold mb-4">Select a condition:</h2>
        <div v-for="(items, type) in getConditionGroups()" :key="type" class="mb-6">
          <h3 class="font-semibold mb-2">{{ type }}</h3>
          <ul class="space-y-2">
            <li v-for="item in items" :key="item.key">
              <a 
                :href="`?group=${item.key}`"
                class="text-blue-600 hover:underline"
              >
                {{ item.name }}
              </a>
            </li>
          </ul>
        </div>
      </div>
    </main>
  </div>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@400&display=swap');

body {
  font-family: 'Space Grotesk', sans-serif !important;
}
</style>

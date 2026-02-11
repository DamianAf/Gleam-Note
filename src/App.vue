<script setup lang="ts">
import { computed, reactive, } from 'vue';
import Editor from './Editor.vue';
import SideBar from './SideBar.vue';

/*
  {
    active: number,
    tabs: [{
    title: string,
    content: string
    }]
  }
*/

const json = reactive({
  active: -1,
  tabs: []
});

const activeTabContent = computed({
  get() {
    return json.tabs[json.active]?.content ?? '';
  },
  set(value: string) {
    if (json.tabs[json.active]) {
      json.tabs[json.active].content = value;
    }
  }
});
const amount = computed(() => json.tabs.length);
</script>

<template>
<main class="flex">
<SideBar :model-value="json" @update:model-value="(val) => Object.assign(json, val)" />
<Editor v-if="amount > 0" v-model="activeTabContent" :key=json.active />
</main>
</template>

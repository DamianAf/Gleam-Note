<script setup lang="ts">
import { computed, reactive, } from 'vue';
import Editor from './Editor.vue';
import SideBar from './SideBar.vue';

/*
  {
    amount: number,
    actove: number,
    id_{num}: {
      title: string,
      content: string,
    }
  }
*/

const json = reactive({
  active: 0,
  tabs: [{
    title: "Tony Stark afsdfasdfasdfasdfasdfasdfasdfasd fasdf sdasfd asdfasdf  asfdasdfasdf  adfas",
    content: `# h1 element
      ---
      - [ ] testing
      $x^2 + \\int x^4 / 2 = 1$
      \`\`\`c
      int i = 3
      \`\`\``,
    }, {
    title: "Batman",
    content: `# h1 element
      ---
      - [ ] asdf
      $x^2 + \\int x^4 / 2 = 1$
      \`\`\`c
      int i = 2
      \`\`\``,
    }, {
    title: "Batman 2",
    content: `# h1 element
      ---
      - [ ] testasdfasdfasdfing
      $x^2 + \\int x^4 / 2 = 1$
      \`\`\`c
      int i = 2
      \`\`\``,
  }],
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

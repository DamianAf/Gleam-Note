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
  amount: 3,
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

const activeContent = computed(() => json.active);

</script>

<template>
<main class="flex">
<SideBar :model-value="json" @update:model-value="(val) => Object.assign(json, val)" />
<Editor v-model="json.tabs[activeContent].content" :key=json.active />
</main>
</template>

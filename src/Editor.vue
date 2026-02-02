<!--
# hello
---
- first item
- second item
- [ ] first task
- [ ] second task

$x^2 + \int x^4 / 2 = 1$

```c
int i = 1;
void main() {
 std::cout<< "stuff";
}
```
-->

<script setup lang="ts">
import { computed, ComputedRef, Ref, ref } from "vue";
import MarkdownIt from "markdown-it";
import markdownItKatex from "@vscode/markdown-it-katex";
import { tasklist } from "@mdit/plugin-tasklist";
import hljs from 'highlight.js';
import 'highlight.js/styles/github-dark.css';

import javascript from 'highlight.js/lib/languages/javascript';
import cpp from 'highlight.js/lib/languages/cpp'
import c from 'highlight.js/lib/languages/c'
import python from 'highlight.js/lib/languages/python';
import rust from 'highlight.js/lib/languages/rust';
import bash from 'highlight.js/lib/languages/bash';
import json from 'highlight.js/lib/languages/json';

hljs.registerLanguage('javascript', javascript);
hljs.registerLanguage('cpp', cpp);
hljs.registerLanguage('c', c);
hljs.registerLanguage('js', javascript);
hljs.registerLanguage('python', python);
hljs.registerLanguage('rust', rust);
hljs.registerLanguage('bash', bash);
hljs.registerLanguage('json', json);

const showPreview: Ref<boolean, boolean> = ref(false);

const props = defineProps<{
  modelValue: string;
}>();

const emit = defineEmits<{
  (e: 'update:modelValue', value: string): void;
}>();

const md: MarkdownIt = MarkdownIt({
  html: false,
  linkify: true,
  typographer: true,
  breaks: true,
  highlight: (str, lang) => {
    if (lang && hljs.getLanguage(lang)) {
        return hljs.highlight(str, { language: lang }).value;
    }
    // No language or unknown → escape HTML
    return md.utils.escapeHtml(str);
  }
})
.use(tasklist)
.use(markdownItKatex, {
  throwOnError: false,
  errorColor: "#ff6666",
});
const content: Ref<string, string> = ref(props.modelValue);

const rendered: ComputedRef<string> = computed(() => {
  let html: string = md.render(props.modelValue);
  html = enableCheckboxes(html);
  return html;
})

function enableCheckboxes(html: string): string {

  return html.replace(
    /<input([^>]*?)type="checkbox"([^>]*?)disabled([^>]*?)>/g,
    '<input$1type="checkbox"$2$3>'
  );
}

const handleCheckboxClick = (e: MouseEvent) => {
  const input = e.target as HTMLInputElement;
  if (!input || input.type !== 'checkbox') return;

  // Find the <li> and get its index
  const li = input.closest('li.task-list-item');
  if (!li) return;

  const ul = li.closest('ul');
  if (!ul) return;

  const taskItems = Array.from(ul.querySelectorAll('li.task-list-item'));
  const itemIndex = taskItems.indexOf(li);

  const lines = content.value.split('\n');
  let taskLineIndex = -1;
  let taskCount = 0;

  for (let i = 0; i < lines.length; i++) {
    if (/^\s*[-*+] \[.\]/.test(lines[i])) {
      if (taskCount === itemIndex) {
        taskLineIndex = i;
        break;
      }
      taskCount++;
    }
  }

  if (taskLineIndex === -1) return;

  // Toggle completion
  const line = lines[taskLineIndex];
  if (line.includes('[ ]')) {
    lines[taskLineIndex] = line.replace('[ ]', '[x]');
  } else if (line.includes('[x]') || line.includes('[X]')) {
    lines[taskLineIndex] = line.replace(/\[.\]/, '[ ]');
  }

  // Update content
  content.value = lines.join('\n');
};

const changeView = () => {
  showPreview.value = !showPreview.value;
}

function onContentChange(newContent: string) {
  emit('update:modelValue', newContent);
}
</script>
<template>
  <div class="w-full flex relative">
    <textarea :class="{'editor-half': showPreview }" name="editor" id="editor" v-model="props.modelValue" @input="onContentChange(($event.target as HTMLTextAreaElement).value)" class="absolute editor textarea textarea-primary bg-black text-white w-full resize-none focus:outline-none h-svh p-2.5" />
    <Transition name="slide">
        <div v-if="showPreview" id="preview" class="absolute right-0 textarea textarea-secondary bg-black text-white w-[50%] h-svh p-2.5 overflow-x-hidden" @click="handleCheckboxClick" v-html="rendered"></div>
    </Transition>
  </div>

  <label class="swap swap-rotate absolute right-4 top-4 border-2 rounded-full">
    <input @click="changeView" type="checkbox" class="theme-controller" value="synthwave" />
    <svg class="swap-on h-10 w-10 fill-current" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 640 640"><!--!Font Awesome Free v7.1.0 by @fontawesome - https://fontawesome.com License - https://fontawesome.com/license/free Copyright 2026 Fonticons, Inc.--><path d="M504.6 148.5C515.9 134.9 514.1 114.7 500.5 103.4C486.9 92.1 466.7 93.9 455.4 107.5L320 270L184.6 107.5C173.3 93.9 153.1 92.1 139.5 103.4C125.9 114.7 124.1 134.9 135.4 148.5L278.3 320L135.4 491.5C124.1 505.1 125.9 525.3 139.5 536.6C153.1 547.9 173.3 546.1 184.6 532.5L320 370L455.4 532.5C466.7 546.1 486.9 547.9 500.5 536.6C514.1 525.3 515.9 505.1 504.6 491.5L361.7 320L504.6 148.5z"/></svg>

    <svg class="swap-off h-10 w-10 fill-current"
    xmlns="http://www.w3.org/2000/svg" viewBox="0 0 640 640"><!--!Font Awesome Free v7.1.0 by @fontawesome - https://fontawesome.com License - https://fontawesome.com/license/free Copyright 2026 Fonticons, Inc.--><path d="M480 576L192 576C139 576 96 533 96 480L96 160C96 107 139 64 192 64L496 64C522.5 64 544 85.5 544 112L544 400C544 420.9 530.6 438.7 512 445.3L512 512C529.7 512 544 526.3 544 544C544 561.7 529.7 576 512 576L480 576zM192 448C174.3 448 160 462.3 160 480C160 497.7 174.3 512 192 512L448 512L448 448L192 448zM224 216C224 229.3 234.7 240 248 240L424 240C437.3 240 448 229.3 448 216C448 202.7 437.3 192 424 192L248 192C234.7 192 224 202.7 224 216zM248 288C234.7 288 224 298.7 224 312C224 325.3 234.7 336 248 336L424 336C437.3 336 448 325.3 448 312C448 298.7 437.3 288 424 288L248 288z"/></svg>
  </label>
</template>

<style scoped>

 /* Animations */

 .editor {
   width: 100%;
   transition: width 0.4s ease; /* 👈 this animates the width */
 }

 .editor.editor-half {
   width: 50%;
 }

 .slide-enter-active {
   transition: all 0.4s ease;
 }

 .slide-leave-active {
   transition: all 0.4s ease;
 }

 .slide-enter-from,
 .slide-leave-to {
   transform: translateX(100%);
 }

 /* Hide bullet for task list items */
:deep(li.task-list-item) {
  list-style: none;
  margin: 10px;
}

/* Change the checkmarks color and size */
:deep(li.task-list-item input[type="checkbox"]) {
  accent-color: green;
  transform: scale(2.5);
  margin-right: 0.75rem;
}

:deep(pre) {
  background-color: #303030;
  padding: 1rem;
  border-radius: 6px;
  overflow-x: auto;
  margin: 1rem 0;
}

:deep(.katex-display) {
  background-color: #505050;
  padding: 1rem;
  border-radius: 6px;
  margin: 1.5rem 0;
  display: block;
  overflow-x: auto;
}
:deep(.katex) {
    background-color: #505050;
    padding: 1rem;
    border-radius: 6px;
    margin: 1.5rem 0;
    display: block;
    overflow-x: auto;
}

</style>

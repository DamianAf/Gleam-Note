<!--
# hello
---
- first item
- second item
- [ ] first task
- [ ] second task

$x^2 + \int x^4 / 2 = 1$

```
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

const showRawMarkdown: Ref<boolean, boolean> = ref(true);

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
const content: Ref<string, string> = ref('');

const rendered: ComputedRef<string> = computed(() => {
  let html: string = md.render(content.value);
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
  showRawMarkdown.value = !showRawMarkdown.value;
}
</script>
<template>
<textarea v-if="showRawMarkdown" name="editor" id="editor" v-model="content" class="bg-black text-white w-full resize-none focus:outline-none h-svh p-2.5" />
<div v-else class=" bg-black text-white w-full h-svh p-2.5" @click="handleCheckboxClick" v-html="rendered"></div>
<button class="cursor-pointer" @click="changeView">Show Preview</button>
</template>

<style scoped>
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

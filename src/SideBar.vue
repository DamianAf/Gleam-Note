<script setup lang="ts">
import { computed, ref } from 'vue';
import { open, save } from '@tauri-apps/plugin-dialog';
import { readTextFile, writeTextFile } from '@tauri-apps/plugin-fs';

const activeTabTitle = computed(() => { return props.modelValue.tabs[props.modelValue.active]?.title || '';});
const props = defineProps<{
  modelValue: any;
}>();

const emit = defineEmits<{
  (e: 'update:modelValue', value: JSON): void;
}>();

function updateJSONProp(newContent: JSON) {
  emit('update:modelValue', newContent);
}

function setActive(e: Event) {

  let tempObj = { ...props.modelValue };
  tempObj.active = Number(e.target.value);
  updateJSONProp(tempObj);
}

function changeTitle() {
  let tempObj = { ...props.modelValue };

  let input = document.getElementById("rename") as HTMLInputElement;
  tempObj.tabs[tempObj.active].title = String(input.value);

  updateJSONProp(tempObj);
}

function addFile() {
  let tempObj = { ...props.modelValue };

  let input = document.getElementById("add") as HTMLInputElement;

  tempObj.tabs.push({
    title: String(input.value || "Untitled"),
    content: ""
  });

  tempObj.active = tempObj.tabs.length - 1;

  input.value = "";

  updateJSONProp(tempObj);
}

function focusOnModalInput() {
  let input = document.getElementById("add") as HTMLInputElement;
  input.focus();
}

function handleDeletion() {
  let tempObj = { ...props.modelValue };
  tempObj.tabs.splice(tempObj.active, 1);
  tempObj.active = -1;
  updateJSONProp(tempObj);
}

async function openProject() {
  const file = await open({
    multiple: false,
    directory: false,
    filters: [{ name: "All Files", extensions: ["txt"] }]
  });
  try {
    const content = await readTextFile(String(file));
    updateJSONProp(JSON.parse(content));
    const dropdown = document.getElementById("dropdownMenu") as HTMLDetailsElement;
    dropdown.open = false;
  } catch(err) {
    console.log("Err: ", err);
  }
}

async function saveProject() {
  const filePath = await save({
    filters: [{ name: "Text Files", extensions: ["txt"] }]
  });

  if (filePath) {
    try {
      await writeTextFile(filePath, JSON.stringify(props.modelValue));
      console.log("File saved successfully!");
    } catch (err) {
      console.error("Failed to write file:", err);
    }
  }
}

</script>

<template>
<div class="flex flex-col">
    <div class="join">

        <details id="dropdownMenu" class="dropdown border-sky-500">
        <summary class="btn join-item rounded-full p-2 border-base-100 border-2 hover:border-slate-50">
        <svg  class="w-5 fill-white" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 640 640">
            <path d="M96 160C96 142.3 110.3 128 128 128L512 128C529.7 128 544 142.3 544 160C544 177.7 529.7 192 512 192L128 192C110.3 192 96 177.7 96 160zM96 320C96 302.3 110.3 288 128 288L512 288C529.7 288 544 302.3 544 320C544 337.7 529.7 352 512 352L128 352C110.3 352 96 337.7 96 320zM544 480C544 497.7 529.7 512 512 512L128 512C110.3 512 96 497.7 96 480C96 462.3 110.3 448 128 448L512 448C529.7 448 544 462.3 544 480z"/>
        </svg>
        </summary>
        <ul tabindex="0" class="menu dropdown-content bg-base-100 rounded-box z-1 w-52 p-2 shadow-sm" style="list-style-type: none; border: 1px solid gray; margin-left: 2px; padding-left: 5px;">
        <li><a @click=openProject() >Open Project</a></li>
        <li><a @click=saveProject() >Save Project</a></li>
        </ul>
        </details>

        <button onclick="renameModal.showModal()" class="join-item btn rounded-full p-2 border-base-100 border-2 hover:border-slate-50"><svg class="w-5 fill-white" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 640 640"><!--!Font Awesome Free v7.1.0 by @fontawesome - https://fontawesome.com License - https://fontawesome.com/license/free Copyright 2026 Fonticons, Inc.--><path d="M100.4 417.2C104.5 402.6 112.2 389.3 123 378.5L304.2 197.3L338.1 163.4C354.7 180 389.4 214.7 442.1 267.4L476 301.3L442.1 335.2L260.9 516.4C250.2 527.1 236.8 534.9 222.2 539L94.4 574.6C86.1 576.9 77.1 574.6 71 568.4C64.9 562.2 62.6 553.3 64.9 545L100.4 417.2zM156 413.5C151.6 418.2 148.4 423.9 146.7 430.1L122.6 517L209.5 492.9C215.9 491.1 221.7 487.8 226.5 483.2L155.9 413.5zM510 267.4C493.4 250.8 458.7 216.1 406 163.4L372 129.5C398.5 103 413.4 88.1 416.9 84.6C430.4 71 448.8 63.4 468 63.4C487.2 63.4 505.6 71 519.1 84.6L554.8 120.3C568.4 133.9 576 152.3 576 171.4C576 190.5 568.4 209 554.8 222.5C551.3 226 536.4 240.9 509.9 267.4z"/></svg></button>

        <button onclick="confirmationModal.showModal()" class="join-item btn rounded-full p-2 border-base-100 border-2 hover:border-slate-50"><svg class="w-5 fill-white" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 640 640"><!--!Font Awesome Free v7.1.0 by @fontawesome - https://fontawesome.com License - https://fontawesome.com/license/free Copyright 2026 Fonticons, Inc.--><path d="M232.7 69.9L224 96L128 96C110.3 96 96 110.3 96 128C96 145.7 110.3 160 128 160L512 160C529.7 160 544 145.7 544 128C544 110.3 529.7 96 512 96L416 96L407.3 69.9C402.9 56.8 390.7 48 376.9 48L263.1 48C249.3 48 237.1 56.8 232.7 69.9zM512 208L128 208L149.1 531.1C150.7 556.4 171.7 576 197 576L443 576C468.3 576 489.3 556.4 490.9 531.1L512 208z"/></svg></button>
    </div>
    <hr class="text-gray-500">
    <div :key=activeTabTitle>
        <div class="tabs tabs-box flex-col w-70" v-for="(tab, index) in props.modelValue.tabs" :key="index">
            <input
            type="radio"
            :aria-label="tab.title"
            :checked="props.modelValue.active === index"
            :value="index"
            @change="setActive"
            name="tabs"
            class="tab w-full"
            />
        </div>
        <form method="dialog" @submit=focusOnModalInput>
            <button class="tab w-full" onclick="addFileModal.showModal()" type="submit"><span class="text-2xl">+</span></button>
        </form>
    </div>
</div>
<!-- Modal for rename a tab -->
<dialog id="renameModal" class="modal" >
  <div class="modal-box">
      <h1 class="text-center">Rename <span class="truncate max-w-100 inline-block align-top text-cyan-600">{{activeTabTitle}}</span></h1>
      <form method="dialog" @submit=changeTitle class="flex flex-col items-center">
        <label class="input flex flex-row items-baseline w-[80%]">
            <svg class="h-[1em] opacity-50" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
                <g
                stroke-linejoin="round"
                stroke-linecap="round"
                stroke-width="2.5"
                fill="none"
                stroke="currentColor"
                >
                <path d="M15 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V7Z"></path>
                <path d="M14 2v4a2 2 0 0 0 2 2h4"></path>
                </g>
            </svg>
            <input type="text" id="rename" class="grow w-full border-slate-50 p-2 mb-4" :value="activeTabTitle" />
        </label>
        <button class="btn w-full mt-5" type="submit">rename</button>
      </form>
  </div>
  <form method="dialog" class="modal-backdrop">
    <button>close</button>
  </form>
</dialog>
<!-- Modal for adding a new file -->
<dialog id="addFileModal" class="modal" >
  <div class="modal-box">
      <h1 class="text-center">Add New File</h1>
      <form method="dialog" @submit=addFile class="flex flex-col items-center">
        <label class="input flex flex-row items-baseline w-[80%]">
            <svg class="h-[1em] opacity-50" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
                <g
                stroke-linejoin="round"
                stroke-linecap="round"
                stroke-width="2.5"
                fill="none"
                stroke="currentColor"
                >
                <path d="M15 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V7Z"></path>
                <path d="M14 2v4a2 2 0 0 0 2 2h4"></path>
                </g>
            </svg>
            <input type="text" id="add" class="grow" placeholder="New File Title" />
        </label>
        <button class="btn w-full mt-5" type="submit">Add</button>
      </form>
  </div>
  <form method="dialog" class="modal-backdrop">
    <button>close</button>
  </form>
</dialog>
<!-- Modal for confirming deletion of a file -->
<dialog id="confirmationModal" class="modal" >
  <div class="modal-box">
      <h1 class="text-center">Are you sure you want to delete <span class="text-cyan-600">{{activeTabTitle}}</span>?</h1>
      <form method="dialog" @submit="" class="flex flex-col items-center">
        <button class="btn w-full mt-5" @click=handleDeletion()>OK</button>
        <button class="btn w-full mt-5" type="submit">Cancel</button>
      </form>
  </div>
  <form method="dialog" class="modal-backdrop">
    <button>close</button>
  </form>
</dialog>
</template>

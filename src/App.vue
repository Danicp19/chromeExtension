<script setup lang="ts">
import { ref, Ref, watch, reactive } from "vue";
import uniqid from "uniqid";
import IconCheckboxBlanckOutline from "~icons/mdi/check-box-outline-blank";
import IconEdit from "~icons/mdi/edit-outline";
import IconTrash from "~icons/mdi/trash";
import draggable from "vuedraggable";
const dragging = ref(false);
const defaultData = {
  columns: [
    {
      name: "now",
      list: [],
    },
    {
      name: "next",
      list: [],
    },
    {
      name: "later",
      list: [],
    },
  ],
} as { columns: Column[] };
const localStorageString = localStorage.getItem("todo");
const localStorageData = localStorageString
  ? JSON.parse(localStorageString)
  : defaultData;
const data = reactive(localStorageData);
watch(data, (newData) => {
  localStorage.setItem("todo", JSON.stringify(newData));
});
interface Column {
  name: string;
  list: {
    id: string;
    description: string;
    state: "new" | "done";
    editing?:boolean
  }[];
}
console.log(uniqid());

function addItem(event: Event, column: Column) {
  const form = event.target as HTMLFormElement;
  const formData = new FormData(form);
  const description = formData.get("description") as string;
  column.list.push({ id: uniqid(), description, state: "new" });
  form.reset();
}

function deleteItem(column: Column, listIndex: number) {
  column.list.splice(listIndex, 1);
}
function editItem(event: Event, column: Column, listIndex: number) {
  const form = event.target as HTMLFormElement;
  const formData = new FormData(form);
  const description = formData.get("description") as string;

  column.list[listIndex].description = description;
  column.list[listIndex].editing = false;
}
</script>

<template>
  <div class="flex flex-row justify-center space-x-6 pt-6 p-4 ">
    <div
      v-for="column of data.columns"
      :key="column.name"
      class="rounded.md shadow-md bg-slate-100 px-4 w-1/3 space-y-4"
    >
      <div class="mb-3">
        {{ column.name }}
      </div>
      <draggable
        v-model="column.list"
        group="items"
        item-key="id"
        class="space-y-4"
        :class="{ 'min-h-[40px] bg-slate-200 rounded-md ': dragging }"
        ghost-class="ghost"
        @start="dragging = true"
        @end="dragging = true"
      >
        <template #item="{ element: item, index: listIndex }">
          <div
            class="group rounded.md shadow-md bg-slate-50 py-2 px-3 flex flex-row space-x-2 items-start relative"
          >
            <template v-if="item.editing">
              <form
                v-if="item.editing"
                @submit.prevent="editItem($event, column, listIndex)"
                class="flex flex-col w-full"
              >
                <input
                  type="text"
                  name="description"
                  class="w-full px-3 py-2 rounded-md mb-2"
                  autocomplete="off"
                  :value="item.description"
                />
                <div class="flex flex-row justify-between">
                  <button type="button" @click="item.editing = false">Cancel</button>
                  <button>Save</button>
                </div>
              </form>
            </template>
            <template v-else>
              <div
                class="absolute hidden group-hover:flex bottom-0 right-0 p-2 bg-slate-50 opacity-90 max-h-full space-x-3 flex flex-row"
              >
                <button @click="item.editing = true">
               <IconEdit/>
                </button>
                <button @click="deleteItem(column, listIndex)">
                  <IconTrash class="text-red-700" />
                </button>
              </div>
              •
              <!-- <button @click="item.state = item.state === 'done' ? 'new' : 'done'">
              <IconCheckboxBlanckOutline v-if="item.state !== 'done'" />
              <IconCheckboxOutline v-else />
            </button> -->
              <span
                class="inline-block breack-words min-w-0"
                :class="{ 'line-through': item.state === 'done' }"
                >{{ item.description }}</span
              >
            </template>
          </div>
        </template>
      </draggable>

      <div class="pt-4">
        <form @submit.prevent="addItem($event, column)" class="flex flex-col items-end">
          <input
            type="text"
            name="description"
            class="w-full px-3 py-2 rounded-md mb-2"
            autocomplete="off"
          />
          <button>Add</button>
        </form>
      </div>
    </div>
  </div>
</template>
<style scoped>
.ghost {
  @apply bg-slate-400 opacity-30;
}
</style>

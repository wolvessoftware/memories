<script lang="ts" setup>
definePageMeta({
  layout: "dashboard",
  middleware: "auth",
});

import { PhotoIcon } from "@heroicons/vue/20/solid";

const client = useSupabaseClient();
const { data: memories } = await useAsyncData("memories", async () => {
  const { data } = await client.from("memories").select("name, id");
  return data;
});

const createMemory = () => {
  navigateTo("/memories");
};
</script>

<template>
  <div
    class="border-b border-gray-200 pb-5 sm:flex sm:items-center sm:justify-between"
  >
    <h3 class="text-lg font-medium leading-6 text-gray-900">Memories</h3>
    <div class="mt-3 sm:mt-0 sm:ml-4">
      <button
        type="button"
        class="inline-flex items-center rounded-md border border-transparent bg-indigo-600 px-4 py-2 text-sm font-medium text-white shadow-sm hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2"
        @click="createMemory"
      >
        Create memory
      </button>
    </div>
  </div>

  <ul role="list" class="grid grid-cols-1 gap-6 sm:grid-cols-2 lg:grid-cols-3">
    <li
      v-for="memory in memories"
      :key="memory.name"
      class="col-span-1 divide-y divide-gray-200 rounded-lg bg-white shadow"
    >
      <div class="flex w-full items-center justify-between space-x-6 p-6">
        <div class="flex-1 truncate">
          <div class="flex items-center space-x-3">
            <h3 class="truncate text-sm font-medium text-gray-900">
              {{ memory.name }}
            </h3>
          </div>
        </div>
      </div>
      <div>
        <div class="-mt-px flex divide-x divide-gray-200">
          <div class="flex w-0 flex-1">
            <NuxtLink
              :to="`/memory/${memory.id}`"
              class="relative -mr-px inline-flex w-0 flex-1 items-center justify-center rounded-bl-lg border border-transparent py-4 text-sm font-medium text-gray-700 hover:text-gray-500"
            >
              <PhotoIcon class="h-5 w-5 text-gray-400" aria-hidden="true" />
              <span class="ml-3">See memory</span>
            </NuxtLink>
          </div>
        </div>
      </div>
    </li>
  </ul>
</template>

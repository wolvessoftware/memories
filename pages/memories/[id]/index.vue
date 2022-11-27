<script lang="ts" setup>
import { PhotoIcon } from "@heroicons/vue/20/solid";
import type { Database } from "~/types";

definePageMeta({
  layout: "dashboard",
  middleware: "auth",
});

const user = useSupabaseUser();
const client = useSupabaseClient<Database>();
const route = useRoute();

const { id } = route.params;
const polaroids = ref([]);
const openSidebar = ref(false);

const { data: memory, refresh } = await useAsyncData("memories", async () => {
  const { data } = await client
    .from("memories")
    .select("name, owner, polaroids (path)")
    .eq("id", id);
  if (!data?.length) return;
  return data[0];
});

const getImageUrls = async () => {
  if (!memory || !memory.value || !memory.value.polaroids) return;
  const { data, error } = await client.storage
    .from("polaroids")
    .createSignedUrls(
      memory.value.polaroids.map((p) => p.path),
      60
    );
  if (error) return;
  polaroids.value = data;
};

const isOwner = computed(() => {
  return user.id === memory.owner;
});

watch(memory, getImageUrls);
getImageUrls();
</script>

<template>
  <div>
    <MemoriesSidebar v-model:open="openSidebar" :refresh="refresh" />
    <div v-if="memory">
      <ul class="flex flex-wrap gap-6">
        <li
          v-for="polaroid in polaroids"
          class="px-4 pt-5 pb-16 rounded-xl border-2 inline-block"
        >
          <img
            width="250"
            height="250"
            class="w object-cover object-center aspect-square"
            :src="polaroid.signedUrl"
          />
        </li>
        <li
          v-if="isOwner"
          class="px-4 pt-5 pb-16 rounded-xl border-2 inline-block"
        >
          <button
            type="button"
            class="relative block w-full rounded-lg border-2 border-dashed border-gray-300 p-12 text-center hover:border-gray-400 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2"
            @click="openSidebar = true"
          >
            <PhotoIcon class="w-12 m-auto" />
            <span class="mt-2 block text-sm font-medium text-gray-900"
              >Create a new Polaroid</span
            >
          </button>
        </li>
      </ul>
    </div>
  </div>
</template>

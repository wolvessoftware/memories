<script lang="ts" setup>
import type { Database } from "~/types";

definePageMeta({
  layout: "dashboard",
  middleware: "auth",
});

const user = useSupabaseUser();
const client = useSupabaseClient<Database>();
const route = useRoute();
const polaroids = ref([]);

const { id: memoryId } = route.params;

const { data: memory, refresh } = await useAsyncData("memories", async () => {
  const { data } = await client
    .from("memories")
    .select("name, owner, polaroids (path, id)")
    .eq("id", memoryId);
  if (!data?.length) return;
  return data[0];
});

const getImageUrls = async () => {
  const { data, error } = await client.storage
    .from("polaroids")
    .createSignedUrls(
      memory.value.polaroids.map((p) => p.path),
      60
    );
  polaroids.value = data;
};

const uploadPhoto = async (e: InputEvent) => {
  if (!e || !user) return;
  const image = e.target.files[0];
  const { data: newImage, error: uploadError } = await client.storage
    .from("polaroids")
    .upload(`${user.value.id}/${image.name}`, image, {
      upsert: false,
    });
  if (uploadError) return;
  const { data, error } = await client
    .from("polaroids")
    .insert({
      path: newImage.path,
      owner: user.value?.id,
      memory: memoryId,
    })
    .select();
  if (!error) {
    await refresh();
    getImageUrls();
  }
};
getImageUrls();
</script>

<template>
  <div class="sm:grid sm:grid-cols-3 sm:items-start sm:gap-4 sm:pt-5">
    <label
      for="cover-photo"
      class="block text-sm font-medium text-gray-700 sm:mt-px sm:pt-2"
      >Photos</label
    >
    <div class="mt-1 sm:col-span-2 sm:mt-0">
      <div
        class="flex max-w-lg justify-center rounded-md border-2 border-dashed border-gray-300 px-6 pt-5 pb-6"
      >
        <div class="space-y-1 text-center">
          <svg
            class="mx-auto h-12 w-12 text-gray-400"
            stroke="currentColor"
            fill="none"
            viewBox="0 0 48 48"
            aria-hidden="true"
          >
            <path
              d="M28 8H12a4 4 0 00-4 4v20m32-12v8m0 0v8a4 4 0 01-4 4H12a4 4 0 01-4-4v-4m32-4l-3.172-3.172a4 4 0 00-5.656 0L28 28M8 32l9.172-9.172a4 4 0 015.656 0L28 28m0 0l4 4m4-24h8m-4-4v8m-12 4h.02"
              stroke-width="2"
              stroke-linecap="round"
              stroke-linejoin="round"
            />
          </svg>
          <div class="flex text-sm text-gray-600">
            <label
              for="file-upload"
              class="relative cursor-pointer rounded-md bg-white font-medium text-indigo-600 focus-within:outline-none focus-within:ring-2 focus-within:ring-indigo-500 focus-within:ring-offset-2 hover:text-indigo-500"
            >
              <span>Upload a file</span>
              <input
                id="file-upload"
                name="file-upload"
                type="file"
                class="sr-only"
                @change="uploadPhoto"
              />
            </label>
            <p class="pl-1">or drag and drop</p>
          </div>
          <p class="text-xs text-gray-500">PNG, JPG, GIF up to 10MB</p>
        </div>
      </div>
      <div v-if="polaroids.length" class="pt-6">
        <h4>Photos</h4>
        <ul role="list" class="divide-y divide-gray-200">
          <li
            v-for="polaroid in polaroids"
            :key="polaroid.path"
            class="flex py-4"
          >
            <img class="h-30 w-20" :src="polaroid.signedUrl" alt="" />
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

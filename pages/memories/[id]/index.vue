<script lang="ts" setup>
import { PencilSquareIcon } from "@heroicons/vue/20/solid";
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

const { data: memory } = await useAsyncData("memories", async () => {
  const { data } = await client
    .from("memories")
    .select("name, owner, polaroids (path)")
    .eq("id", id);
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

const isOwner = computed(() => {
  return user.id === memory.owner;
});

getImageUrls();
</script>

<template>
  <div>
    <div v-if="isOwner">
      <NuxtLink :to="`/memories/${id}/photos`">
        <PencilSquareIcon class="h-5 w-5" />
      </NuxtLink>
    </div>
  </div>
  <div v-if="memory">
    <ul>
      <li v-for="polaroid in polaroids">
        <img :src="polaroid.signedUrl" />
      </li>
    </ul>
  </div>
</template>

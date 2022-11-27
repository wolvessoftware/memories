<script lang="ts" setup>
import type { Database } from "~/types";

definePageMeta({
  layout: "dashboard",
  middleware: "auth",
});

const client = useSupabaseClient<Database>();
const user = useSupabaseUser();
const router = useRouter();

const name = ref("");
const isPrivat = ref(true);

const createMemory = async (e: Event) => {
  e.preventDefault();
  if (name.value) {
    const { data, error } = await client
      .from("memories")
      .insert({
        name: name.value,
        privat: isPrivat.value,
        owner: user.value?.id,
      })
      .select();
    // TODO: Error handling
    if (error) return;
    if (data && data[0]) {
      navigateTo("/memories/" + data[0].id);
    }
  }
};
</script>

<template>
  <form class="space-y-8 divide-y divide-gray-200">
    <div class="space-y-8 divide-y divide-gray-200 sm:space-y-5">
      <div class="space-y-6 sm:space-y-5">
        <div>
          <h3 class="text-lg font-medium leading-6 text-gray-900">Memory</h3>
          <p class="mt-1 max-w-2xl text-sm text-gray-500">
            You are currently creating a new memeory.
          </p>
        </div>

        <div class="space-y-6 sm:space-y-5">
          <div
            class="sm:grid sm:grid-cols-3 sm:items-start sm:gap-4 sm:border-t sm:border-gray-200 sm:pt-5"
          >
            <label
              for="name"
              class="block text-sm font-medium text-gray-700 sm:mt-px sm:pt-2"
              >Name</label
            >
            <div class="mt-1 sm:col-span-2 sm:mt-0">
              <input
                v-model="name"
                type="text"
                name="name"
                id="name"
                class="block w-full max-w-lg rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:max-w-xs sm:text-sm"
              />
            </div>
          </div>
        </div>
        <div class="pt-6 sm:pt-5">
          <div role="group" aria-labelledby="label-notifications">
            <div class="sm:grid sm:grid-cols-3 sm:items-baseline sm:gap-4">
              <div>
                <div
                  class="text-base font-medium text-gray-900 sm:text-sm sm:text-gray-700"
                  id="label-notifications"
                >
                  Privacy
                </div>
              </div>
              <div class="sm:col-span-2">
                <div class="max-w-lg">
                  <p class="text-sm text-gray-500">
                    You can either make your board acessable for everyone with
                    the link to the board or only for people you invite.
                  </p>
                  <div class="mt-4 space-y-4">
                    <div class="flex items-center">
                      <input
                        v-model="isPrivat"
                        id="public"
                        name="isPrivat"
                        type="radio"
                        checked
                        :value="false"
                        class="h-4 w-4 border-gray-300 text-indigo-600 focus:ring-indigo-500"
                      />
                      <label
                        for="public"
                        class="ml-3 block text-sm font-medium text-gray-700"
                      >
                        Public
                      </label>
                    </div>
                    <div class="flex items-center">
                      <input
                        v-model="isPrivat"
                        id="private"
                        name="isPrivat"
                        type="radio"
                        :value="true"
                        class="h-4 w-4 border-gray-300 text-indigo-600 focus:ring-indigo-500"
                      />
                      <label
                        for="private"
                        class="ml-3 block text-sm font-medium text-gray-700"
                      >
                        Only via invite
                      </label>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div class="pt-5">
      <div class="flex justify-end">
        <button
          type="button"
          class="rounded-md border border-gray-300 bg-white py-2 px-4 text-sm font-medium text-gray-700 shadow-sm hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2"
          @click="router.back()"
        >
          Cancel
        </button>
        <button
          type="submit"
          class="ml-3 inline-flex justify-center rounded-md border border-transparent bg-indigo-600 py-2 px-4 text-sm font-medium text-white shadow-sm hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2"
          @click="createMemory"
        >
          Save
        </button>
      </div>
    </div>
  </form>
</template>

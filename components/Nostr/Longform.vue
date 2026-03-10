<script setup>
import { ref, onMounted } from 'vue';
import NDK from '@nostr-dev-kit/ndk';
import setup from "~/config/setup";
import { bech32 } from "bech32";
import { useI18n } from "vue-i18n";

// -----------------------------
// Refs & state
// -----------------------------
const events = ref([]);
const isLoading = ref(true);
const page = ref(0);
const pageSize = 6; // smaller chunks for better UX
const lastFetchedTimestamp = ref(0); // track last event time
const hasMore = ref(true); // controls Load More button

// -----------------------------
// NDK connection
// -----------------------------
const ndk = new NDK({ explicitRelayUrls: setup.relays });

// -----------------------------
// Helper functions
// -----------------------------
const bytesToHex = (bytes) =>
  Array.from(bytes).map(b => b.toString(16).padStart(2,'0')).join('');

const npubToHex = (npub) => {
  const decoded = bech32.decode(npub);
  const pubkeyBytes = bech32.fromWords(decoded.words);
  return bytesToHex(Uint8Array.from(pubkeyBytes));
};

const skHex = npubToHex(setup.nostradmin);

// -----------------------------
// Fetch events function (incremental)
// -----------------------------
const fetchEvents = async () => {
  if (!hasMore.value) return; // stop if no more events

  isLoading.value = true;
  if (!ndk.isConnected) await ndk.connect();

  const filter = {
    kinds: [30023],
    authors: [skHex],
    limit: pageSize,
    until: lastFetchedTimestamp.value || undefined, // fetch older events
  };

  const fetched = await ndk.fetchEvents(filter);
  const newEvents = Array.from(fetched)
    .map(event => {
      const imageTag = event.tags.find(tag => tag[0] === "image");
      const summaryTag = event.tags.find(tag => tag[0] === "summary");
      const titleTag = event.tags.find(tag => tag[0] === "title");
      return {
        ...event,
        image: imageTag ? imageTag[1] : null,
        summary: summaryTag ? summaryTag[1] : null,
        title: titleTag ? titleTag[1] : null,
      };
    })
    // newest first
    .sort((a, b) => b.created_at - a.created_at)
    // remove duplicates
    .filter(e => !events.value.find(ev => ev.id === e.id));

  // Append to existing events
  events.value.push(...newEvents);

  // Update last fetched timestamp for next page
  if (newEvents.length > 0) {
    lastFetchedTimestamp.value = Math.min(...newEvents.map(e => e.created_at));
  } else {
    hasMore.value = false; // no more events to load
  }

  isLoading.value = false;
};

// -----------------------------
// Load more
// -----------------------------
const loadMore = async () => {
  await fetchEvents();
};

// -----------------------------
// On mounted
// -----------------------------
onMounted(async () => {
  await fetchEvents();
});

// -----------------------------
// i18n
// -----------------------------
const { t } = useI18n({ useScope: "local" });
</script>

<template>
  <div class="bg-colorBgLight dark:bg-colorBgDark my-24">
    <div class="mx-auto max-w-7xl px-6 lg:px-8">
      <!-- Header -->
      <div class="mx-auto max-w-2xl text-center">
        <h2 class="text-3xl font-bold tracking-tight text-gray-900 dark:text-white sm:text-4xl">
          {{ t("title") }}
        </h2>
      </div>

      <!-- Loading -->
      <div v-if="isLoading" class="text-center my-10">
        <svg class="animate-spin h-8 w-8 text-gray-500 dark:text-gray-300 mx-auto" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
          <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
          <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
        </svg>
      </div>

      <!-- Grid -->
      <div v-else>
        <div v-if="events.length === 0" class="text-center mt-12">
          <p>No Notes found</p>
        </div>

        <div v-else class="mx-auto grid max-w-2xl grid-cols-1 gap-x-8 gap-y-20 lg:mx-0 lg:max-w-none lg:grid-cols-3 mt-12">
          <article v-for="event in events" :key="event.id" class="flex flex-col items-start">
            <NuxtLink :to="localePath('/note/' + event.id)">
              <div class="relative w-full">
                <img :src="event.image || '/placeholder-img.png'" alt="Event Image" class="aspect-[16/9] w-full rounded-2xl bg-gray-100 object-cover sm:aspect-[2/1] lg:aspect-[3/2] border dark:border-white" />
                <div class="absolute inset-0 rounded-2xl ring-1 ring-inset ring-gray-900/10"></div>
              </div>
            </NuxtLink>
            <div class="max-w-xl">
              <div class="mt-8 flex items-center gap-x-4 text-xs">
                <time :datetime="new Date(event.created_at * 1000).toISOString()" class="text-gray-500 dark:text-white">
                  {{ new Date(event.created_at * 1000).toLocaleDateString() }}
                </time>
              </div>
              <div class="group relative">
                <h3 class="mt-3 text-lg font-semibold leading-6 text-gray-900 dark:text-white">
                  <NuxtLink :to="localePath('/note/' + event.id)">
                    <span class="absolute inset-0"></span>
                    {{ event.title }}
                  </NuxtLink>
                </h3>
                <p class="mt-5 line-clamp-3 text-sm leading-6 text-gray-900 dark:text-gray-100">
                  {{ event.summary }}
                </p>
              </div>
            </div>
          </article>
        </div>

        <!-- Load More -->
        <div v-if="hasMore" class="text-center mt-8">
          <button @click="loadMore" class="bg-blue-500 text-white px-4 py-2 rounded-lg">
            Load More
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

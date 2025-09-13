<script setup lang="ts">
import ArtistSearch from '@/components/ArtistSearch.vue'
import TrackCard from '@/components/TrackCard.vue'
import { ref } from 'vue'

const artistName = ref('')
const currentTrack = ref<object>({})

async function handleSearch(name: string) {
  artistName.value = name
}
</script>

<template>
  <main>
    <div class="home-container">
      <ArtistSearch
        v-model:artist-name="artistName"
        v-model:currentTrack="currentTrack"
        @search="handleSearch"
      />
      <TrackCard v-if="Object.keys(currentTrack).length" :track="currentTrack" />
    </div>
  </main>
</template>

<style scoped>
.home-container {
  display: flex;
  flex-direction: row;
  align-items: center;
  padding: 16px;
}
</style>

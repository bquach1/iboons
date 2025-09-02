<script setup lang="ts">
import ArtistSearch from '@/components/ArtistSearch.vue'
import ArtistCard from '@/components/ArtistCard.vue'
import { ref } from 'vue'

const artistName = ref('')
const artist = ref(null)

async function handleSearch(name: string) {
  artistName.value = name
}
</script>

<template>
  <main>
    <div class="home-container">
      <ArtistSearch v-model="artistName" @search="handleSearch" />
      <ArtistCard v-if="artist" :artist="artist" />
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

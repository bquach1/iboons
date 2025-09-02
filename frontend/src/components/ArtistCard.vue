<script setup lang="ts">
import { defineProps } from 'vue'

interface Artist {
  id: string
  name: string
  images: { url: string }[]
  albums: { id: string; name: string; images: { url: string }[] }[]
  related: { id: string; name: string; images: { url: string }[] }[]
}

const props = defineProps<{ artist: Artist }>()
</script>

<template>
  <v-card class="artist-card" elevation="3">
    <v-img :src="props.artist.images[0]?.url" height="200" class="mb-2" />
    <v-card-title>{{ props.artist.name }}</v-card-title>
    <v-card-subtitle>Albums</v-card-subtitle>
    <v-slide-group show-arrows class="mb-2">
      <v-slide-item v-for="album in props.artist.albums" :key="album.id">
        <v-card class="album-card" elevation="1">
          <v-img :src="album.images[0]?.url" height="100" />
          <v-card-text>{{ album.name }}</v-card-text>
        </v-card>
      </v-slide-item>
    </v-slide-group>
    <v-card-subtitle>Related Artists</v-card-subtitle>
    <v-slide-group show-arrows>
      <v-slide-item v-for="rel in props.artist.related" :key="rel.id">
        <v-card class="related-card" elevation="1">
          <v-img :src="rel.images[0]?.url" height="100" />
          <v-card-text>{{ rel.name }}</v-card-text>
        </v-card>
      </v-slide-item>
    </v-slide-group>
  </v-card>
</template>

<style scoped>
.artist-card {
  max-width: 600px;
  margin: 2rem auto;
}
.album-card,
.related-card {
  width: 120px;
  margin: 0 8px;
}
</style>

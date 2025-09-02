<script setup lang="ts">
const props = defineProps<{ modelValue: string }>()
const emit = defineEmits<{ (e: 'update:modelValue', value: string): void }>()
import { ref } from 'vue'

const localInput = ref(props.modelValue)

function onSubmit() {
  emit('update:modelValue', localInput.value)
  searchArtist(localInput.value)
}
import axios from 'axios'

const loading = ref(false)
const error = ref('')
interface Track {
  id: string
  name: string
  artists: { name: string }[]
  preview_url: string | null
}
const tracks = ref<Track[]>([])

const clientId = import.meta.env.VITE_SPOTIFY_CLIENT_ID
const clientSecret = import.meta.env.VITE_SPOTIFY_CLIENT_SECRET

async function getSpotifyAccessToken() {
  const creds = btoa(`${clientId}:${clientSecret}`)
  const params = new URLSearchParams()
  params.append('grant_type', 'client_credentials')
  const res = await axios.post('https://accounts.spotify.com/api/token', params, {
    headers: {
      Authorization: `Basic ${creds}`,
      'Content-Type': 'application/x-www-form-urlencoded',
    },
  })
  return res.data.access_token
}

async function searchArtist(artistName: string) {
  error.value = ''
  tracks.value = []
  if (!artistName) {
    error.value = 'Please enter an artist name.'
    return
  }
  loading.value = true
  const accessToken = await getSpotifyAccessToken()
  const searchRes = await fetch(
    `https://api.spotify.com/v1/search?q=${encodeURIComponent(artistName)}&type=artist&limit=1`,
    {
      headers: {
        Authorization: `Bearer ${accessToken}`,
      },
    },
  )
  const searchData = await searchRes.json()
  const artist = searchData.artists?.items?.[0]
  if (!artist) {
    error.value = 'Artist not found.'
    loading.value = false
    return
  }
  const tracksRes = await fetch(
    `https://api.spotify.com/v1/artists/${artist.id}/top-tracks?market=US`,
    {
      headers: {
        Authorization: `Bearer ${accessToken}`,
      },
    },
  )
  const tracksData = await tracksRes.json()
  tracks.value = tracksData.tracks || []
  loading.value = false
}
</script>

<template>
  <div class="home">
    <div style="display: flex; margin-bottom: 1rem;">
      <v-icon class="icon">mdi-music</v-icon>
      <div>iBoons</div>
    </div>
    <v-card class="search-card" elevation="2">
      <v-card-title>Search Spotify Artist Top Tracks</v-card-title>
      <v-text-field
        v-model="localInput"
        label="Artist Name"
        prepend-inner-icon="mdi-account-music"
        :disabled="loading"
        @keyup.enter="onSubmit"
      />
      <v-btn
        color="primary"
        :loading="loading"
        @click="onSubmit"
        style="margin-left: 10px; margin-bottom: 10px"
        >Search</v-btn
      >
      <div v-if="error" class="error">{{ error }}</div>
      <v-list v-if="tracks.length">
        <v-list-item v-for="track in tracks" :key="track.id">
          <template #prepend>
            <img
              v-if="track.album && track.album.images && track.album.images.length"
              :src="track.album.images[track.album.images.length - 1].url"
              alt="Album Art"
              width="64"
              height="64"
              style="margin-right: 16px; border-radius: 4px"
            />
          </template>
          <div class="track">
            <strong>{{ track.name }}</strong>
            <div class="track-artist">{{ track.artists.map((a) => a.name).join(', ') }}</div>
            <audio
              v-if="track.preview_url"
              :src="track.preview_url"
              controls
              style="width: 100%; margin-top: 4px"
            />
          </div>
        </v-list-item>
      </v-list>
    </v-card>
  </div>
</template>

<style scoped>
h1 {
  font-weight: 500;
  font-size: 2.6rem;
  position: relative;
}

h3 {
  font-size: 1.2rem;
}

.icon {
  margin-right: 0.5rem;
}

.search-card {
  max-width: 500px;
  max-height: 500px;
  overflow-y: scroll;
  margin: 0 auto;
}

.track {
  cursor: pointer;
}

.error {
  color: red;
  margin-top: 8px;
}

.home {
  display: flex;
  flex-direction: column;
}

.track-artist {
  font-size: 0.9rem;
  color: #666;
}
</style>

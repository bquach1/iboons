<script setup lang="ts">
import { defineProps, onMounted, ref } from 'vue'

declare global {
  interface Window {
    Spotify: {
      Player: new (options: {
        name: string
        getOAuthToken: (cb: (token: string) => void) => void
        volume?: number
      }) => SpotifyPlayer
    }
  }
}

interface SpotifyPlayer {
  connect(): Promise<boolean>
  disconnect(): Promise<void>
  removeListener(event: string): void
  getCurrentState(): Promise<void>
  addListener(event: string, cb: (...args: unknown[]) => void): boolean
}

interface Track {
  id?: string
  name?: string
  artists?: { name: string }[]
  explicit?: boolean
  preview_url?: string | null
  album?: {
    images?: { url: string }[]
  }
  uri?: string
}

const playerReady = ref(false)
const isPlaying = ref(false)
const sdkLoaded = ref(false)
const accessToken = ref('')

const CLIENT_ID = import.meta.env.VITE_SPOTIFY_CLIENT_ID
const REDIRECT_URI = import.meta.env.VITE_SPOTIFY_REDIRECT_URI
const SCOPES = ['user-modify-playback-state', 'user-read-playback-state', 'streaming']

function getSpotifyAuthUrl() {
  const params = new URLSearchParams({
    client_id: CLIENT_ID,
    response_type: 'token',
    redirect_uri: REDIRECT_URI,
    scope: SCOPES.join(' '),
    show_dialog: 'true',
  })
  return `https://accounts.spotify.com/authorize?${params.toString()}`
}

function parseAccessTokenFromHash() {
  const hash = window.location.hash.substring(1)
  const params = new URLSearchParams(hash)
  return params.get('access_token') || ''
}

function loginWithSpotify() {
  window.location.href = getSpotifyAuthUrl()
}

onMounted(() => {
  const tokenFromHash = parseAccessTokenFromHash()
  if (tokenFromHash) {
    accessToken.value = tokenFromHash
    localStorage.setItem('spotify_access_token', tokenFromHash)
    window.location.hash = ''
  } else {
    const storedToken = localStorage.getItem('spotify_access_token')
    if (storedToken) {
      accessToken.value = storedToken
    }
  }
})

function loadSpotifySDK() {
  return new Promise<void>((resolve) => {
    if (window.Spotify) {
      sdkLoaded.value = true
      resolve()
      return
    }
    const script = document.createElement('script')
    script.src = 'https://sdk.scdn.co/spotify-player.js'
    script.onload = () => {
      sdkLoaded.value = true
      resolve()
    }
    document.body.appendChild(script)
  })
}

let player: SpotifyPlayer | null = null

onMounted(async () => {
  await loadSpotifySDK()
  if (!accessToken.value) {
    return
  }
  player = new window.Spotify.Player({
    name: 'Web Playback SDK Player',
    getOAuthToken: (cb: (token: string) => void) => {
      cb(accessToken.value)
    },
    volume: 0.8,
  })
  player.addListener('ready', (...args: unknown[]) => {
    const event = args[0] as { device_id: string }
    playerReady.value = true
    if (event && event.device_id) {
      localStorage.setItem('spotify_device_id', event.device_id)
    }
  })
  player.addListener('not_ready', () => {
    playerReady.value = false
  })
  await player.connect()
})

async function playTrack() {
  if (!playerReady.value || !props.track.uri || !accessToken.value) return
  isPlaying.value = true
  const device_id = localStorage.getItem('spotify_device_id')
  if (!device_id) {
    alert('Spotify device not ready. Try again in a moment.')
    return
  }
  await fetch('https://api.spotify.com/v1/me/player', {
    method: 'PUT',
    headers: {
      Authorization: `Bearer ${accessToken.value}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({ device_ids: [device_id], play: true }),
  })
  const res = await fetch('https://api.spotify.com/v1/me/player/play?device_id=' + device_id, {
    method: 'PUT',
    headers: {
      Authorization: `Bearer ${accessToken.value}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({ uris: [props.track.uri] }),
  })
  if (res.ok) {
    isPlaying.value = true
  } else {
    isPlaying.value = false
    alert('Failed to play track. Make sure Spotify is open and you have a Premium account.')
  }
}

const props = defineProps<{ track: Track }>()
</script>

<template>
  <v-card class="track-card" elevation="3">
    <v-img :src="props.track.album?.images?.[0]?.url" height="200" class="mb-2" />
    <v-card-title>{{ props.track.name }}</v-card-title>
    <v-card-subtitle>Spotify Playback</v-card-subtitle>
    <v-btn
      :disabled="!playerReady || !sdkLoaded || !props.track.uri || !accessToken"
      @click="playTrack"
      color="primary"
    >
      Play on Spotify
    </v-btn>
    <v-btn v-if="!accessToken" @click="loginWithSpotify" color="secondary">
      Login with Spotify
    </v-btn>
  </v-card>
</template>

<style scoped>
.track-card {
  max-width: 600px;
  margin: 2rem auto;
}
.album-card,
.related-card {
  width: 120px;
  margin: 0 8px;
}
</style>

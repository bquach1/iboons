<template>
  <header class="header">
    <div class="header-left">
      <nav>
        <button @click="$emit('show-artist-search')">Artist Search</button>
        <button @click="$emit('show-profile')">My Profile</button>
        <button @click="$emit('show-playlists')">My Playlists</button>
      </nav>
    </div>
    <div class="header-right">
      <template v-if="user">
        <v-avatar v-if="user.images && user.images.length" class="mr-2 profile-avatar">
          <img :src="user.images[0].url" alt="Profile" class="avatar-img" />
        </v-avatar>
        <span>{{ user.display_name }}</span>
      </template>
      <button v-else class="profile-btn" @click="loginWithSpotify" title="Login with Spotify">
        <v-icon class="icon">mdi-account-circle</v-icon>
      </button>
    </div>
  </header>
</template>

<script lang="ts" setup>
import { ref, onMounted } from 'vue'

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

function loginWithSpotify() {
  const authUrl = getSpotifyAuthUrl()
  window.location.href = authUrl
}

interface SpotifyUser {
  display_name?: string
  images?: { url: string }[]
}

const user = ref<SpotifyUser | null>(null)
const accessToken = ref('')

function parseAccessTokenFromHash() {
  const hash = window.location.hash.substring(1)
  const params = new URLSearchParams(hash)
  return params.get('access_token') || ''
}

onMounted(() => {
  const tokenFromHash = parseAccessTokenFromHash()
  if (tokenFromHash) {
    accessToken.value = tokenFromHash
    localStorage.setItem('spotify_access_token', tokenFromHash)
    window.location.hash = ''
    fetchSpotifyUser(tokenFromHash)
  } else {
    const storedToken = localStorage.getItem('spotify_access_token')
    if (storedToken) {
      accessToken.value = storedToken
      fetchSpotifyUser(storedToken)
    }
  }
})

async function fetchSpotifyUser(token: string) {
  try {
    const res = await fetch('https://api.spotify.com/v1/me', {
      headers: {
        Authorization: `Bearer ${token}`,
      },
    })
    if (res.ok) {
      user.value = await res.json()
    } else {
      user.value = null
    }
  } catch {
    user.value = null
  }
}
</script>

<style scoped>
.header {
  width: 100%;
  position: sticky;
  top: 0;
  z-index: 100;
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 1rem;
  background: #181818;
  color: #fff;
}
.header-left {
  display: flex;
  align-items: center;
}
.logo {
  height: 40px;
  margin-right: 24px;
}
nav button {
  margin-right: 16px;
  background: #1db954;
  color: #fff;
  border: none;
  padding: 8px 16px;
  border-radius: 20px;
  cursor: pointer;
  font-weight: 500;
}
nav button:last-child {
  margin-right: 0;
}
.header-right {
  display: flex;
  align-items: center;
}
.icon {
  font-size: 32px;
  color: #fff;
}
.profile-btn {
  background: none;
  border: none;
  cursor: pointer;
  padding: 0;
  display: flex;
  align-items: center;
}
.profile-btn {
  background: none;
  border: none;
  cursor: pointer;
}
.profile-avatar {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 40px;
  height: 40px;
}
.avatar-img {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  object-fit: cover;
  display: block;
  margin: 0 auto;
}
</style>

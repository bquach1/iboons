<script setup lang="ts">
import { ref } from 'vue';
import axios from 'axios';

const artistName = ref('');
const loading = ref(false);
const error = ref('');
interface Track {
  id: string;
  name: string;
  artists: { name: string }[];
  explicit: boolean;
  preview_url: string | null;
}
const tracks = ref<Track[]>([]);

const clientId = import.meta.env.VITE_SPOTIFY_CLIENT_ID;
const clientSecret = import.meta.env.VITE_SPOTIFY_CLIENT_SECRET;

async function getSpotifyAccessToken( ) {
  const creds = btoa(`${clientId}:${clientSecret}`);
  const params = new URLSearchParams();
  params.append('grant_type', 'client_credentials');
  const res = await axios.post(
    'https://accounts.spotify.com/api/token',
    params,
    {
      headers: {
        'Authorization': `Basic ${creds}`,
        'Content-Type': 'application/x-www-form-urlencoded',
      },
    }
  );
  return res.data.access_token;
}

async function searchArtist() {
  error.value = '';
  tracks.value = [];
  if (!artistName.value) {
    error.value = 'Please enter an artist name.';
    return;
  }
  loading.value = true;
  try {
    const accessToken = await getSpotifyAccessToken();
    const searchRes = await fetch(
      `https://api.spotify.com/v1/search?q=${encodeURIComponent(artistName.value)}&type=artist&limit=1`,
      {
        headers: {
          'Authorization': `Bearer ${accessToken}`,
        },
      }
    );
    const searchData = await searchRes.json();
    const artist = searchData.artists?.items?.[0];
    if (!artist) {
      error.value = 'Artist not found.';
      loading.value = false;
      return;
    }
    // Get top tracks
    const tracksRes = await fetch(
      `https://api.spotify.com/v1/artists/${artist.id}/top-tracks?market=US`,
      {
        headers: {
          'Authorization': `Bearer ${accessToken}`,
        },
      }
    );
    const tracksData = await tracksRes.json();
    tracks.value = tracksData.tracks || [];
  } catch {
    error.value = 'Error fetching data.';
  }
  loading.value = false;
}
</script>

<template>
  <div class="home">
    <v-icon class="icon">mdi-music</v-icon>
    <div>iBoons</div>
  </div>
  <v-card class="search-card" elevation="2">
    <v-card-title>Search Spotify Artist Top Tracks</v-card-title>
    <v-card-text>
      <v-text-field
        v-model="artistName"
        label="Artist Name"
        prepend-inner-icon="mdi-account-music"
        :disabled="loading"
        @keyup.enter="searchArtist"
      />
      <v-btn
        color="primary"
        :loading="loading"
        @click="searchArtist"
        class="mt-2"
      >Search</v-btn>
      <div v-if="error" class="error">{{ error }}</div>
      <v-list v-if="tracks.length">
        <v-list-item v-for="track in tracks" :key="track.id">
          <template #prepend>
            <v-icon v-if="track.explicit" color="red">mdi-alert</v-icon>
          </template>
          <div class="track">
            <strong>{{ track.name }}</strong>
            <div class="track-artist">{{ track.artists.map(a => a.name).join(', ') }}</div>
            <audio v-if="track.preview_url" :src="track.preview_url" controls style="width: 100%; margin-top: 4px;" />
          </div>
        </v-list-item>
      </v-list>
    </v-card-text>
  </v-card>
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

.home {
  display: flex;
  justify-content: center;
  margin: auto;
}

.icon {
  margin-right: 0.5rem;
}

.search-card {
  max-width: 500px;
  margin: 0 auto;
} 

.track {
  cursor: pointer;
}

.error {
  color: red;
  margin-top: 8px;
}

.track-artist {
  font-size: 0.9rem;
  color: #666;
}
</style>

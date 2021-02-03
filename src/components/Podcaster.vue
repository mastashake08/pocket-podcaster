<template>
  <v-container>
    <v-row class="text-center">
      <v-col cols="12">
        <v-img
          :src="require('../assets/logo.png')"
          class="my-3"
          contain
          height="200"
        />
      </v-col>

      <v-col class="mb-4">
        <h1 class="display-2 font-weight-bold mb-3">
          Welcome to Pocket Podcaster
        </h1>

        <p class="subheading font-weight-regular text-center">
          <v-text-field type="url" placeholder="Enter stream URL" v-model="url" label="Stream URL" />
        </p>
        <v-row class="text-center">
          <v-btn v-on:click="playAudio" v-if="!isPlaying">Play</v-btn>
            <v-btn v-on:click="stopAudio" v-else color="red">Stop</v-btn>
        </v-row>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
  export default {
    name: 'Podcaster',

    data: () => ({
      isPlaying: false,
      audio: {},
      url : 'https://playerservices.streamtheworld.com/api/livestream-redirect/WEKUFM.mp3'
    }),
    methods: {
      setMediaControls: function () {
        if ('mediaSession' in navigator) {
          navigator.mediaSession.metadata = new window.MediaMetadata({
            title: 'Pocket Podcaster',
            artist: 'J Computer Solutions LLC',
            album: 'Pocket Podcaster',
            artwork: [
              { src: '../assets/logo-96.png',   sizes: '96x96',   type: 'image/png' },
              { src: '../assets/logo-128.png', sizes: '128x128', type: 'image/png' },
              { src: '../assets/logo-192.png', sizes: '192x192', type: 'image/png' },
              { src: '../assets/logo-256.png', sizes: '256x256', type: 'image/png' },
              { src: '../assets/logo-384.png', sizes: '384x384', type: 'image/png' },
              { src: '../assets/logo-512.png', sizes: '512x512', type: 'image/png' },
            ]
          });

          navigator.mediaSession.setActionHandler('play', this.playAudio());
          navigator.mediaSession.setActionHandler('pause', this.pauseAudio());
          navigator.mediaSession.setActionHandler('stop', this.stopAudio());
        }
      },
      playAudio: function () {
        this.audio = new Audio(this.url)
        this.isPlaying = true
        this.audio.play()
          .then(()=> {
        }).catch(error => { console.log(error) });
      },
      pauseAudio: function () {
        this.audio.pause()
        this.isPlaying = false
      },
      stopAudio: function () {
        this.audio.pause()
        this.audio = {}
        this.isPlaying = false
      }
    },
    mounted () {
      this.setMediaControls()
    }
  }
</script>

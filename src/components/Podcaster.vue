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
          <v-btn v-on:click="playAudio">Play</v-btn>
        </v-row>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
  export default {
    name: 'Podcaster',

    data: () => ({
      audio: {},
      url : 'https://playerservices.streamtheworld.com/api/livestream-redirect/WEKUFM.mp3'
    }),
    methods: {
      setMediaControls: function () {
        if ('mediaSession' in navigator) {
          navigator.mediaSession.metadata = new window.MediaMetadata({
            title: 'Unforgettable',
            artist: 'Nat King Cole',
            album: 'The Ultimate Collection (Remastered)',
            artwork: [
              { src: 'https://dummyimage.com/96x96',   sizes: '96x96',   type: 'image/png' },
              { src: 'https://dummyimage.com/128x128', sizes: '128x128', type: 'image/png' },
              { src: 'https://dummyimage.com/192x192', sizes: '192x192', type: 'image/png' },
              { src: 'https://dummyimage.com/256x256', sizes: '256x256', type: 'image/png' },
              { src: 'https://dummyimage.com/384x384', sizes: '384x384', type: 'image/png' },
              { src: 'https://dummyimage.com/512x512', sizes: '512x512', type: 'image/png' },
            ]
          });

          navigator.mediaSession.setActionHandler('play', this.playAudio());
          navigator.mediaSession.setActionHandler('pause', this.pauseAudio());
          navigator.mediaSession.setActionHandler('stop', this.stopAudio());
        }
      },
      playAudio: function () {
        this.audio = new Audio(this.url)
        console.log(this.audio)
        this.audio.play()
          .then(()=> {
        }).catch(error => { console.log(error) });
      },
      pauseAudio: function () {
        this.audio.pause()
      },
      stopAudio: function () {
        this.audio.pause()
        this.audio = {}
      }
    },
    mounted () {
      this.setMediaControls()
    }
  }
</script>

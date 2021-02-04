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
          Welcome to PWA Radio
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
    <v-row class="text-center">
      <v-btn class="pa-md-4 mx-lg-auto" v-for="x in presets" v-on:click="setAudio(x)" :key="x.name" :color="x.color"> {{x.name}} </v-btn>
    </v-row>
    <v-row class="text-center">
      <v-select
          v-model="currentPodcast"
          hint="Favorite Podcasts"
          :items="favoritePodcasts"
          item-text="name"
          item-value="url"
          label="Favorite Podcasts"
          persistent-hint
          return-object
          single-line
          @change="playPodcast"
        ></v-select>
    </v-row>
  </v-container>
</template>

<script>
  export default {
    name: 'Radio',

    data: () => ({
      isPlaying: false,
      audio: {},
      url : '',
      currentPodcast: {},
      selectedEpisode: {},
      presets : [
        {
          name: 'WEKU-NPR',
          url : 'https://playerservices.streamtheworld.com/api/livestream-redirect/WEKUFM.mp3',
          color: "green"
        },
        {
          name: 'WEKU-Classical',
          url: 'https://playerservices.streamtheworld.com/api/livestream-redirect/WEKUHD2.mp3',
          color: 'orange'
        },
        {
          name: 'Vocalo Radio',
          url: 'https://stream.wbez.org/vocalo128',
          color: 'blue'
        },
        {
          name: 'WFPK',
          url: 'https://lpm.streamguys1.com/wfpk-popup',
          color: 'yellow'
        },
        {
          name: 'KEXP',
          url: 'https://kexp-mp3-128.streamguys1.com/kexp128.mp3?listenerid=8044407b7410ad01f8210fd508279708&awparams=companionAds%3Atrue',
          color: '#cb349a'
        }
      ],
      favoritePodcasts: [],

      podcastURLS: [
        { url: 'https://anchor.fm/s/fdc3ac0/podcast/rss', name: 'Code Life' },
        { url: 'https://anchor.fm/s/42d5fca4/podcast/rss' , name: 'Intimate Spaces' },
        { url: 'https://feeds.npr.org/510289/podcast.xml', name: 'Project Money'}

      ]
    }),
    methods: {
      setMediaControls: function () {
        if ('mediaSession' in navigator) {
          navigator.mediaSession.metadata = new window.MediaMetadata({
            title: 'Pocket Radio',
            artist: 'J Computer Solutions LLC',
            album: 'Pocket Radio',
            artwork: [
              { src: 'https://radio.jcompsolu.com/images/logo-96.png',   sizes: '96x96',   type: 'image/png' },
              { src: 'https://radio.jcompsolu.com/images/logo-128.png', sizes: '128x128', type: 'image/png' },
              { src: 'https://radio.jcompsolu.com/images/logo-192.png', sizes: '192x192', type: 'image/png' },
              { src: 'https://radio.jcompsolu.com/images/logo-256.png', sizes: '256x256', type: 'image/png' },
              { src: 'https://radio.jcompsolu.com/images/logo-384.png', sizes: '384x384', type: 'image/png' },
              { src: 'https://radio.jcompsolu.com/images/logo-512.png', sizes: '512x512', type: 'image/png' },
            ]
          });

          navigator.mediaSession.setActionHandler('play', this.playAudio());
          navigator.mediaSession.setActionHandler('pause', this.pauseAudio());
          navigator.mediaSession.setActionHandler('stop', this.stopAudio());
          navigator.mediaSession.setActionHandler('seekbackward', function() { /* Code excerpted. */ });
          navigator.mediaSession.setActionHandler('seekforward', function() { /* Code excerpted. */ });
          navigator.mediaSession.setActionHandler('seekto', function() { /* Code excerpted. */ });
          navigator.mediaSession.setActionHandler('previoustrack', function() { /* Code excerpted. */ });
          navigator.mediaSession.setActionHandler('nexttrack', function() { /* Code excerpted. */ });
        }
      },
      playPodcast: function () {
        this.setAudio(this.currentPodcast)
        this.playAudio()
      },
      playAudio: function () {
        if(this.isPlaying){
          this.isPlaying = false
          this.audio.pause()
          this.audio = {}
        }
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
      },
      setAudio: function(preset) {
        this.url = preset.url
        navigator.mediaSession.metadata.title = preset.name
        if(preset.image) {
          navigator.mediaSession.metadata.artwork = [
            { src: preset.image }
          ]
        }
        this.playAudio()
      }
    },
    mounted () {
      this.setMediaControls()
      this.podcastURLS.forEach(pod => {
        fetch(pod.url)
        .then(response => response.text())
        .then(str => new window.DOMParser().parseFromString(str, "text/xml"))
        .then(data => {
          const items = data.querySelectorAll("item");
          for (let i = 0; i < items.length; i++) {
            let item = items[i];
            let image = item.getElementsByTagName("itunes:image")[0].getAttribute("href")
            let title = item.querySelector("title").innerHTML
            let url = item.querySelector("enclosure").getAttribute("url")
            let podcast = { name: title.replace("<![CDATA[", "").replace("]]>", ""), url: url, image: image }
            this.favoritePodcasts.push(podcast)
          }
        })
      })
    }
  }
</script>

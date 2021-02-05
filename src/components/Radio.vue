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
        <p class="subheading font-weight-regular text-center">
          <v-text-field type="url" placeholder="Enter podcast RSS feed" v-model="feed" label="Podcast Feed" />
        </p>
        <v-row class="text-center">
          <v-btn v-on:click="playAudio" v-if="!isPlaying">Play</v-btn>
          <v-btn v-on:click="stopAudio" v-else color="red">Stop</v-btn>
          <v-btn v-on:click="getFeed" color="blue">Get Feed</v-btn>
        </v-row>
      </v-col>
    </v-row>
    <v-row class="text-center">
      <v-btn class="pa-md-4 mx-lg-auto" v-for="x in presets" v-on:click="setAudio(x)" :key="x.name" :color="x.color"> {{x.name}} </v-btn>
    </v-row>
    <v-row class="text-center">
      <v-btn class="pa-md-4 mx-lg-auto ma-md-4" v-for="x in podcastURLS" v-on:click="setFeed(x)" :key="x.name" color="#be4164"> {{x.name}} </v-btn>
    </v-row>
    <v-row class="text-center" v>
      <v-select
          v-model="currentPodcast"
          :items="favoritePodcasts"
          item-text="name"
          item-value="url"
          :label="currentPodcast.name"
          persistent-hint
          return-object
          single-line
          @change="playPodcast"
        ></v-select>
    </v-row>
    <v-row>
      <canvas id = "analyser_render"></canvas>
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
      feed: '',
      currentPodcast: {},
      selectedEpisode: {},
      presets : [
        {
          name: 'WEKU-NPR',
          url : 'https://playerservices.streamtheworld.com/api/livestream-redirect/WEKUFM.mp3',
          color: "green",
          author: 'NPR'
        },
        {
          name: 'WEKU-Classical',
          url: 'https://playerservices.streamtheworld.com/api/livestream-redirect/WEKUHD2.mp3',
          color: 'orange',
          author: 'NPR'
        },
        {
          name: 'Vocalo Radio',
          url: 'https://stream.wbez.org/vocalo128',
          color: 'blue',
          author: 'NPR'
        },
        {
          name: 'WFPK',
          url: 'https://lpm.streamguys1.com/wfpk-popup',
          color: 'yellow',
          author: 'NPR'
        },
        {
          name: 'KEXP',
          url: 'https://kexp-mp3-128.streamguys1.com/kexp128.mp3?listenerid=8044407b7410ad01f8210fd508279708&awparams=companionAds%3Atrue',
          color: '#cb349a',
          author: 'NPR'
        },
        {
          name: 'Lofi Hip Hop Radio',
          url: 'https://stream.zeno.fm/0r0xa792kwzuv',
          color: '#4f99cb',
          author: 'Lofi Hip Hop Radio'
        }
      ],
      favoritePodcasts: [],
      ctx: {},
      canvas: {},
      analyser: {},
      audioContext: {},

      podcastURLS: [
        { url: 'https://anchor.fm/s/fdc3ac0/podcast/rss', name: 'Code Life' },
        { url: 'https://anchor.fm/s/42d5fca4/podcast/rss' , name: 'Intimate Spaces' },
        { url: 'https://feeds.megaphone.fm/darknetdiaries', name: 'Darknet Diaries'}

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
        const AudioContext = window.AudioContext || window.webkitAudioContext;

        this.audioContext = new AudioContext();

        if(this.isPlaying){
          this.isPlaying = false
          this.audio.pause()
          this.audio = {}
        }
        this.audio = new Audio(this.url)
        this.audio.crossOrigin = "no-cors";
        this.isPlaying = true
        this.audio.play()
          .then(()=> {
        }).catch(error => { console.log(error) });


        this.source = this.audioContext.createMediaElementSource(this.audio);
        this.analyser = this.audioContext.createAnalyser()
        console.log('Source', this.source)
        this.source.connect(this.analyser);
        this.analyser.connect(this.audioContext.destination);
        this.canvas = document.getElementById('analyser_render');
        this.ctx = this.canvas.getContext('2d');
        this.frameLooper();

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
        navigator.mediaSession.metadata.artist = preset.author
        if(preset.image) {
          navigator.mediaSession.metadata.artwork = [
            { src: preset.image }
          ]
        }
        this.playAudio()
      },
      getFeed: function () {
        this.favoritePodcasts = []
        fetch(this.feed)
        .then(response => response.text())
        .then(str => new window.DOMParser().parseFromString(str, "text/xml"))
        .then(data => {
          const items = data.querySelectorAll("item");
          for (let i = 0; i < items.length; i++) {
            let item = items[i];
            console.log(item)
            let image = item.getElementsByTagName("itunes:image")[0].getAttribute("href")
            let title = item.querySelector("title").innerHTML.replace("<![CDATA[", "").replace("]]>", "")
            let author = item.getElementsByTagName("dc:creator")[0]

            if(!author){
              author = item.getElementsByTagName("itunes:author")[0].innerHTML
            } else {
              author.innerHTML.replace("<![CDATA[", "").replace("]]>", "")
            }
            let url = item.querySelector("enclosure").getAttribute("url")
            let podcast = { name: title, url: url, image: image, author: author }
            console.log(podcast)
            this.favoritePodcasts.push(podcast)
          }
        })
      },
      setFeed: function(podcast) {
        this.feed = podcast.url
        this.currentPodcast = podcast
        this.getFeed()
      },
      frameLooper: function () {
        window.webkitRequestAnimationFrame(this.frameLooper);
        const fbc_array = new Uint8Array(this.analyser.frequencyBinCount);
        this.analyser.getByteFrequencyData(fbc_array);
        this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
        this.ctx.fillStyle = "#00CCFF";
        let bars = 100;
        let bar_x, bar_width, bar_height = 0
        for (var i = 0; i < bars; i++){
            bar_x = i * 3;
            bar_width = 2;
            bar_height = -(fbc_array[i]/2);
            this.ctx.fillRect(bar_x, this.canvas.height, bar_width, bar_height);
        }
      }
    },
    mounted () {
      this.setMediaControls()
    },
    created () {
    }
  }
</script>

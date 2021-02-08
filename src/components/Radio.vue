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
          Welcome to Pocket Radio
        </h1>

        <p class="subheading font-weight-regular text-center">
          Add an online radio station name and URL to add to your favorites!
          <v-text-field type="text" placeholder="Enter stream Name" v-model="streamName" label="Stream Name" />
          <v-text-field type="url" placeholder="Enter stream URL" v-model="url" label="Stream URL" />
          <v-btn v-on:click="addStation" color="blue">Add Station</v-btn>
        </p>
        <p class="subheading font-weight-regular text-center">
          Add an online podcast feed name and URL to add to your favorites!
          <v-text-field type="text" placeholder="Enter podcast name" v-model="podcastName" label="Podcast Name" />
          <v-text-field type="url" placeholder="Enter podcast RSS feed" v-model="feed" label="Podcast Feed" />
          <v-btn class="ma-md-4" v-on:click="addFeed" color="blue">Add Feed</v-btn>

        </p>
        <v-row class="text-center">
          <v-btn class="ma-md-4" v-on:click="playAudio" v-if="!isPlaying">Play</v-btn>
          <v-btn v-on:click="stopAudio" v-else color="red">Stop</v-btn>
          </v-row>
      </v-col>
    </v-row>
    <v-row class="text-center">
      <v-btn class="pa-md-4 mx-lg-auto" v-for="x in myStations" v-on:click="setAudio(x)" :key="x.name" color="orange"> {{x.name}} </v-btn>
    </v-row>
    <v-row class="text-center">
      <v-btn class="pa-md-4 mx-lg-auto ma-md-4" v-for="x in myPodcasts" v-on:click="setFeed(x)" :key="x.name" color="green"> {{x.name}} </v-btn>
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
  </v-container>
</template>

<script>
import * as JsStore from 'jsstore'
import * as JsStoreWorker from 'jsstore/dist/jsstore.worker.commonjs2'
window.JsStoreWorker = JsStoreWorker
  export default {
    name: 'Radio',

    data: () => ({
      isPlaying: false,
      audio: {},
      url : '',
      streamName: '',
      feed: '',
      currentPodcast: {},
      selectedEpisode: {},
      connection: {},
      favoritePodcasts: [],
      skipTime: 10,
      myPodcasts: [],
      myStations: [],
      podcastName: ''
    }),
    methods: {
      setMediaControls: function () {
        if ('mediaSession' in navigator) {
          navigator.mediaSession.metadata = new window.MediaMetadata({
            title: 'Pocket Radio',
            artist: 'J Computer Solutions LLC',
            album: 'Pocket Radio',
            artwork: [
              { src: 'https://radio.jcompsolu.com/images/logo-512.png', sizes: '512x512', type: 'image/png' },
            ]
          });

          navigator.mediaSession.setActionHandler('play', this.playAudio());
          navigator.mediaSession.setActionHandler('pause', this.pauseAudio());
          navigator.mediaSession.setActionHandler('stop', this.stopAudio());
          navigator.mediaSession.setActionHandler('seekbackward', () => {
           // User clicked "Seek Backward" media notification icon.
           this.audio.currentTime = Math.max(this.audio.currentTime - this.skipTime, 0);
          });

          navigator.mediaSession.setActionHandler('seekforward', () => {
           // User clicked "Seek Forward" media notification icon.
           this.audio.currentTime = Math.min(this.audio.currentTime + this.skipTime,
                         this.audio.duration);
          });
          navigator.mediaSession.setActionHandler('seekto', function() { /* Code excerpted. */ });
          navigator.mediaSession.setActionHandler('previoustrack', function() { /* Code excerpted. */ });
          navigator.mediaSession.setActionHandler('nexttrack', function() { /* Code excerpted. */ });
        }
      },
      playPodcast: function () {
        this.setAudio(this.currentPodcast)
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
        navigator.mediaSession.metadata.artist = 'Pocket Radio'
        if(preset.image) {
          navigator.mediaSession.metadata.artwork = [
            { src: preset.image }
          ]
        }
        this.playAudio()
      },
      addFeed: function () {
        const pod = {url: this.feed, name: this.podcastName}
        this.addToPodcasts(pod)
        this.feed = ''
        this.podcastName = ''
      },
      addStation: function () {
        const radio = {url: this.url, name: this.streamName}
        this.addToStations(radio)
        this.url = ''
        this.streamName = ''
      },
      getFeed: function () {
        this.favoritePodcasts = []
        let author = ''
        fetch(this.feed)
        .then(response => response.text())
        .then(str => new window.DOMParser().parseFromString(str, "text/xml"))
        .then(data => {
          const items = data.querySelectorAll("item");
          for (let i = 0; i < items.length; i++) {
            let item = items[i];
            let image = item.getElementsByTagName("itunes:image")[0].getAttribute("href")
            let title = item.querySelector("title").innerHTML.replace("<![CDATA[", "").replace("]]>", "")
            if(item.getElementsByTagName("dc:creator").length >0){
               author = item.getElementsByTagName("dc:creator")[0].innerHTML.replace("<![CDATA[", "").replace("]]>", "")
            } else {
               author = item.getElementsByTagName("itunes:author")[0].innerHTML
            }
            let url = item.querySelector("enclosure").getAttribute("url")
            let podcast = { name: title, url: url, image: image, author: author }
            this.favoritePodcasts.push(podcast)
          }
        })
      },
      setFeed: function(podcast) {
        this.feed = podcast.url
        this.currentPodcast = podcast
        this.getFeed()
      },
      getDbSchema: function () {
        const tblPodcasts = {
          name: 'Podcasts',
          columns: {
              // Here "Id" is name of column
              id:{ primaryKey: true, autoIncrement: true },
              name:  { notNull: true, dataType: "string" },
              url:  { notNull: true, dataType: "string" },
          }
        };
        const tblStations = {
          name: 'Stations',
          columns: {
            id:{ primaryKey: true, autoIncrement: true },
            name:  { notNull: true, dataType: "string" },
            url:  { notNull: true, dataType: "string" }
          }
        }
        const db = {
            name: 'pocketPodcasts',
            tables: [tblPodcasts, tblStations]
        }
        return db
      },
      getPodcasts: async function (){
        this.myPodcasts = await this.connection.select({
            from: "Podcasts"
        })
      },
      getStations: async function (){
        this.myStations = await this.connection.select({
            from: "Stations"
        })
      },
      addToPodcasts: async function (podcast) {
        const noOfRowsInserted = await this.connection.insert({
            into: "Podcasts",
            values: [podcast], //you can insert multiple values at a time
        });
        if (noOfRowsInserted > 0) {
            this.myPodcasts.push(podcast)
        }
      },
      addToStations: async function (station) {
        const noOfRowsInserted = await this.connection.insert({
            into: "Stations",
            values: [station], //you can insert multiple values at a time
        });
        if (noOfRowsInserted > 0) {
            this.myStations.push(station)
        }
      }
    },
    mounted () {
      this.setMediaControls()
    },
    async created () {
      this.connection = new JsStore.Connection()
      const database = this.getDbSchema();
      const isDbCreated = await this.connection.initDb(database);
      if(isDbCreated===true){
          this.getPodcasts()
          this.getStations()
      }
      else {
          this.getPodcasts()
          this.getStations()
      }

    }
  }
</script>

<template>
  <v-app>
    <v-app-bar
      app
      color="primary"
      dark
    >
      <div class="d-flex align-center">
        <h2>Pocket Podcaster</h2>
      </div>
      <v-spacer></v-spacer>
    </v-app-bar>

    <v-main>
      <Podcaster/>
    </v-main>
  </v-app>
</template>

<script>
import Podcaster from './components/Podcaster';

export default {
  name: 'App',

  components: {
    Podcaster,
  },

  data: () => ({
    //
  }),
};
</script>

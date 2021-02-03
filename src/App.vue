<template>
  <v-app>
    <v-app-bar
      app
      color="primary"
      dark
    >
      <div class="d-flex align-center">
        <h2>Pocket Radio</h2>
      </div>
      <v-spacer></v-spacer>
    </v-app-bar>

    <v-main>
      <Radio/>
    </v-main>
  </v-app>
</template>

<script>
import Radio from './components/Radio';

export default {
  name: 'App',

  components: {
    Radio,
  },

  data: () => ({
    //
  }),
};
</script>

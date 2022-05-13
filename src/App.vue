<template>
  <v-app>
    <v-app-bar app absolute color="white" dark>
      <v-row>
        <v-col cols="3">
          <div class="d-flex">
            <img
              alt="Vuetify Name"
              class="shrink mt-1"
              contain
              style="width: 150px"
              src="../src/assets/logo_obotrons.png"
            />
          </div>
        </v-col>
        <v-col cols="7" style="text-align: center; align-self: center">
          <v-btn style="color: red" text @click="$router.push('/')">
            DashBoard
          </v-btn>
          <v-btn style="color: red" text @click="$router.push('/Config')">
            Config
          </v-btn>
        </v-col>
        <v-col cols="2" style="align-self: center">
          <span class="mr-2" style="color: red">{{ date }}</span>
        </v-col>
      </v-row>
    </v-app-bar>

    <v-main>
      <v-row>
        <v-col cols="10">
          <router-view />
        </v-col>
      </v-row>
    </v-main>
  </v-app>
</template>

<script>
import moment from "moment";

export default {
  name: "App",

  components: {},

  data: () => ({
    date: "",
  }),
  mounted() {
    setInterval(() => {
      this.date = moment(Date()).format("DD MMMM  YYYY | HH:mm:ss");
    }, 1000);
  },
};
</script>


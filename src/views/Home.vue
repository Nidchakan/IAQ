<template>
  <div class="home">
   <v-row >
    
     <v-col cols="12"> <DashBoard/></v-col>
     
   </v-row>
  
  </div>
</template>

<script>
// @ is an alias to /src
import DashBoard from '@/components/Dashboard.vue'

export default {
  name: 'Home',
  components: {
    DashBoard
}
}
</script>

<style >
.row{
  margin: 0px;
}
</style>

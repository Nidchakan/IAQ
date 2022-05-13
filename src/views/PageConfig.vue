<template>
  <div class="home">
   <v-row >
    
     <v-col cols="12"> <Config/></v-col>
     
   </v-row>
  
  </div>
</template>

<script>
// @ is an alias to /src
import Config from '@/components/Config.vue'

export default {
  name: 'Home',
  components: {
    Config
}
}
</script>

<style >
.row{
  margin: 0px;
}
</style>

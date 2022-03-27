<template>
  <div class="about container">
    <h1>This is an about page</h1>
    <router-link 
      :to="{query: {seccion: 1}}" 
      class="btn btn-primary btn-block">1</router-link>
      <router-link 
      :to="{query: {seccion: 2}}" 
      class="btn btn-primary btn-block">2</router-link>
  </div>
</template>

<script>
  export default {
    data() {
      return {
      
      }
    },
    watch: {
      "$route.query.seccion":{
        immediate: true,
        handler(seccion){
          console.log(`Seccion a cambiado: ${seccion}`);
        }
      }
    }
    
  }
</script>
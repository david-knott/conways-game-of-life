<template>
  <div class="home">
  <Torus />
    <HelloWorld msg="Welcome to Your Vue.js App"/>
  </div>
</template>

<script>
// @ is an alias to /src
import HelloWorld from '@/components/HelloWorld.vue';
import Torus from '@/components/Torus.vue';

export default {
  name: 'Home',
  components: {
    HelloWorld,
    Torus
  }
}
</script>

<template>
  <div>
    <NavBar />
    <RouterView />
    <!-- <Footer /> -->
  </div>
</template>

<script>
import { RouterView } from 'vue-router'
import NavBar from '../components/NavBar.vue'
import Footer from '../components/Footer.vue'
export default {
  name: 'MainLayout',
  components: {
    NavBar,
    RouterView,
    Footer
  }
}
</script>

<style scoped></style>

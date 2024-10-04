<template lang="">
  <!-- declare Navbar component dan properti nya -->
   <NavBar :name=userName :role=roleId />
  <div>
    {{ userName }}
  </div>
</template>
<script>
import router from '@/router';
import NavBar from '@/components/NavBar.vue';

export default {
  // declare komponen navbar 
  components: {
    NavBar
  },
  data() {
    return {
      // Mengembalikan objek data untuk komponen dan diisi dari local
      userName: '',
      roleId: ''
    }
  },
  mounted() {
    // mengambil dari local
    this.userName = localStorage.getItem('name')
    this.roleId = localStorage.getItem('role_id')
    // cek login user
    if (!this.userName) {
      router.push({name: 'login'})
    }
  },
}
</script>
<style>
  
</style>
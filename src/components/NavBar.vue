<template lang="">
  <!-- v-if untuk menyembunyikan navbar ketika berada pada route login -->
  <header v-if="$route.name != 'login'">
  <nav class="navbar navbar-expand-lg bg-body-tertiary navbar-light bg-light shadow-sm">
    <div class="container-fluid">

      <!-- Navbar Toggle for Mobile View -->
      <button 
        class="navbar-toggler" 
        type="button" 
        data-bs-toggle="collapse" 
        data-bs-target="#navbarSupportedContent"  
        aria-controls="navbarSupportedContent" 
        aria-expanded="false" 
        aria-label="Toggle navigation"
      >
        <span class="navbar-toggler-icon"></span>
      </button>

      <!-- Navbar Links -->
      <div class="collapse navbar-collapse" id="navbarSupportedContent">
        <ul class="navbar-nav me-auto mb-2 mb-lg-0">
          <li class="nav-item me-3">
            <RouterLink class="nav-link active" to="/">Home</RouterLink>
          </li>
          <!-- Conditional Menu based on Role -->
          <li v-if="role == 4 || role == 1" class="nav-item me-3">
            <RouterLink class="nav-link" to="/order">Order</RouterLink>
          </li>
          <li class="nav-item me-3">
            <RouterLink class="nav-link" to="/order-list">Order List</RouterLink>
          </li>
          <li v-if="role == 4" class="nav-item me-3">
            <RouterLink class="nav-link" to="/order-report">Report Order</RouterLink>
          </li>
          <li v-if="role == 4" class="nav-item me-3">
            <RouterLink class="nav-link" to="/product">Product</RouterLink>
          </li>
        </ul>

        <!-- Right-side User Info and Logout -->
        <ul class="navbar-nav ms-auto d-flex align-items-center">
          <li class="nav-item me-4">
            <span class="text-secondary">Hi, {{ name }}!</span>
          </li>
          <li class="nav-item">
            <a href="#" class="btn btn-outline-danger" @click.prevent="logout()">Logout</a>
          </li>
        </ul>
      </div>
    </div>
  </nav>
</header>

</template>

<script>

import axios from 'axios';
import router from '@/router';

export default {
    name: 'NavBar',

   // menerima data dari parent component
    props: {
      name: {
        type: String,
        required: true
      },
      role: {
        type: [String, Number],
        required: true
      }
    },

    data(){
      return {
        tokenCheckInterval: null
      }
    },

    created() {
      // Set up periodic token check
      this.startTokenCheck();
    },

    beforeUnmount() {
      // Clean up interval when component is destroyed
      this.stopTokenCheck();
    },
    
    methods: {
    checkTokenExpiration() {
      const expiresAt = localStorage.getItem('token_expires_at');
      if (!expiresAt) return true;

      const now = new Date().getTime();
      const expiration = new Date(expiresAt).getTime();
      
      // If token will expire in less than 5 minutes, try to refresh it
      const fiveMinutes = 5 * 60 * 1000;
      if ((expiration - now) < fiveMinutes) {
        this.refreshToken();
      }
      
      return now >= expiration;
    },

    startTokenCheck() {
      // Check token expiration every minute
      this.tokenCheckInterval = setInterval(() => {
        if (this.checkTokenExpiration()) {
          this.handleLogout('Your session has expired. Please log in again.');
        }
      }, 3600000); // 60000 ms = 1 minute & 3600000 ms = 1 hour
    },

    stopTokenCheck() {
      if (this.tokenCheckInterval) {
        clearInterval(this.tokenCheckInterval);
      }
    },

    async refreshToken() {
      try {
        const response = await axios.post('http://localhost/food-order-app/public/api/auth/refresh-token', {}, {
          headers: {
            "Authorization": `Bearer ${localStorage.getItem('token')}`
          }
        });

        // Update token and expiration time
        localStorage.setItem('token', response.data.token);
        localStorage.setItem('token_expires_at', response.data.token_expires_at);
      } catch (error) {
        console.error('Failed to refresh token:', error);
        // If refresh fails, force logout
        this.handleLogout('Session expired. Please login again.');
      }
    },

    async logout() {
      try {
        await axios.get('http://localhost/food-order-app/public/api/auth/logout', {
          headers: {
            "Authorization": `Bearer ${localStorage.getItem('token')}`
          }
        });
        this.handleLogout();
      } catch (error) {
        if (error.response?.status === 401) {
          this.handleLogout('Your session has expired. Please log in again.');
        } else {
          console.error(error);
          this.handleLogout('An error occurred during logout.');
        }
      }
    },

    handleLogout(message = null) {
      // Clear all local storage items
      this.clearLocalStorage();
      
      // Stop token check interval
      this.stopTokenCheck();
      
      // Show message if provided
      if (message) {
        alert(message);
      }
      
      // Redirect to login
      router.push({ name: 'login' });
    },

    clearLocalStorage() {
      localStorage.removeItem('email');
      localStorage.removeItem('name');
      localStorage.removeItem('role_id');
      localStorage.removeItem('token');
      localStorage.removeItem('token_expires_at');
    }
  }
}
</script>

<style scoped>
.navbar {
  border-bottom: 1px solid #e9ecef;
}

.nav-link {
  transition: color 0.2s ease-in-out;
}

.nav-link:hover {
  color: #0d6efd;
}
</style>
<template lang="">
    <div class="container mt-5">
        <div class="row vh-100 justify-content-center align-items-center">
            <div class="col-md-6">
                <div class="card">
                    <div class="card-header">
                        <h3 class="text-center">Login</h3>
                    </div>
                    <div class="card-body">
                        <!-- mencegah refresh halaman -->
                        <form @submit.prevent>
                            <div class="mb-3">
                                <label for="email" class="form-label">Email</label>
                                <input
                                    type="email"
                                    v-model="email"
                                    class="form-control"
                                    id="email"
                                    placeholder="Enter your email"
                                    required
                                />
                            </div>
                            <div class="mb-3">
                                <label for="password" class="form-label">Password</label>
                                <input
                                    type="password"
                                    v-model="password"
                                    class="form-control"
                                    id="password"
                                    placeholder="Enter your password"
                                    required
                                />
                            </div>
                            <div class="mb-3 form-check">
                                <input
                                    type="checkbox"
                                    class="form-check-input"
                                    id="rememberMe"
                                />
                                <label class="form-check-label" for="rememberMe"
                                    >Remember me</label
                                >
                            </div>
                            <div class="d-grid">
                                <button @click="login()" class="btn btn-primary form-control">Login</button>
                            </div>
                        </form>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>
<script>
import router from '@/router';
import axios from 'axios';

export default {
data() {
    return {
        email:"",
        password:""
    }
},
methods: {
    login(){
        // menggunakan axios third party JS untuk menarik API
        axios.post('http://localhost/food-order-app/public/api/auth/login', {
            email: this.email,
            password: this.password
        })
        .then(function (response) {
            localStorage.setItem('email', response.data.data.email)
            localStorage.setItem('name', response.data.data.name)
            localStorage.setItem('role_id', response.data.data.role_id)
            localStorage.setItem('token', response.data.data.token)
            router.push({name: 'home'})
        })
        .catch(function (error) {
            console.log(error);
        });
    }
},
mounted() {
    this.userName = localStorage.getItem('name')
    if (this.userName) {
        router.push({name: 'home'})
    }
},
}
</script>
<style lang=""></style>

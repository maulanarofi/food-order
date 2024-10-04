<template lang="">
    <NavBar :name=userName :role=roleId />
    <div class="container mt-3">
        <div class="col-12 col-lg-6">
            <h3 class="mb-3">Add New Product</h3>

            <!-- Menggunakan method baru -->
            <form @submit.prevent="createProduct()">
                <div class="mb-3">
                    <label class="mb-1" for="name">Name</label>
                    <input type="text" class="form-control" v-model="name" required placeholder="Product Name" />
                </div>
                <div class="mb-3">
                    <label class="mb-1" for="price">Price</label>
                    <input type="number" class="form-control" v-model="price" required placeholder="Product Price" />
                </div>
                <div class="mb-3">
                    <label class="mb-1" for="image">Image</label>
                    <!--  memanggil method imageChanged setiap kali ada perubahan pada input file, yaitu ketika pengguna memilih file baru. -->
                    <input type="file" class="form-control" @change="imageChanged($event)" id="image" />
                </div>
                <div class="mb-3">
                    <button class="btn btn-primary form-control" type="submit">Save</button>
                </div>
            </form>
        </div>
    </div>
</template>

<script>

import NavBar from '@/components/NavBar.vue';
import axios from 'axios';
import router from '@/router';

export default {
    components: {
        NavBar
    },
    data() {
    return {
      userName: '',
      roleId: '',
      items: [],
      url: "http://localhost/food-order-app/public/storage/items/",
      name: "",
      price: "",
      file: ""
    }
  },
  mounted() {
    this.userName = localStorage.getItem('name')
    this.roleId = localStorage.getItem('role_id')
    if (!this.userName) {
      router.push({name: 'login'})
    }
    if (this.roleId != 4) {
        router.push({name: 'home'})
    }
  },
  methods: {
    // Declare methods baru
    createProduct(){
        // Validasi sederhana jika form kosong
        if (this.name == '' || this.price == '') {
            alert('data cannot empty')
            return;
        }

        // membuat variable untuk menambahkan data
        let formData = new FormData();
        formData.append('name', this.name)
        formData.append('price', this.price)
        formData.append('image_file', this.file)

        axios.post('http://localhost/food-order-app/public/api/item', formData, {
            headers: {
                'Authorization': `Bearer ${localStorage.getItem('token')}`,
                
            }
			})
			.then((response) => {
                router.push({ name: 'product' })
			})
			.catch(function (error) {
				console.log(error);
                console.log('error fetch items')
			});
    },
    // Declare methods
    imageChanged(e){
        let file = e.target.files[0]
        this.file = file
    }
  },

}
</script>
<style>
    
</style>
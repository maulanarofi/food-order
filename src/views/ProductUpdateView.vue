<template lang="">
    <NavBar :name=userName :role=roleId />
    <div class="container mt-3">
        <div class="col-12 col-lg-6">
            <h3 class="mb-3">Edit Product</h3>

            <!-- Menggunakan method baru -->
            <form @submit.prevent="updateProduct()">
                <div class="mb-3">
                    <label class="mb-1" for="name">Name</label>
                    <input type="text" class="form-control" v-model="item.name" required placeholder="Product Name" />
                </div>
                <div class="mb-3">
                    <label class="mb-1" for="price">Price</label>
                    <input type="number" class="form-control" v-model="item.price" required placeholder="Product Price" />
                </div>
                <div class="mb-3">
                    <label class="mb-1">Current Image</label> <br>
                    <img v-if="item.image" :src="url+item.image" style="width: 100px; height:100px" class="object-fit-cover">
                    <img v-else src="@/assets/images/no-image.jpg" style="width: 100px; height:100px" class="object-fit-cover">
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
import router from '@/router';
import axios from 'axios';

export default {
    components: {
    NavBar
  },
  data() {
    return {
      userName: '',
      roleId: '',
      url: "http://localhost/food-order-app/public/storage/items/",
      productId: '',
      item: '',
      file: ''
    }
  },
  mounted() {
    this.productId = this.$route.params.productId
    this.userName = localStorage.getItem('name')
    this.roleId = localStorage.getItem('role_id')
    if (!this.userName) {
      router.push({name: 'login'})
    }
    if (this.roleId != 4) {
        router.push({name: 'home'})
    }
    this.showItems()
  },
  methods: {
    showItems(){
        axios.get('http://localhost/food-order-app/public/api/item/'+this.productId, {
            headers: {
                'Authorization': `Bearer ${localStorage.getItem('token')}`
            }
			})
			.then((response) => {
            this.item = response.data.data
			})
			.catch(function (error) {
				console.log(error);
            console.log('error fetch items')
			});
    },
    updateProduct(){
      // Validasi form tidak boleh kosong
      if (this.name == '' || this.price == '') {
            alert('data cannot empty')
            return;
        }

        // membuat variable untuk menambahkan data
        let formData = new FormData();
        formData.append('name', this.item.name)
        formData.append('price', this.item.price)
        formData.append('image_file', this.file)
        // Menggunakan method patch disini untuk mengupdate
        formData.append('_method', 'patch')

        axios.post('http://localhost/food-order-app/public/api/item/'+this.productId, formData, {
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
    imageChanged(e){
        let file = e.target.files[0]
        this.file = file
    }
  },
}
</script>
<style>
    
</style>
<template>
  <NavBar :name="userName" :role="roleId" />
  <div class="container mt-5">
    <h2 class="text-center">Product List</h2>

    <div class="row mb-3 mt-3">
      <div class="col-md-6">
        <a href="/product-add" class="btn btn-success">Add Product</a>
      </div>
      <!-- Membuat number of rows -->
      <div class="col-md-2">
        <select v-model="rowsPerPage" @change="updateRowsPerPage" class="form-select">
          <option value="5">5 rows</option>
          <option value="10">10 rows</option>
          <option value="20">20 rows</option>
        </select>
      </div>
      <!-- Membuat fitur search -->
      <div class="col-md-4">
        <input v-model="keyword" @input="search" class="form-control" placeholder="Search products...">
      </div>
    </div>

    <table class="table table-hover mt-3">
      <thead class="table-info">
        <tr>
          <th>No.</th>
          <th>Product Name</th>
          <th>Price</th>
          <th>Image</th>
          <th>Actions</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(item, index) in paginatedItems" :key="item.id">
          <td>{{ (currentPage - 1) * rowsPerPage + index + 1 }}</td>
          <td>{{ item.name }}</td>
          <td>{{ `Rp ${item.price}` }}</td>
          <td>
            <img v-if="item.image" :src="url + item.image" style="width: 100px; height:100px" class="object-fit-cover">
            <img v-else src="@/assets/images/no-image.jpg" style="width: 100px; height:100px" class="object-fit-cover">
          </td>
          <td>
            <RouterLink :to="{ name: 'productUpdate', params: { productId: item.id }}" class="btn btn-primary btn-sm">
              Edit
            </RouterLink>
            <button class="btn btn-danger btn-sm">Delete</button>
          </td>
        </tr>
      </tbody>
    </table>

    <nav aria-label="Page navigation">
      <ul class="pagination justify-content-center">
        <li class="page-item" :class="{ disabled: currentPage === 1 }">
          <a class="page-link" href="#" @click.prevent="changePage(currentPage - 1)">Previous</a>
        </li>
        <li v-for="page in totalPages" :key="page" class="page-item" :class="{ active: page === currentPage }">
          <a class="page-link" href="#" @click.prevent="changePage(page)">{{ page }}</a>
        </li>
        <li class="page-item" :class="{ disabled: currentPage === totalPages }">
          <a class="page-link" href="#" @click.prevent="changePage(currentPage + 1)">Next</a>
        </li>
      </ul>
    </nav>
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
      items: [],
      url: "http://localhost/food-order-app/public/storage/items/",
      keyword: '',
      rowsPerPage: 10,
      currentPage: 1
    }
  },
  computed: {
    filteredItems() {
      return this.items.filter(item =>
        item.name.toLowerCase().includes(this.keyword.toLowerCase())
      )
    },
    totalPages() {
      return Math.ceil(this.filteredItems.length / this.rowsPerPage)
    },
    paginatedItems() {
      const start = (this.currentPage - 1) * this.rowsPerPage
      const end = start + this.rowsPerPage
      return this.filteredItems.slice(start, end)
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
    this.getItems()
  },
  methods: {
    getItems() {
      axios.get('http://localhost/food-order-app/public/api/item', {
        headers: {
          'Authorization': `Bearer ${localStorage.getItem('token')}`
        }
      })
      .then((response) => {
        this.items = response.data.data
      })
      .catch(function (error) {
        console.log(error);
        console.log('error fetch items')
      });
    },
    search() {
      this.currentPage = 1
    },
    updateRowsPerPage() {
      this.currentPage = 1
    },
    changePage(page) {
      if (page >= 1 && page <= this.totalPages) {
        this.currentPage = page
      }
    }
  }
}
</script>
<style scoped>
.edit-btn {
  transition: all 0.3s ease;
}

.edit-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
}
</style>
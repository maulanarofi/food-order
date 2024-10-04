<template lang="">
    <div>
        <!-- memanggil komponen navbar beserta props nya -->
        <NavBar :name=userName :role=roleId />

        <div class="container-fluid mt-4">
            <div class="row">    
                <!-- Item List -->
                 <div class="col-12 col-sm-8 mb-3">
                    <!-- Search Box untuk pencarian-->
                     <div class="col-12 mb-3">
                        <input type="text" v-model="keyword" class="form-control" placeholder="Search Here" :onchange="searchItem()" />
                     </div>

                     <hr/>

                     <!-- Item List Box -->
                      <div class="col-12">
                        <div class="row">
                            <div v-for="item in filteredItems" class="col-12 col-sm-6 col-md-4 col-lg-3 mb-3">
                                <div class="card">
                                    <img :src="url+item.image" height="200px" class="card-img-top object-fit-cover" alt="...">
                                    <div class="card-body text-center">
                                        <h5 class="card-title">{{ item.name }}</h5>
                                        <p class="card-text">{{ `Rp. ${item.price}` }}</p>
                                        <p><button class="btn btn-success" @click="orderItem(item.id)">Order</button></p>
                                    </div>
                                </div>
                            </div>
                            
                        </div>
                      </div>
                 </div>
                <!-- Ordered List -->
                 <div class="col-12 col-sm-4 mb-3 order-box">
                    <h2>Order List</h2>
                    <div class="mb-3">
                        <label for="customerName" class="form-label">Customer Name</label>
                        <input type="text" class="form-control" v-model="customerName" id="customerName" placeholder="Enter your name" required>
                    </div>
                    <div class="mb-3">
                        <label for="tableNo" class="form-label">Table No.</label>
                        <input type="number" class="form-control" id="tableNo" v-model ="tableNo" placeholder="Enter your name">
                    </div>
                    <hr>
                    <div class="item-box">
                        <!-- <div v-for="order in orders" class="d-flex justify-content-between"> -->
                            <table class="table table-borderless">
                                <thead>
                                    <tr>
                                        <th>Name</th>
                                        <th>Qty</th>
                                        <th>Price</th>
                                    </tr>
                                </thead>
                                <tbody>
                                    <tr v-for="order in orders">
                                        <td>
                                            {{ order.name }} <br>
                                            <span style="font-size: 12px" class="text-muted">@Rp{{ order.eachPrice }}</span> <br>
                                            <button class="btn btn-sm btn-outline-secondary me-2" @click="decreaseItemQty(order)">-</button>
                                            <button  class="btn btn-sm btn-outline-secondary me-2" @click="increaseItemQty(order)">+</button>
                                            <button class="btn btn-sm btn-outline-danger" @click="deleteItem(order)">delete</button>
                                        </td>
                                        <td>{{ order.qty }}</td>
                                        <td>Rp.{{ order.price }}</td>
                                    </tr>
                                </tbody>
                            </table>
                            <!-- <span>{{ order.name }}</span>
                            <span>{{ order.qty }}</span>
                            <span>Rp {{ order.price }}</span> -->
                        <!-- </div> -->
                    </div>
                    <hr>
                    <div class="total-box">
                        <div class="d-flex justify-content-between">
                            <span>Total</span>
                            <span>Rp {{ total }}</span>
                        </div>
                    </div>
                    <div class="mt-3">
                        <button class="btn btn-success form-control" :disabled=processing @click="submitOrder()">{{ processing ? 'Processing Order...' : 'Submit' }}</button>
                    </div>
                 </div>
            </div>
        </div>
    </div>
</template>
<script>

import axios from 'axios';
import router from '@/router';
import NavBar from '@/components/NavBar.vue';

export default {
    components: {
    NavBar
  },
  data() {
    return {
      userName: '',
      roleId: '',
      items: [],
      filteredItems: [],
      keyword: '',
      url: "http://localhost/food-order-app/public/storage/items/",
      orders: [],
      total: 0,
      customerName: '',
      tableNo: '',
      processing: false
    }
  },
  mounted() {
    this.userName = localStorage.getItem('name')
    this.roleId = localStorage.getItem('role_id')
    if (!this.userName) {
      router.push({name: 'login'})
    }
    if (this.roleId != 4 && this.roleId != 1) {
        router.push({name: 'home'})
    }
    this.getItems()
  },
  methods: {
    getItems(){
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
    searchItem(){
        this.filteredItems = this.items.filter(item => item.name.toLowerCase().includes(this.keyword.toLowerCase()))
    },
    orderItem(id){
        let item = this.filteredItems.filter(item => item.id == id)[0]
        let list = Object.assign({}, item)
        list.eachPrice = item.price
        let indexOfArrayItem = this.orders.map(e => e.id).indexOf(list.id)

        if (indexOfArrayItem != -1) { //Jika item yang diorder ada di array orders, maka tambah qty & kalikan harga
            this.orders[indexOfArrayItem].qty++
            this.orders[indexOfArrayItem].price = this.orders[indexOfArrayItem].eachPrice * this.orders[indexOfArrayItem].qty
        }
        else{ //Jika item yang diorder belum ada di array orders, maka push item ke dalam array orders dengan qty 1
            list.qty = 1
            this.orders.push(list)
        }

        let listPrice = this.orders.map(order => order.price)
        let priceTotal = 0

        listPrice.forEach(price => {
            priceTotal += price
        })

        this.total = priceTotal
    },
    decreaseItemQty(item){

        //rule bates qty hanya 1
        if (item.qty <= 1) {
            return
        }

        //pengurangan qty
        let indexOfArrayItem = this.orders.map(e => e.id).indexOf(item.id)
        this.orders[indexOfArrayItem].qty--
        //hitung ulang price nya
        this.orders[indexOfArrayItem].price = this.orders[indexOfArrayItem].eachPrice * this.orders[indexOfArrayItem].qty
        console.log(indexOfArrayItem)

        //hitung ulang total harga
        let orderPrice = this.orders.map(order => order.price)
        let priceTotal = 0
        orderPrice.forEach(price => {
            priceTotal += price
        })

        this.total = priceTotal
    },
    increaseItemQty(item){

        //penambahan qty
        let indexOfArrayItem = this.orders.map(e => e.id).indexOf(item.id)
        this.orders[indexOfArrayItem].qty++
        //hitung ulang price nya
        this.orders[indexOfArrayItem].price = this.orders[indexOfArrayItem].eachPrice * this.orders[indexOfArrayItem].qty
        console.log(indexOfArrayItem)

        //hitung ulang total harga
        let orderPrice = this.orders.map(order => order.price)
        let priceTotal = 0
        orderPrice.forEach(price => {
            priceTotal += price
        })

        this.total = priceTotal
    },
    deleteItem(item){
        let indexOfArrayItem = this.orders.map(e => e.id).indexOf(item.id)
        this.orders.splice(indexOfArrayItem, 1)

        let orderPrice = this.orders.map(order => order.price)
        let priceTotal = 0
        orderPrice.forEach(price => {
            priceTotal += price
        })

        this.total = priceTotal
    },
    submitOrder(){
        if (this.customerName == '' && this.tableNo == '') {
            alert('Customer Name dan Table No cannot be empty!')
        }

        if (this.orders == '') {
            alert('Silahkan pilih pesanan terlebih dahulu!')
        }

        let item = this.orders.map(item => {
            return {
                "id": item.id,
                "qty": item.qty
            }
        })

        this.processing = true

        axios.post('http://localhost/food-order-app/public/api/order', {
            'customer_name' : this.customerName,
            'table_no' : this.tableNo,
            'items' : item
        }, {
            headers: {
                'Authorization': `Bearer ${localStorage.getItem('token')}`
            }
			})
			.then((response) => {
                console.log(response)
                alert('Order saved successfully!')
                this.orders = []
                this.customerName = ''
                this.tableNo = ''
                this.total = 0
			})
			.catch(function (error) {
				console.log(error);
                console.log('error fetch items')
			})
            .finally(() => this.processing = false);
    }
  },
}
</script>
<style>
    .order-box{
        border-left: solid 1px #ccc
    }
    .item-box{
        font-size: 16px;
    }
    .total-box{
        font-size: 18px;
        font-weight: bold;
    }
</style>
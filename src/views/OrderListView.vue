<template lang="">
    <div>
        <NavBar :name=userName :role=roleId />
        <div class="container mt-5">
            <h2 class="text-center">Order List</h2>

            <table class="table table-striped mt-3">
            <thead>
                <tr>
                    <th scope="col">No.</th>
                    <th scope="col">Customer Name</th>
                    <th scope="col">No Table</th>
                    <th scope="col">Order Date</th>
                    <th scope="col">Time</th>
                    <th scope="col">Status</th>
                    <th scope="col">Total</th>
                    <th scope="col">Waiters</th>
                    <th scope="col">Cashier</th>
                    <th scope="col">Action</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="(order, index) in orders" ::key="index">
                    <th scope="row">{{ index + 1 }}</th>
                    <td>{{ order.customer_name }}</td>
                    <td>{{ order.table_no }}</td>
                    <td>{{ order.order_date }}</td>
                    <td>{{ order.order_time }}</td>
                    <td>{{ order.status }}</td>
                    <td>Rp. {{ order.total }}</td>
                    <td>{{ order.waiters ? order.waiters.name : '' }}</td>
                    <td>{{ order.cashier ? order.cashier.name : '' }}</td>
                    <td>
                        <router-link :to="{ name : 'orderDetail', params : {orderId: order.id}}">View Detail</router-link>
                    </td>
                </tr>
            </tbody>
            </table>
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
            orders: []
        }
    },
    mounted() {
        this.userName = localStorage.getItem('name')
        this.roleId = localStorage.getItem('role_id')

        if (!this.userName) {
        router.push({name: 'login'})
        }

        this.getOrders()
    },
    methods: {
        getOrders(){
            axios.get('http://localhost/food-order-app/public/api/order', {
                headers: {
                    'Authorization': `Bearer ${localStorage.getItem('token')}`
                }
			})
			.then((response) => {
                this.orders = response.data.data
			})
			.catch(function (error) {
				console.log(error);
                console.log('error fetch items')
			});
        }
    }

}
</script>
<style lang="">
    
</style>
<template lang="">
    <div>
        <NavBar :name=userName :role=roleId />
        
        <div class="container mt-5">
            <table class="table table-responsive table-bordered">
                <tbody>
                    <tr>
                        <td>Customer Name : {{ order.customer_name }}</td>
                        <td>Table No : {{ order.table_no }}</td>
                        <td>Order Date : {{ order.order_date }}</td>
                        <td>Status : {{ order.status }}</td>
                    </tr>
                    <tr>
                        <td>Waiters : {{ order.waiters ? order.waiters.name : '-' }}</td>
                        <td>Cashier : {{ order.cashier ? order.cashier.name : '-' }}</td>
                        <td>Order Time : {{ order.order_time }}</td>
                        <td>Grand Total : Rp. {{ order.total }}</td>
                    </tr>
                </tbody>
            </table>

            <hr class="my-5">

            <table class="table table-striped">
                <thead>
                    <tr>
                        <td>#</td>
                        <td>Item Name</td>
                        <td>Price</td>
                        <td>Qty</td>
                        <td>Total</td>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="(item, index) in order.order_detail">
                        <td>{{ index +1 }}</td>
                        <td>{{ item.item ? item.item.name : '-' }}</td>
                        <td>Rp. {{ item.price }}</td>
                        <td>{{ item.qty }}</td>
                        <td>Rp. {{ item.price * item.qty }}</td>
                    </tr>
                </tbody>
            </table>

            <!-- done if user is chef AND order.status == ordered -->
             <!-- paid if user is cashier or manager AND order.status == done -->

            <div class="mt-3">
                <button v-if="(order.status == 'ordered') && (roleId == 2)" class="btn btn-primary" @click="setAsDone(order.id)">Done</button>
                <button v-if="(order.status == 'done') && (roleId == 3 || roleId == 4)" class="btn btn-success" @click="setAsPaid(order.id)">Paid</button>
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
            order: ''
        }
    },
    mounted() {
        this.userName = localStorage.getItem('name')
        this.roleId = localStorage.getItem('role_id')

        if (!this.userName) {
        router.push({name: 'login'})
        }
        this.getOrder()
    },
    methods : {
        getOrder(){
            axios.get('http://localhost/food-order-app/public/api/order/'+this.$route.params.orderId, {
                headers: {
                    'Authorization': `Bearer ${localStorage.getItem('token')}`
                }
			})
			.then((response) => {
                this.order = response.data.data
			})
			.catch(function (error) {
				console.log(error);
                console.log('error fetch items')
			});
        },
        setAsDone(orderId){
            axios.get('http://localhost/food-order-app/public/api/order/'+ orderId + '/set-as-done', {
                headers: {
                    'Authorization': `Bearer ${localStorage.getItem('token')}`
                }
			})
			.then((response) => {
                if (response.status == 200){
                    alert('ubah status order menjadi done berhasil!')
                }
                this.order = response.data.data
			})
			.catch(function (error) {
				console.log(error);
                console.log('error fetch items')
			});
        },
        setAsPaid(orderId){
            axios.get('http://localhost/food-order-app/public/api/order/'+ orderId + '/payment', {
                headers: {
                    'Authorization': `Bearer ${localStorage.getItem('token')}`
                }
			})
			.then((response) => {
                if (response.status == 200){
                    alert('ubah status done menjadi paid berhasil!')
                }
                this.order = response.data.data
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
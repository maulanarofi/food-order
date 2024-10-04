<template lang="">
    <div>
        <NavBar :name=userName :role=roleId />

        <div class="container mt-5">
            <div class="mb-3 w-50">
                <label for="month" class="form-label">Month</label>
                <select class="form-control" id="month" v-model="month" @change="getReport()">
                    <option value="">Chose Month Period</option>
                    <option v-for="month in months" :value=month.value>
                        {{ month.name }}
                    </option>
                </select>
            </div>

            <div class="col-12 mt-5">
                <div class="row">
                    <div class="col-12 col-sm-4">
                        <div class="box text-center">
                            <label>Order Count</label>
                            <label>{{ data.orderCount }}</label>
                        </div>
                    </div>
                    <div class="col-12 col-sm-4">
                        <div class="box text-center">
                            <label>Min Payment</label>
                            <label>Rp. {{ data.minPayment }}</label>
                        </div>
                    </div>
                    <div class="col-12 col-sm-4">
                        <div class="box text-center">
                            <label>Max Payement</label>
                            <label>Rp. {{ data.maxPayment }}</label>
                        </div>
                    </div>
                </div>
            </div>

            <div class="col-12 mt-5">
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
                        </tr>
                    </thead>
                    <tbody>
                        <tr v-for="(order, index) in data.orders" ::key="index">
                            <th scope="row">{{ index + 1 }}</th>
                            <td>{{ order.customer_name }}</td>
                            <td>{{ order.table_no }}</td>
                            <td>{{ order.order_date }}</td>
                            <td>{{ order.order_time }}</td>
                            <td>{{ order.status }}</td>
                            <td>Rp. {{ order.total }}</td>
                            <td>{{ order.waiters ? order.waiters.name : '' }}</td>
                            <td>{{ order.cashier ? order.cashier.name : '' }}</td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>
    </div>
</template>


<script>

import router from '@/router';
import axios from 'axios';
import NavBar from '@/components/NavBar.vue';

export default {
    components: {
        NavBar
    },
    data() {
        return {
            userName: '',
            roleId: '',
            months: [
                { value: 1, name: 'January'},
                { value: 2, name: 'February'},
                { value: 3, name: 'March'},
                { value: 4, name: 'April'},
                { value: 5, name: 'May'},
                { value: 6, name: 'June'},
                { value: 7, name: 'July'},
                { value: 8, name: 'August'},
                { value: 9, name: 'September'},
                { value: 10, name: 'October'},
                { value: 11, name: 'November'},
                { value: 12, name: 'December'},
            ],
            month: '',
            data: {
                orderCount: 0,
                minPayment: 0,
                maxPayment: 0
            }
        }
    },
    mounted() {
        this.userName = localStorage.getItem('name')
        this.roleId = localStorage.getItem('role_id')

        if (!this.userName) {
        router.push({name: 'login'})
        }
    },
    methods: {
        getReport(){
            axios.get('http://localhost/food-order-app/public/api/order-report?month='+this.month, {
                headers: {
                    'Authorization': `Bearer ${localStorage.getItem('token')}`
                }
			})
			.then((response) => {
                // console.log(response)
                this.data = response.data.data
			})
			.catch(function (error) {
				console.log(error);
                console.log('error fetch items')
			});
        }
    }
}
</script>
<style scoped>
    .box {
        border-radius: 4px;
        border: solid 1px;
        padding: 30px;
        font-size: 20px;
    }
    .box label{
        display: block
    }
</style>
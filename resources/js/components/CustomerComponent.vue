<template>
    <div id="customers">
        <div class="card">
                <div class="card-header">
                    <h5><b>Customer List</b>
                        <button @click="refresh" type="button" class="btn btn-danger btn-sm float-end">Refresh</button>
                        <button @click="createCustomer" type="button" class="btn btn-primary btn-sm float-end me-1">Add Customer</button>

                    </h5>
                </div>
                <div class="card-body">
                    <div class="py-2">
                        <div class="row align-items-center pb-2">
                            <div class="col-12 col-lg-2 "><b>Search By</b></div>
                            <div class="col-12 col-lg-4">
                                <select class="form-control" v-model="field">
                                    <option value="name">Name</option>
                                    <option value="email">Email</option>
                                    <option value="phone">Phone</option>
                                    <option value="address">Address</option>
                                </select>
                            </div>
                            <div class="col-12 col-lg-6">
                                <input type="text" v-model="query" class="form-control" placeholder="Search">
                            </div>
                        </div>
                    </div>
                    <div class="table-responsive">
                        <table class="table table-bordered table-striped">
                            <thead>
                              <tr>
                                <th>SI</th>
                                <th>Name</th>
                                <th>Email</th>
                                <th>Phone</th>
                                <th>Address</th>
                                <th>Action</th>
                              </tr>
                            </thead>
                            <tbody>
                              <tr v-show="customers.length" v-for="(customer, index) in customers" :key="customer.id">
                                <td>{{ index + 1}}</td>
                                <td>{{ customer.name }}</td>
                                <td>{{ customer.email }}</td>
                                <td>{{ customer.phone }}</td>
                                <td>{{ customer.address.substr(0, 15) }}</td>
                                <td>
                                    <button @click="showCustomer(customer)" type="button" class="btn btn-info btn-sm">Show</button>
                                    <button @click="editCustomer(customer)" type="button" class="btn btn-primary btn-sm">Edit</button>
                                    <button @click="destroy(customer)" type="button" class="btn btn-danger btn-sm">Delete</button>
                                </td>
                              </tr>
                              <tr v-show="!customers.length">
                                    <td colspan="6">
                                        <div class="alert alert-danger">Sorry, No data found</div>
                                    </td>
                              </tr>

                            </tbody>
                          </table>
                          <pagination-component v-if="pagination.last_page > 1"
                                                :pagination = "pagination"
                                                :offset = "5"
                                                @paginate="query == '' ? getAllCustomers() : searchCustomer() "
                                                    >

                          </pagination-component>
                      </div>
                </div>
        </div>
        <vue-progress-bar></vue-progress-bar>
        <vue-snotify></vue-snotify>
        <!-- Create customer -->
        <!-- store/edit customer Modal -->
        <div class="modal fade" id="createCustomer" data-bs-backdrop="static" data-bs-keyboard="false" tabindex="-1" aria-labelledby="staticBackdropLabel" aria-hidden="true">
        <div class="modal-dialog">
            <div class="modal-content">
            <div class="modal-header">
                <h1 class="modal-title fs-5" id="staticBackdropLabel">{{editMode ? 'edit' : 'Add'}} Customer</h1>
                <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
            </div>
            <form @submit.prevent="editMode ? updateCustomer() : addCustomer()" @keydown="form.onKeydown($event)">
                <AlertError :form="form"></AlertError>
                <div class="modal-body">
                    <div class="mb-3">
                        <input v-model="form.name" type="text" name="name" class="form-control" placeholder="Name">
                        <HasError :form="form" field="name" />
                    </div>
                    <div class="mb-3">
                        <input v-model="form.email" type="email" class="form-control" name="email" placeholder="Email">
                        <HasError :form="form" field="email" />
                    </div>
                    <div class="mb-3">
                    <input v-model="form.phone" type="text" class="form-control" name="phone" placeholder="Phone">
                    <HasError :form="form" field="phone" />
                    </div>
                    <div class="mb-3">
                        <input v-model="form.address" type="text" class="form-control" name="address" placeholder="Address">
                        <HasError :form="form" field="address" />
                    </div>

                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Close</button>
                    <button type="submit" :disabled="form.busy" class="btn btn-primary">Submit</button>
                </div>
                </form>
            </div>
        </div>
        </div>
         <!-- show customer Modal -->
         <div class="modal fade" id="showCustomer" data-bs-backdrop="static" data-bs-keyboard="false" tabindex="-1" aria-labelledby="staticBackdropLabel" aria-hidden="true">
        <div class="modal-dialog">
            <div class="modal-content">
            <div class="modal-header">
                <h1 class="modal-title fs-5" id="staticBackdropLabel">Customer Details</h1>
   
            </div>
                <AlertError :form="form"></AlertError>
                <div class="modal-body">
                    <div class="mb-3">
                        <input v-model="form.name" type="text" name="name" class="form-control" placeholder="Name">
                        <HasError :form="form" field="name" />
                    </div>
                    <div class="mb-3">
                        <input v-model="form.email" type="email" class="form-control" name="email" placeholder="Email">
                        <HasError :form="form" field="email" />
                    </div>
                    <div class="mb-3">
                    <input v-model="form.phone" type="text" class="form-control" name="phone" placeholder="Phone">
                    <HasError :form="form" field="phone" />
                    </div>
                    <div class="mb-3">
                        <input v-model="form.address" type="text" class="form-control" name="address" placeholder="Address">
                        <HasError :form="form" field="address" />
                    </div>

                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Close</button>
                
                </div>
                
            </div>
        </div>
        </div>
    </div>
</template>

<script>
import { Button, HasError, AlertError } from 'vform/src/components/bootstrap5'
    export default {
        components: {
            Button, HasError, AlertError
        },
        data() {
            return {
                editMode: false,
                form: new Form({
                    id: '',
                    name: '',
                    email: '',
                    phone: '',
                    address: '',
                }),
                customers: [],
                pagination: {
                    current_page: 1,
                },
                field: 'name',
                query: ''

            }
        },
        watch:{
            query:function(newQ, old){
                if(newQ === ''){
                    this.getAllCustomers();
                }else{
                    this.searchCustomer();
                }
            }
        },
        mounted() {
            this.getAllCustomers();
        },
        methods:{
            getAllCustomers(){
                this.$Progress.start();
                axios.get('/api/customers?page='+this.pagination.current_page)
                .then(res => {
                    this.customers = res.data.data
                    this.pagination = res.data.meta
                   this.$Progress.finish();
                })
                .catch(e => {
                    console.log(e)
                    this.$Progress.fail();
                })
            },
            searchCustomer(){
                this.$Progress.start();
                axios.get('/api/search/customers/'+this.field+'/'+this.query+'?page='+this.pagination.current_page)
                .then(response => {
                    this.customers = response.data.data;
                    this.pagination = response.data.meta;
                    this.$Progress.finish();
                })
                .catch(e => {
                    console.log(e)
                    this.$Progress.fail();
                })
            },
            refresh(){
                this.getAllCustomers();
                this.field = 'name';
                this.query= '';
                this.$snotify.success('Page reload successfully', 'Success');
            },
            createCustomer(){
                this.editMode = false;
                this.form.reset();
                this.form.clear();
                $('#createCustomer').modal('show');
            },
            async addCustomer(){
                this.$Progress.start();
                this.form.busy = true;
                await this.form.post('/api/customers')
                .then(res => {
                    this.getAllCustomers();
                    $('#createCustomer').modal('hide');
                    if(this.form.successful){
                        this.$Progress.finish();
                        this.$snotify.success('Customer store successfully', 'Success');
                    }else{
                        this.$Progress.fail();
                        this.$snotify.error('Customer not store', 'Error');
                    }
                })
                .catch( e => {
                    this.$Progress.fail();
                    console.log(e)
                })
            },
            editCustomer(customer){
                this.editMode = true;
                this.form.reset();
                this.form.clear();
                this.form.fill(customer);
                $('#createCustomer').modal('show');
            },
            showCustomer(customer){
                this.editMode = true;
                this.form.reset();
                this.form.clear();
                this.form.fill(customer);
                $('#showCustomer').modal('show');
            },
            async updateCustomer(){
                this.$Progress.start();
                this.form.busy = true;
                await this.form.put('/api/customers/'+this.form.id)
                .then(res => {
                    this.getAllCustomers();
                    $('#createCustomer').modal('hide');
                    if(this.form.successful){
                        this.$Progress.finish();
                        this.$snotify.success('Customer Update successfully', 'Success');
                    }else{
                        this.$Progress.fail();
                        this.$snotify.error('Customer not Updated', 'Error');
                    }
                })
                .catch( e => {
                    this.$Progress.fail();
                    console.log(e)
                })
            },
            destroy(customer) {
            this.$snotify.clear();
            this.$snotify.confirm(
                "You will not be able to recover this data!",
                "Are you sure?",
                {
                showProgressBar: false,
                closeOnClick: false,
                pauseOnHover: true,
                buttons: [
                    {
                    text: "Yes",
                    action: toast => {
                        this.$snotify.remove(toast.id);
                        this.$Progress.start();
                        axios.delete("/api/customers/"+customer.id)
                        .then(response => {
                            this.getAllCustomers();
                            this.$Progress.finish();
                            this.$snotify.success(
                            "Customer Successfully Deleted",
                            "Success"
                            );
                        })
                        .catch(e => {
                            this.$Progress.fail();
                            console.log(e);
                        });
                    },
                    bold: true
                    },
                    {
                    text: "No",
                    action: toast => {
                        this.$snotify.remove(toast.id);
                    },
                    bold: true
                    }
                ]
                }
            );
            }

        }
    }
</script>

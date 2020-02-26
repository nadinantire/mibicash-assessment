<template>
  <div class=" py-10" style="width:80%;">
    <div class="text-red" v-if="error">{{ error }}</div>
    <table class="table-auto">
    <thead>
    <tr>
      <th class="px-4 py-2">Name</th>
      <th class="px-4 py-2">Phone</th>
      <th class="px-4 py-2">Date of birth</th>
      <th class="px-4 py-2">Nationality</th>
      <th class="px-4 py-2">Action</th>
    </tr>
  </thead>
  <tbody v-for="customer in customers" :key="customer.id" :customer="customer">
    <tr>
      <td class="border px-4 py-2">{{ customer.name }}</td>
      <td class="border px-4 py-2">{{ customer.phone }}</td>
      <td class="border px-4 py-2">{{ customer.dob }}</td>
      <td class="border px-4 py-2">{{ customer.nationality }}</td>
      <td><button class="bg-tranparent text-sm hover:bg-blue hover:text-white text-blue border border-blue no-underline font-bold py-2 px-4 mr-2 rounded"
          @click.prevent="editCustomer(customer)">Edit</button>

          <button class="bg-transprent text-sm hover:bg-red text-red hover:text-white no-underline font-bold py-2 px-4 rounded border border-red"
         @click.prevent="removeCustomer(customer)">Delete</button></td>
    </tr>
    <div v-if="customer == editedCustomer" style="width:100%">
            <form action="" @submit.prevent="updateCustomer(customer)">
              <div class="mb-6 p-4 bg-white rounded border border-grey-light mt-4">
                <input class="input form-control" v-model="customer.name" /><br>
                <input class="input form-control" v-model="customer.phone" /><br>
                <input class="input form-control" v-model="customer.dob" /><br>
                <input class="input form-control" v-model="customer.nationality" /><br>
                <input type="submit" value="Update" class="btn-info text-sm hover:bg-blue hover:text-white text-blue border border-blue no-underline font-bold py-2 px-4 rounded cursor-pointer">
              </div>
            </form>
          </div>
  </tbody>
    </table>
      <form class="w-full max-w-sm" method="GET" action="http://localhost:3000/api/v1/customerpdf">
        <div class="flex items-center border-b border-b-2 border-teal-500 py-2">
           <input type="submit"  class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded" name="search_button" value="Print pdf">
        </div>
      </form>
        </div>
</template>

<script>
import axios from 'axios'
export default {
  name: 'Customers',
  data () {
    return {
      customers: [],
      newCustomer: [],
      error: '',
      editedCustomer: '',
      upc: '',
      productName: '',
      productCopmleted: '',
      productUser: '',
      productId: ''
    }
  },
  created () {
    if (!localStorage.signedIn) {
      this.$router.replace('/')
    } else {
      this.$http.secured.get('/api/v1/customers')
        .then(response => { this.customers = response.data })
        .catch(error => this.setError(error, 'Something went wrong'))
    }
  },
  methods: {
    setError (error, text) {
      this.error = (error.response && error.response.data && error.response.data.error) || text
    },
    addCustomer () {
      const value = this.newCustomer
      if (!value) {
        return
      }
      this.$http.secured.post('/api/v1/customers/', { customer: { name: this.newCustomer.name, phone: this.newCustomer.phone, nationality: this.newCustomer.nationality, dob: this.newCustomer.dob } })
        .then(response => {
          this.customers.push(response.data)
          this.newCustomer = ''
        })
        .catch(error => this.setError(error, 'Cannot create customer'))
    },
    removeCustomer (customer) {
      this.$http.secured.delete(`/api/v1/customers/${customer.id}`)
        .then(response => {
          this.customers.splice(this.customers.indexOf(customer), 1)
        })
        .catch(error => this.setError(error, 'Cannot delete customer'))
    },
    editCustomer (customer) {
      this.editedCustomer = customer
    },
    updateCustomer (customer) {
      this.editedCustomer = ''
      this.$http.secured.patch(`/api/v1/customers/${customer.id}`, { customer: { name: customer.name } })
        .catch(error => this.setError(error, 'Cannot update customer'))
    },
    lookupUpc: function () {
      var thisApp = this
      thisApp.productName = 'Please wait...'
      axios.get('https://jsonplaceholder.typicode.com/todos/' + thisApp.upc)
        .then(function (response) {
          thisApp.productId = 'id:' + response.data.id
          thisApp.productUser = 'userId:' + response.data.userId
          thisApp.productName = 'title: ' + response.data.title
          thisApp.productCopmleted = 'completed:' + response.data.completed
        })
    }

  },
  watch: {
    upc: function () {
      this.productName = ''
      if (this.upc.length >= 1) {
        this.lookupUpc()
      }
    }
  }
}
</script>

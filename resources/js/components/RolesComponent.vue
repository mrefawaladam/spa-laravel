<template>
   <v-data-table item-key="id" class="elevation-1" 
   :loading="loading"
    loading-text="Loading... Please wait"
   :headers="headers"
    :items="roles"
   >

        <template v-slot:top>
      <v-toolbar flat color="white">
        <v-toolbar-title>My CRUD</v-toolbar-title>
        <v-divider
          class="mx-4"
          inset
          vertical
        ></v-divider>
        <v-spacer></v-spacer>
        <v-dialog v-model="dialog" max-width="500px">
          <template v-slot:activator="{ on }">
            <v-btn color="primary" dark class="mb-2" v-on="on">New Item</v-btn>
          </template>
          <v-card>
            <v-card-title>
              <span class="headline">{{ formTitle }}</span>
            </v-card-title>

            <v-card-text>
              <v-container>
                <v-row>
                  <v-col cols="12" sm="12" >
                    <v-text-field v-model="editedItem.name" label="Dessert name"></v-text-field>
                  </v-col>
                 
                </v-row>
              </v-container>
            </v-card-text>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="close">Cancel</v-btn>
              <v-btn color="blue darken-1" text @click="save">Save</v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-toolbar>
    </template>
    <template v-slot:item.actions="{ item }">
      <v-icon
        small
        class="mr-2"
        @click="editItem(item)"
      >
        mdi-pencil
      </v-icon>
      <v-icon
        small
        @click="deleteItem(item)"
      >
        mdi-delete
      </v-icon>
    </template>
    <template v-slot:no-data>
      <v-btn color="primary" @click="initialize">Reset</v-btn>
    </template>
     <v-snackbar
      v-model="snackbar"
     >
        Record Deleted Succesfully
      <v-btn
        color="pink"
        text
        @click="snackbar = false"
      >
        Close
      </v-btn>
    </v-snackbar>

   </v-data-table>
</template>

<script>
  export default {
    data: () => ({
      dialog: false,
      loading: false,
      snackbar: false,
      headers: [
        {
          text: '#',
          align: 'start',
          sortable: false,
          value: 'id',
        },
        { text: 'Name', value: 'name' },
        { text: 'Created At', value: 'created_at' },
        { text: 'Update At', value: 'updated_at' },
        { text: 'Actions', value: 'actions', sortable: false },
      ],
      roles: [],
      editedIndex: -1,
      editedItem: {
        name: '',
        created_at: '',
        updated_at: ''
      },
      defaultItem: {
        name: '',
        created_at: '',
        updated_at: ''
     
      },
    }),

    computed: {
      formTitle () {
        return this.editedIndex === -1 ? 'New Item' : 'Edit Item'
      },
    },

    watch: {
      dialog (val) {
        val || this.close()
      },
    },

    created () {
      this.initialize()
    },

    methods: {
      initialize () {
  
      // Add a request interceptor
      axios.interceptors.request.use((config) => {
          this.loading = true;
          return config;
        }, (error) => {
          this.loading = false;
          return Promise.reject(error);
        });

      // Add a response interceptor
      axios.interceptors.response.use((response) => {
        
          this.loading = false;
          return response;
        }, (error) => {
          this.loading = false;
          return Promise.reject(error);
        });
      // proses pengambilana api
      axios.get('/api/roles',{})
      .then(res => this.roles = res.data.roles)
      
      .catch(err => {
        if(err.response.status == 401){
          localStorage.removeItem('token');
          this.$router.push('/login');
        }
      }) 
      },

      editItem (item) {
        this.editedIndex = this.roles.indexOf(item)
        this.editedItem = Object.assign({}, item)
        this.dialog = true
      },

      deleteItem (item) {
        const index = this.roles.indexOf(item)
        let decide =  confirm('Are you sure you want to delete this item?') 
       if(decide){
          axios.delete('/api/roles/' + item.id)
          .then(res => {
            this.snackbar = true})
            this.roles.splice(index, 1) 
       }
      },

      close () {
        this.dialog = false
        setTimeout(() => {
          this.editedItem = Object.assign({}, this.defaultItem)
          this.editedIndex = -1
        }, 300)
      },

      save () {
        if (this.editedIndex > -1) {
          axios.put('/api/roles/' + this.editedItem.id, {'name':this.editedItem.name})
          .then(res =>   this.initialize())
         
         
        } else {
         axios.post('/api/roles',{'name' : this.editedItem.name})
         .then(res => this.roles.push(res.data.role) )
        }
        this.close()
      },
    },
  }
</script>
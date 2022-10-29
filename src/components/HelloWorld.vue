<template>
  <v-container>
    <v-data-table
            :headers="headers"
            :items="items"
            sort-by="calories"
            class="elevation-1"
    >
      <template v-slot:top>
        <v-toolbar
                flat
        >
          <v-toolbar-title>Test Task</v-toolbar-title>
          <v-divider
                  class="mx-4"
                  inset
                  vertical
          ></v-divider>
          <v-spacer></v-spacer>
          <v-dialog
                  v-model="dialog"
                  max-width="500px"
          >
            <template v-slot:activator="{ on, attrs }">
              <v-btn
                      color="primary"
                      dark
                      class="mb-2"
                      v-bind="attrs"
                      v-on="on"
              >
                New Item
              </v-btn>
            </template>
            <v-card>
              <v-card-title>
                <span class="text-h5">{{ formTitle }}</span>
              </v-card-title>

              <v-card-text>
                <v-container>
                  <v-row>
                    <v-col
                            cols="12"
                            sm="6"
                            md="4"
                    >
                      <v-text-field
                              v-model="editedItem.title"
                              label="Title"
                      ></v-text-field>
                    </v-col>
                    <v-col
                            cols="12"
                            sm="6"
                            md="4"
                    >
                      <v-text-field
                              v-model="editedItem.body"
                              label="Body"
                      ></v-text-field>
                    </v-col>
                    <v-col
                            cols="12"
                            sm="6"
                            md="4"
                    >
                      <v-text-field
                              v-model="editedItem.userId"
                              label="userId"
                      ></v-text-field>
                    </v-col>
                  </v-row>
                </v-container>
              </v-card-text>

              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn
                        color="blue darken-1"
                        text
                        @click="close"
                >
                  Cancel
                </v-btn>
                <v-btn
                        color="blue darken-1"
                        text
                        @click="save"
                >
                  Save
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
          <v-dialog v-model="dialogDelete" max-width="500px">
            <v-card>
              <v-card-title class="text-h5">Are you sure you want to delete this item?</v-card-title>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" text @click="closeDelete">Cancel</v-btn>
                <v-btn color="blue darken-1" text @click="deleteItemConfirm">OK</v-btn>
                <v-spacer></v-spacer>
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
        <v-btn
                color="primary"
                @click="initialize"
        >
          Reset
        </v-btn>
      </template>
    </v-data-table>
  </v-container>
</template>

<script>
  import axios from 'axios'
  export default {
    name: 'HelloWorld',

    data: () => ({
      headers: [
        { text: 'ID',
          value: 'id',
          align: 'start',
          sortable: false,},
        { text: 'Title', value: 'title' },
        { text: 'Body', value: 'body' },
        { text: 'User ID', value: 'userId' },
        { text: 'Actions', value: 'actions', sortable: false },
      ],
      dialog: false,
      dialogDelete: false,
      items: [],
      editedIndex: -1,
      editedItem: {
        title: '',
        body: '',
        userId: 0,
      },
      defaultItem: {
        title: '',
        body: '',
        userId: 0,
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
      dialogDelete (val) {
        val || this.closeDelete()
      },
    },
    created () {
      this.initialize()
    },
    methods: {
      initialize () {
        axios.get('https://jsonplaceholder.typicode.com/posts').then((res) => {
          console.log(res)
          this.items = res.data
        })
      },

      editItem (item) {
        this.editedIndex = this.items.indexOf(item)
        this.editedItem = Object.assign({}, item)
        this.dialog = true
      },

      deleteItem (item) {
        this.editedIndex = this.items.indexOf(item)
        this.editedItem = Object.assign({}, item)
        this.dialogDelete = true
      },

      deleteItemConfirm () {
        this.items.splice(this.editedIndex, 1)
        this.closeDelete()
      },

      close () {
        this.dialog = false
        this.$nextTick(() => {
          this.editedItem = Object.assign({}, this.defaultItem)
          this.editedIndex = -1
        })
      },

       closeDelete () {
        this.dialogDelete = false
        this.$nextTick( async () => {
          await axios.delete('https://jsonplaceholder.typicode.com/posts/'+this.editedItem.id)
          this.editedItem = Object.assign({}, this.defaultItem)
          this.editedIndex = -1
        })
      },

      async save () {
        console.log(this.editedItem)
        await axios.post('https://jsonplaceholder.typicode.com/posts', {
          title: this.editedItem.title,
          body: this.editedItem.body,
          userId: this.editedItem.userId,
        }, {
          headers: {
            'Content-type': 'application/json; charset=UTF-8',
          },
        })
        if (this.editedIndex > -1) {
          Object.assign(this.items[this.editedIndex], this.editedItem)
        } else {
          this.items.push(this.editedItem)
        }
        this.close()
      }
    }
  }
</script>

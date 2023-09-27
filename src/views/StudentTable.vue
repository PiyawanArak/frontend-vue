<template>
    <div>
    <v-data-table
        :headers="headers"
        :items="studentItem"
        class="elevation-1"
      >
        <template v-slot:top>
          <v-toolbar
            flat
          >
            <v-toolbar-title>Table Student</v-toolbar-title>
            <v-divider
              class="mx-4"
              inset
              vertical
            ></v-divider>
            <v-spacer></v-spacer>
            <v-btn
                  color="primary"
                  dark
                  class="mb-2"
                  @click="openDialog('add', defaultItem)"
                >
                  เพิ่มข้อมูล
                </v-btn>
    
          </v-toolbar>
        </template>
        
        <template v-slot:[`item.actions`] = "{ item }">
          <v-btn icon  @click="openDialog('edit', item)" color="blue">
          <v-icon >
            mdi-pencil
          </v-icon>
          </v-btn>
          <v-btn icon @click="deleteItem(item)" color="red" class="ml-2">
          <v-icon>
            mdi-delete
          </v-icon>
          </v-btn>
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
      <v-dialog
              v-model="dialogCreate"
              max-width="500px"
            >
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
                        md="6"
                      >
                        <v-text-field
                          v-model="studentId"
                          label="รหัสนักศึกษา"
                        ></v-text-field>
                      </v-col>
                      <v-col
                        cols="12"
                        sm="6"
                        md="6"
                      >
                        <v-text-field
                          v-model="name"
                          label="ชื่อ - นามสกุล"
                        ></v-text-field>
                      </v-col>
                      <v-col
                        cols="12"
                        sm="6"
                        md="12"
                      >
                     
                        <v-text-field
                          v-model="email"
                          label="E-mail"
                        ></v-text-field>
                      </v-col>
                      <v-col
                        cols="12"
                        sm="6"
                        md="6"
                      >
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
                    ยกเลิก
                  </v-btn>
                  <v-btn
                    color="blue darken-1"
                    text
                    @click="save(formTitle)"
                  >
                    บันทึกข้อมูล
                  </v-btn>
                </v-card-actions>
              </v-card>
            </v-dialog>
            <v-dialog v-model="dialogDelete" max-width="500px">
              <v-card>
                <v-card-title class="text-h5">ตุณต้องการลบข้อมูลนี้ในตารางใช่ หรือ ไม่?</v-card-title>
                <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn color="blue darken-1" text @click="closeDelete">ยกเลิก</v-btn>
                  <v-btn color="blue darken-1" text @click="deleteItemConfirm">ตกลง</v-btn>
                  <v-spacer></v-spacer>
                </v-card-actions>
              </v-card>
            </v-dialog>
    </div>
    </template>
    
    <script>
    export default {
      data: () => ({
        studentId: '',
        name: '',
        email: '',
        dialogCreate: false,
        dialogDelete: false,
        headers: [
          {
            text: 'id',
            align: 'start',
            sortable: false,
            value: 'id'
          },
          { text: 'รหัสนักศึกษา', value: 'studentId' },
          { text: 'ชื่อ - นามสกุล', value: 'name' },
          { text: 'E-mail', value: 'email' },
        
          { text: 'Actions', value: 'actions', sortable: false }
        ],
        studentItem: [],
        editedIndex: -1,
        editedItem: {
            studentId: '',
            name: 0,
            email: 0,
        },
        defaultItem: {
            studentId: '',
            name: 0,
            email: 0,
        },
        formTitle: '',
        idStudent: '',
        idforDelete: ''
      }),
    
      watch: {
        dialog (val) {
          val || this.close()
        },
        dialogDelete (val) {
          val || this.closeDelete()
        }
      },
    
      created () {
        this.initialize()
      },
    
      methods: {
        async initialize () {
          this.studentItem = []
          try {
            var data = await this.axios.get('http://localhost:9000/students')
            console.log('data student ====>', data)
            this.studentItem = data.data
          } catch (error) {
    
          }
        },
        openDialog (Action, item) {
          this.formTitle = ''
          if (Action === 'add') {
            this.dialogCreate = true
            this.formTitle = 'เพิ่มข้อมูล'
            this.editedItem = item
          } else {
            this.formTitle = 'แก้ไขข้อมูล'
            this.dialogCreate = true
            this.studentId = item.studentId
            this.name = item.name
            this.email = item.email
            this.idStudent = item.id
          }
        },
    
        editItem (item) {
          console.log('item select', item)
          this.editedIndex = this.studentItem.indexOf(item)
          this.editedItem = Object.assign({}, item)
          this.dialog = true
        },
    
        deleteItem (item) {
          this.idforDelete = item.id
          this.dialogDelete = true
        },
    
        async deleteItemConfirm () {
          try {
            var response = await this.axios.delete('http://localhost:9000/student/' + this.idforDelete)
            this.initialize()
          } catch (error) {
            console.log(error.message)
          }
          this.closeDelete()
        },
    
        close () {
          this.dialogCreate = false
          this.editedItem = []
          this.editedIndex = -1
          this.defaultItem = {
            studentId: '',
            name: 0,
            email: 0,
           
          }
        },
    
        closeDelete () {
          this.dialogDelete = false
          this.$nextTick(() => {
            this.editedItem = Object.assign({}, this.defaultItem)
            this.editedIndex = -1
          })
        },
    
        async save (action) {
          var data = {
            studentId: this.studentId,
            name: this.name,
            email: this.email,
          
          }
          if (action === 'เพิ่มข้อมูล') {
            try {
              var dataResponse = await this.axios.post('http://localhost:9000/students', data)
              this.$router.go(0)
              console.log('dataResponse ====>', dataResponse)
              this.close()
              this.initialize()
            } catch (error) {
              console.log(error.message)
            }
          } else {
            try {
              var dataResponse = await this.axios.put('http://localhost:9000/student/' + this.idStudent, data)
              console.log('dataResponse ====>', dataResponse)
              this.close()
              this.initialize()
            } catch (error) {
              console.log(error.message)
            }
          }
        }
      }
    }
    </script>
    
    <style>
    
    </style>
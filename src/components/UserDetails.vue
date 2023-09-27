<template>
  <div class="ma-8">
    <v-data-table
      :headers="headers"
      :items="users"
      :items-per-page="10"
      class="elevation-1"
      :options.sync="options"
      :server-items-length="total"
      :loading="loading"
    >
    <template v-slot:item.icons="{ item }">
      <!-- <v-chip
        :color="getColor(item.icons)"
        dark
      >
        {{ item.calories }}
      </v-chip> -->
      <v-avatar color="primary" size="30">
      <span class="white--text text-h8">{{getAvatarText(item.firstName,item.lastName)}}</span>
    </v-avatar>
    </template>
  
  
  </v-data-table>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      total : 0,
      loading : false,
      users: [
        {
          id: 1,
          firstName: "Sayan",
          lastName: "Saha",
        },
        {
          id: 2,
          firstName: "Priyanshu",
          lastName: "Dey",
        },
        {
          id: 3,
          firstName: "Saikat",
          lastName: "Manna",
        },
      ],
      headers: [
        {
          text: "Roll",
          align: "start",
          sortable: false,
          value: "id",
        },
        {
          text : "Avatar",
          value : "icons",
          sortable: false,
        },
        { text: "First Name", value: "firstName" },
        { text: "Last Name", value: "lastName" },
        { text: "Email", value: "email" },
        { text: "Age", value: "age" },
        { text: "Gender", value: "gender" },
      ],
      options: {},
    };
  },
  watch: {
    options: {
      handler(n) {
        console.log(n);
        this.loadData( {
          offset : (n.page -1) * n.itemsPerPage,
          limit : n.itemsPerPage,
          sortBy : n.sortBy,
          sortDesc : n.sortDesc
        })

      },
    },
    deep: true,
    immediate: true,
  },
  methods: {
    getDetails(obj) {
      //console.log(obj)
      this.loading = true;
      return new Promise((resolve, reject) => {
        axios
          .get("https://dummyjson.com/users")
          .then((response) => {
            // console.log(response.data);
            // this.users = response.data['users'];
            // console.log(this.users);
            
            let data = {
              user:  response.data['users'].slice(obj.offset,  obj.limit + obj.offset),
              meta: {
                limit: obj.limit,
                offset: obj.offset,
                total: response.data['users'].length,
              },
            };

            resolve(data);
            console.log(data);
          })
          .catch((error) => {
            console.log(error);
            reject(error);
          })
          .finally(() => {
            this.loading = false;
          })
      });

      //   Promise.all([fetch("https://dummyjson.com/users")])
      //     .then(async ([res]) => {
      //       const data = await res.json();
      //       return [data];
      //     })
      //     .then((result) => {
      //       this.useData = result[0];
      //       console.log(result[0]["users"]);
      //       let userArray = [];
      //       result[0]["users"].map((items) => {
      //         console.log(items);
      //         let userObj = {
      //           id: items.id,
      //           firstName: items.firstName,
      //           lastName: items.lastName,
      //           email:items.email,
      //           age: items.age,
      //           gender: items.gender
      //         };
      //         userArray = [...userArray, userObj];
      //       });
      //       console.log(userArray);
      //       this.users = userArray;
      //     })
      //     .catch((err) => {
      //       console.log(err);
      //     });
    },
    async loadData(obj){
      let userData = await this.getDetails( obj);
        console.log(userData);
        this.users = userData.user;
        this.total = userData.meta.total;
        
        // if (obj.sortBy && obj.sortBy.length === 1 && obj.sortDesc && obj.sortDesc.length === 1) {
          if (obj.sortBy.length === 1 && obj.sortDesc.length === 1) {
          this.users = this.users.sort((a, b) => {
              // const sortA = a[obj.sortBy]
              // const sortB = b[obj.sortBy]
              const sortA = a[obj.sortBy[0]]
              const sortB = b[obj.sortBy[0]]

              if (obj.sortDesc[0]) {
                if (sortA < sortB) return 1
                if (sortA > sortB) return -1
                return 0
              } else {
                if (sortA < sortB) return -1
                if (sortA > sortB) return 1
                return 0
              }
            })
          }
    },
    getAvatarText(fname,lname)
    {
      let avatarText = fname.split('')[0] + lname.split('')[0];
      return avatarText
    },
  },
mounted () {
  // this.loadData();
},
};
</script>
<style></style>

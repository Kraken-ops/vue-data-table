<template>
  <div class="ma-8">
    <v-container fluid>
      <v-card class="mx-auto elevation-2" max-width="2000" outlined>
        <v-card-title class="d-flex justify-space-between">
          <div class="" style="color: rgba(0, 0, 0, 0.54)">
            <v-icon class="ml-4">mdi-account</v-icon>
            User Details
          </div>
          <div class="d-flex">
            <v-text-field
              v-model.trim="search"
              label="Search"
              append-icon="mdi-magnify"
              @keyup.13="searchData"
            ></v-text-field>
            <v-icon class="ma-4 pa-2" @click="downloadCsv">mdi-download</v-icon>
          </div>
        </v-card-title>

        <v-data-table
          ref="dataTable"
          :headers="headers"
          :items="filteredUsers"
          :items-per-page="10"
          class="elevation-1 ma-2"
          :options.sync="options"
          :server-items-length="total"
          :loading="loading"
        >
          <template #[`item.icons`]="{ item }">
            <v-tooltip bottom>
              <template v-slot:activator="{ on, attrs }">
                <v-avatar color="primary" size="30" v-bind="attrs" v-on="on">
                  <span class="white--text text-h8">{{
                    getAvatarText(item.firstName, item.lastName)
                  }}</span>
                </v-avatar>
              </template>
              <span>{{ item.firstName }} {{ item.lastName }}</span>
            </v-tooltip>
          </template>
        </v-data-table>
        
      </v-card>
    </v-container>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      total: 0,
      loading: false,
      search: "",
      filteredUsers: [],
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
          text: "Avatar",
          value: "icons",
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
        this.loadData({
          offset: (n.page - 1) * n.itemsPerPage,
          limit: n.itemsPerPage,
          sortBy: n.sortBy,
          sortDesc: n.sortDesc,
        });
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
              user: response.data["users"].slice(
                obj.offset,
                obj.limit + obj.offset
              ),
              meta: {
                limit: obj.limit,
                offset: obj.offset,
                total: response.data["users"].length,
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
          });
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
    async loadData(obj) {
      let userData = await this.getDetails(obj);
      console.log(userData);
      this.users = userData.user;
      this.filteredUsers = userData.user;
      this.total = userData.meta.total;

      // if (obj.sortBy && obj.sortBy.length === 1 && obj.sortDesc && obj.sortDesc.length === 1) {
      if (obj.sortBy.length === 1 && obj.sortDesc.length === 1) {
        this.users = this.users.sort((a, b) => {
          // const sortA = a[obj.sortBy]
          // const sortB = b[obj.sortBy]
          const sortA = a[obj.sortBy[0]];
          const sortB = b[obj.sortBy[0]];

          if (obj.sortDesc[0]) {
            if (sortA < sortB) return 1;
            if (sortA > sortB) return -1;
            return 0;
          } else {
            if (sortA < sortB) return -1;
            if (sortA > sortB) return 1;
            return 0;
          }
        });
      }
    },
    getAvatarText(fname, lname) {
      let avatarText = fname.split("")[0] + lname.split("")[0];
      return avatarText;
    },
    searchData() {
      // Filter the data based on searchText
      // this.search = this.search.toLowerCase();
      if (this.search) {
        this.filteredUsers = this.users.filter(
          (user) =>
            user.firstName.toLowerCase().includes(this.search.toLowerCase()) ||
            user.lastName.toLowerCase().includes(this.search.toLowerCase())
        );
      } else {
        this.filteredUsers = this.users;
      }
    },
    exportCsv() {
      console.log(this.$refs.dataTable);
      const dataTable = this.$refs.dataTable;
      if (!dataTable) {
        console.error("Data table ref not found");
        return;
      }
      if (typeof dataTable.exportCsv !== "function") {
        console.error("Export CSV method not found");
        return;
      }
      dataTable.exportCsv();
    },
    convertToCsv(data) {
      // Empty array for storing the values
      let csvRows = [];

      // Headers is basically a keys of an
      // object which is id, name, and
      // profession
      const headers = Object.keys(data);

      // As for making csv format, headers
      // must be separated by comma and
      // pushing it into array
      csvRows.push(headers.join(","));

      // Pushing Object values into array
      // with comma separation
      const values = Object.values(data).join(",");
      csvRows.push(values);

      // Returning the array joining with new line
      return csvRows.join("\n");
    },
    downloadCsv() {
      // let data = ["1", "name", "jyhqef"];
      let data = this.filteredUsers;
      console.log(data, "datas download");
      // const csv = this.convertToCsv(data);
      const csv = [
        ["id,firstName,lastName,email,age,gender"],
        ...data.map((item) => [
          item.id,
          item.firstName,
          item.lastName,
          item.email,
          item.age,
          item.gender,
        ]),
      ]
        .map((e) => e.join(","))
        .join("\n");
      console.log(csv);
      const blob = new Blob([csv], { type: "text/xlsx" });
      const url = URL.createObjectURL(blob);
      const link = document.createElement("a");
      link.setAttribute("href", url);
      link.setAttribute("download", "data.xlsx");
      link.style.visibility = "hidden";
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
    },
  },
  //   computed: {
  //   filteredUsers() {
  //     if (this.search) {
  //       return this.users.filter(user => user.firstName.toLowerCase().includes(this.search) || user.lastName.toLowerCase().includes(this.search));
  //     } else {
  //       return this.users;
  //     }
  //   }
  // },
  mounted() {
    // this.loadData();
  },
};
</script>
<style></style>

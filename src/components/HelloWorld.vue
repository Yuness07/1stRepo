<template>
  <v-container>
    <v-row class="text-center">
      <v-toolbar height="100px">
        <v-col cols="6" class="mb-4">
          <h1 id="custom-h1" class="display-2 font-weight-bold mt-7">
            Welcome to the Vuetify 3 Beta
          </h1>
        </v-col>
        <v-col cols="6">
          <v-img
            :src="require('../assets/logo.svg')"
            class="my-3"
            contain
            height="100"
          />
        </v-col>
      </v-toolbar>
    </v-row>
    <v-row>
      <v-col cols="6">
        <v-form method="post" ref="form">
          <v-select
            :disabled="!checked"
            v-model="selectedId"
            :items="idList"
            placeholder="Select an ID"
            @update:modelValue="updateTextFields"
          ></v-select>
          <v-select
            label="UserID"
            v-model="item.userId"
            :items="useridList"
          ></v-select>
          <v-text-field label="Tilte" v-model="item.title"></v-text-field>
          <v-text-field label="Body" v-model="item.body"></v-text-field>
        </v-form>
      </v-col>

      <v-col cols="6">
        <v-snackbar
          v-model="snackbar"
          :text="snackbarText"
          :timeout="timeout"
          :color="snackbarColor"
          :rounded="rounded"
        >
          <template v-slot:actions>
            <v-btn
              color="purple-lighten-5"
              variant="text"
              @click="snackbar = false"
            >
              Close
            </v-btn>
          </template></v-snackbar
        >
        <v-checkbox
          v-model="checked"
          @click="Uncheck"
          label="Please Select an ID to Update or to Delete an Item"
        ></v-checkbox>
        <v-btn
          :disabled="checked"
          size="large"
          block
          color="primary"
          class="mt-5"
          @click="ToAdd"
          >STORE</v-btn
        >
        <v-btn
          :disabled="!checked"
          size="large"
          block
          color="secondary"
          class="mt-5"
          @click="ToUpdate"
          >UPDATE</v-btn
        >
        <v-btn
          :disabled="!checked"
          size="large"
          block
          color="error"
          class="mt-5"
          @click="ToDelete"
          >DELETE</v-btn
        >
      </v-col>

      <v-col cols="12">
        <v-data-table
          hover
          :headers="headers"
          :items="items"
          class="elevation-1"
        ></v-data-table>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
//import axios from "axios";
export default {
  name: "HelloWorld",
  data() {
    return {
      selectedId: null,
      checked: false,
      snackbar: false,
      snackbarText: "",
      snackbarColor: "",
      rounded: "pill",
      timeout: 2000,
      headers: [
        { title: "UserId", key: "userId" },
        { title: "ID", key: "id" },
        { title: "Title", key: "title" },
        { title: "Body", key: "body" },
      ],
      items: [],
      idList: [],
      useridList: [],
      item: {
        id: "",
        userId: "",
        title: "",
        body: "",
      },
    };
  },
  created() {
    this.ToList();
  },
  mounted() {
    this.fetchIds();
    this.fetchUserIds();
    this.fetchPosts();
  },
  methods: {
    // Listing All Resources
    ToList() {
      fetch("https://jsonplaceholder.typicode.com/posts/")
        .then((response) => response.json())
        .then((json) => this.items = json)
        .catch((error) => this.IamCallbackFun(error));
    },
    //Extract the IDs
    fetchIds() {
      // Simulating fetching data from a fake API
      fetch("https://jsonplaceholder.typicode.com/posts/")
        .then((response) => response.json())
        .then((json) => {
          // Assuming the response is an array of posts and you want to extract the IDs
          this.idList = json.map((post) => post.id);
        })
        .catch(this.IamCallbackFun);
    },
    // Extract the UserIDs
    fetchUserIds() {
      // Simulating fetching data from a fake API
      fetch("https://jsonplaceholder.typicode.com/posts/")
        .then((response) => response.json())
        .then((json) => {
          // Assuming the response is an array of posts and you want to extract the UserIDs
          const uniqueUserIds = [...new Set(json.map((post) => post.userId))];
          this.useridList = uniqueUserIds;
        })
        .catch(this.IamCallbackFun);
    },
    fetchPosts() {
      fetch("https://jsonplaceholder.typicode.com/posts/")
        .then((response) => response.json())
        .then((json) => {
          // Assuming the response is an array of posts
          this.posts = json;
          this.idList = json.map((post) => post.id);
        })
        .catch(this.IamCallbackFun);
    },
    watch: {
      selectedId(newId) {
        this.updateTextFields(newId);
      },
    },
    updateTextFields() {
      const selectedPost = this.posts.find(
        (post) => post.id === this.selectedId
      );
      if (selectedPost) {
        this.item.userId = selectedPost.userId;
        this.item.title = selectedPost.title;
        this.item.body = selectedPost.body;
        console.log(selectedPost);
      } else {
        this.item.userId = "";
        this.item.title = "";
        this.item.body = "";
      }
    },
    // Adding a Resource
    ToAdd() {
      if (
        this.item.userId == "" ||
        this.item.title == "" ||
        this.item.body == ""
      ) {
        this.snackbarText = "Please Fill all Required!";
        this.snackbarColor = "red-darken-2";
        this.snackbar = true;
      } else {
        fetch("https://jsonplaceholder.typicode.com/posts", {
          method: "POST",
          body: JSON.stringify({
            userId: this.item.userId,
            title: this.item.title,
            body: this.item.body,
          }),
          headers: {
            "Content-type": "application/json; charset=UTF-8",
          },
        })
          .then((response) => response.json())
          .then((json) => {
            this.snackbarText = "Item Successfully Added!";
            this.snackbarColor = "blue-darken-2"; // You can customize the color
            this.snackbar = true;
            console.log(json);
          })
          .catch(this.ScndCallbackFun)
          .finally(() => {
            this.$refs.form.reset();
          });
      }
    },
    // Updating a resource
    ToUpdate() {
      if (this.item.userId == "") {
        this.snackbarText = "Please Select an ID!";
        this.snackbarColor = "red-darken-2"; // You can customize the color
        this.snackbar = true;
      } else if (this.item.title == "" || this.item.body == "") {
        this.snackbarText = "تاعمر كلشي مالك";
        this.snackbarColor = "violet-darken-2"; // You can customize the color
        this.snackbar = true;
      } else {
        fetch(`https://jsonplaceholder.typicode.com/posts/${this.item.id}`, {
          method: "PUT",
          body: JSON.stringify({
            title: this.item.title,
            body: this.item.body,
            userId: this.item.userId,
          }),
          headers: {
            "Content-type": "application/json; charset=UTF-8",
          },
        })
          .then((response) => response.json())
          .then((json) => {
            this.snackbarText = "Item Successfully Updated!";
            this.snackbarColor = "success"; // You can customize the color
            this.snackbar = true;
            console.log(json);
          })
          .catch(this.ScndCallbackFun)
          .finally(() => {
            this.item = {
              id: "",
              userId: "",
              title: "",
              body: "",
            };
          });
      }
    },
    // Deleting a resource
    ToDelete() {
      if (
        this.item.userId == "" ||
        this.item.title == "" ||
        this.item.body == ""
      ) {
        this.snackbarText = "Please Select The ID Again!";
        this.snackbarColor = "red-darken-2"; // You can customize the color
        this.snackbar = true;
      } else {
        fetch(`https://jsonplaceholder.typicode.com/posts/${this.item.id}`, {
          method: "DELETE",
        })
          .then(() => {
            this.snackbarText = "Item Successfully Deleted!";
            this.snackbarColor = "green-darken-2";
            this.snackbar = true;
          })
          .catch(this.ScndCallbackFun)
          .finally(() => {
            this.item = {
              id: "",
              userId: "",
              title: "",
              body: "",
            };
          });
      }
    },
    Uncheck() {
      this.item = {
        id: "",
        userId: "",
        title: "",
        body: "",
      };
    },
    IamCallbackFun(error) {
      this.snackbarText = error;
      this.snackbarColor = "red-darken-3";
      this.snackbar = true;
    },
    ScndCallbackFun() {
      this.snackbarText = "Error: Something Goes Wrong!";
      this.snackbarColor = "orange-darken-3";
      this.snackbar = true;
    },
  },
};
</script>

<style scoped>
#custom-h1 {
  color: rgb(3, 73, 97);
  animation: flicker 0.5s ease-in-out infinite alternate;
}
@keyframes flicker {
  0% {
    opacity: 0.5;
    text-shadow: 2px 2px 10px rgb(203, 232, 216);
  }
  100% {
    opacity: 1;
    text-shadow: 2px 2px 20px rgb(73, 73, 162);
  }
}
.v-data-table {
  color: gray;
  background: rgb(174, 220, 238);
  background: radial-gradient(
    circle,
    rgba(174, 220, 238, 1) 0%,
    rgb(148, 168, 233) 100%
  );
}
</style>

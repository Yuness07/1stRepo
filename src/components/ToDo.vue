<template>
  <v-container>
    <v-row>
      <v-col cols="12">
        <v-data-table
          hover
          :headers="headers"
          :items="Dataitems"
          class="elevation-1 bg-cyan-lighten-4"
          item-key="id"
          v-if="!clickedRow"
        >
          <template v-slot:item="{ item }">
            <tr @click="onRowClick(item)">
              <td>{{ item.id }}</td>
              <td>{{ item.userId }}</td>
              <td>{{ item.title }}</td>
              <td>{{ item.body }}</td>
              <td>
                <v-icon size="small" class="me-2" @click.stop="editItem(item)">
                  mdi-pencil
                </v-icon>
              </td>
              <td>
                <v-icon size="small" @click.stop="deleteItem(item)">
                  mdi-delete
                </v-icon>
              </td>
            </tr>
          </template>
          <template v-slot:top>
            <v-toolbar class="bg-cyan-lighten-3" flat>
              <v-toolbar-title>Fetched DATA</v-toolbar-title>
              <v-divider class="mx-4" inset vertical></v-divider>
              <v-spacer></v-spacer>
              <v-dialog v-model="dialog" max-width="500px">
                <template v-slot:activator="{ props }">
                  <v-btn
                    @click="Enabler"
                    color="primary"
                    dark
                    class="mb-2"
                    v-bind="props"
                  >
                    Store New Item
                  </v-btn>
                </template>
                <v-card>
                  <v-card-title class="bg-cyan-lighten-3">
                    <span class="text-h5">{{ formTitle }}</span>
                  </v-card-title>

                  <v-card-text class="bg-cyan-lighten-3">
                    <v-container>
                      <v-row>
                        <v-col cols="6">
                          <v-text-field
                            class="bg-cyan-lighten-4"
                            v-model="editedItem.id"
                            :disabled="disabled"
                            label="ID"
                          ></v-text-field>
                        </v-col>
                        <v-col cols="6">
                          <v-text-field
                            class="bg-cyan-lighten-4"
                            v-model="editedItem.userId"
                            :disabled="disabled"
                            label="User ID"
                          ></v-text-field>
                        </v-col>
                      </v-row>
                      <v-row>
                        <v-col cols="12">
                          <v-text-field
                            clearable
                            clear-icon="mdi-close-circle"
                            auto-grow
                            class="bg-cyan-lighten-4"
                            v-model="editedItem.title"
                            label="Title"
                          ></v-text-field>
                        </v-col>
                      </v-row>
                      <v-row>
                        <v-col cols="12">
                          <v-textarea
                            clearable
                            clear-icon="mdi-close-circle"
                            auto-grow
                            class="bg-cyan-lighten-4"
                            v-model="editedItem.body"
                            label="Body"
                          ></v-textarea>
                        </v-col>
                      </v-row>
                    </v-container>
                  </v-card-text>

                  <v-card-actions class="bg-cyan-lighten-3">
                    <v-spacer></v-spacer>
                    <v-btn color="white" variant="text" @click="close">
                      Cancel
                    </v-btn>
                    <v-btn color="white" variant="text" @click="save">
                      Save
                    </v-btn>
                  </v-card-actions>
                </v-card>
              </v-dialog>
              <v-dialog v-model="dialogDelete" max-width="500px">
                <v-card class="bg-cyan-lighten-3">
                  <v-card-title class="text-h5 text-red"
                    >Are you sure you want to delete this item?</v-card-title
                  >
                  <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="white" variant="text" @click="closeDelete"
                      >Cancel</v-btn
                    >
                    <v-btn
                      color="white"
                      variant="text"
                      @click="deleteItemConfirm"
                      >OK</v-btn
                    >
                    <v-spacer></v-spacer>
                  </v-card-actions>
                </v-card>
              </v-dialog>
            </v-toolbar>
          </template>
        </v-data-table>

        <clicked-row-details
          v-if="clickedRow"
          :clickedRow="clickedRow"
          @goBack="goBack"
        />
      </v-col>
      <v-snackbar
        v-model="snackbar"
        :text="snackbarText"
        :timeout="timeout"
        :color="snackbarColor"
        :rounded="rounded"
      ></v-snackbar>
    </v-row>
  </v-container>
</template>

<script>
import ClickedRowDetails from "@/components/ClickedRowDetails.vue";

export default {
  components: {
    ClickedRowDetails,
  },
  data() {
    return {
      headers: [
        { title: "ID", key: "id" },
        { title: "UserID", key: "userId" },
        { title: "Title", key: "title" },
        { title: "Body", key: "body" },
        { title: "Actions" },
        { title: "" },
      ],
      Dataitems: [],
      clickedRow: null,
      snackbar: false,
      snackbarText: "",
      snackbarColor: "",
      rounded: "pill",
      timeout: 4000,
      disabled: true,
      dialog: false,
      dialogDelete: false,
      editedIndex: -1,
      editedItem: {
        id: "",
        userId: "",
        title: "",
        body: "",
      },
      defaultItem: {
        id: "",
        userId: "",
        title: "",
        body: "",
      },
    };
  },
  created() {
    this.getData();
  },
  methods: {
    getData() {
      // Check if data is in local storage
      const cachedData = localStorage.getItem("cachedData");

      if (cachedData) {
        // Use cached data if present
        this.Dataitems = JSON.parse(cachedData);
        this.snackbarText = "Using Cached Data";
        this.snackbarColor = "black-darken-3";
        this.snackbar = true;
        return;
      }

      // Fetch data from the API
      const url = "https://jsonplaceholder.typicode.com/posts";
      fetch(url)
        .then((response) => response.json())
        .then((json) => {
          this.Dataitems = json;

          // Store data in local storage
          localStorage.setItem("cachedData", JSON.stringify(json));
          this.snackbarText = "Data fetched from API and Cached to the LocalStorage:";
          this.snackbarColor = "black-darken-3";
          this.snackbar = true;
        })
        .catch((error) => console.log(error));
    },

    onRowClick(item) {
      this.clickedRow = item;
    },

    goBack() {
      this.clickedRow = null;
    },
    editItem(item) {
      this.editedIndex = this.Dataitems.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialog = true;
    },

    deleteItem(item) {
      this.editedIndex = this.Dataitems.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialogDelete = true;
    },

    deleteItemConfirm() {
      this.Dataitems.splice(this.editedIndex, 1);
      this.closeDelete();
    },

    close() {
      this.dialog = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
      this.disabled = true;
    },

    closeDelete() {
      this.dialogDelete = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
    },

    save() {
      if (this.editedIndex > -1) {
        Object.assign(this.Dataitems[this.editedIndex], this.editedItem);
      } else {
        this.Dataitems.push(this.editedItem);
      }
      this.disabled = true;
      this.close();
    },

    Enabler() {
      this.disabled = false;
    },
  },
};
</script>

<style></style>

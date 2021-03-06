<template>
  <div class="category-maintenance">
    <v-app-bar app color="#F52900" dense height="3px" flat></v-app-bar>
    <v-data-table
      :headers="headers"
      :items="category"
      class="elevation-1"
      hide-default-footer
      :items-per-page="-1"
    >
      <template v-slot:top>
        <v-toolbar flat color="white">
          <v-toolbar-title>My CATEGORY</v-toolbar-title>
          <v-divider class="mx-4" inset vertical></v-divider>
          <v-spacer></v-spacer>
          <v-dialog v-model="dialog" max-width="500px">
            <template v-slot:activator="{ on, attrs }">
              <v-btn color="#F52900" fab dark top right v-bind="attrs" v-on="on" class="mt-10">
                <v-icon dark>mdi-plus</v-icon>
              </v-btn>
            </template>
            <v-card>
              <v-card-title>
                <span class="headline">{{ formTitle }}</span>
              </v-card-title>
              <v-card-text>
                <v-container id="container-dialog-category-maintenance">
                  <v-row>
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field v-model="editedItem.name" label="Category"></v-text-field>
                    </v-col>
                  </v-row>
                </v-container>
              </v-card-text>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="#F52900" text @click="close">{{$t("Cancel")}}</v-btn>
                <v-btn color="blue darken-1" text @click="save()">{{$t("Save")}}</v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-toolbar>
      </template>
      <template v-slot:item.actions="{ item }">
        <v-icon small class="mr-2" @click="editItem(item)">mdi-pencil</v-icon>
        <v-icon small @click="deleteItem(item)">mdi-delete</v-icon>
      </template>
    </v-data-table>
    <v-footer app padless height="44px">
      <CommonFooter />
    </v-footer>
  </div>
</template>

<script>
import firebase from "firebase";
import "firebase/firestore";
import utilsMixin from "../utils";
import CommonFooter from "./CommonFooter.vue";

export default {
  mixins: [utilsMixin],
  components: {
    CommonFooter
  },
  data: () => ({
    dialog: false,
    headers: [
      { text: "Category", value: "name" },
      { text: "Actions", value: "actions", sortable: false }
    ],
    category: [],
    editedIndex: -1,
    editedItem: {
      id: "",
      name: ""
    },
    defaultItem: {
      id: "",
      name: ""
    }
  }),

  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "New Item" : "Edit Item";
    },
    company: function() {
      return this.$route.query.company;
    }
  },

  watch: {
    dialog(val) {
      val || this.close();
    }
  },

  created() {
    this.getCategoryDb();
  },

  methods: {
    getCategoryDb: function() {
      console.log("getCategoryDb start");
      this.db = firebase.firestore();
      const _this = this;

      this.db
        .collection(this.company)
        .doc("category")
        .onSnapshot(function(doc) {
          _this.category = [];
          let data = doc.data();
          Object.keys(data).forEach(key => _this.category.push(data[key]));
        });
      console.log("getCategoryDb end");
    },
    editItem(item) {
      this.editedIndex = this.category.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialog = true;
    },

    deleteItem(item) {
      let deleteField = {};
      (deleteField[item.id] = firebase.firestore.FieldValue.delete()),
        confirm("Are you sure you want to delete this item?") &&
          this.db
            .collection(this.company)
            .doc("category")
            .update(deleteField);
    },

    close() {
      this.dialog = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
    },
    save() {
      this.db = firebase.firestore();
      let newCategoryRef = this.db.collection(this.company).doc("category");

      let updateId = "";
      if (this.editedIndex > -1) {
        // Add a new document with a generated id.
        updateId = this.editedItem.id;
      } else {
        updateId = this.createRandomId();
      }

      let updateData = {};
      updateData[updateId] = {
        id: updateId,
        name: this.editedItem.name
      };

      newCategoryRef.set(updateData, { merge: true });
      this.close();
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
#container-dialog-category-maintenance .v-input input {
  max-height: 100px !important;
}
</style>

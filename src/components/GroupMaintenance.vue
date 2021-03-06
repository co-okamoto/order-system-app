<template>
  <div class="groupMaintenance">
    <v-app-bar app color="#F52900" dense height="3px" flat></v-app-bar>
    <v-data-table
      :headers="headers"
      :items="group"
      class="elevation-1"
      group-by="category.name"
      hide-default-footer
      :items-per-page="-1"
    >
      <template v-slot:top>
        <v-toolbar flat color="white">
          <v-toolbar-title>My GROUP</v-toolbar-title>
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
                <v-container>
                  <v-row>
                    <v-col cols="12" sm="6" md="4">
                      <v-select
                        v-model="editedItem.category"
                        :items="category"
                        item-text="name"
                        item-value="id"
                        label="Category"
                        height="44px"
                        return-object
                      ></v-select>
                    </v-col>
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field
                        id="text-input-dialog-group"
                        v-model="editedItem.groupName"
                        label="Group"
                        height="44px"
                      ></v-text-field>
                    </v-col>
                  </v-row>
                </v-container>
              </v-card-text>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="#00BFBF" text @click="close">{{$t("Cancel")}}</v-btn>
                <v-btn color="#F52900" text @click="save">{{$t("Save")}}</v-btn>
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
  components: {
    CommonFooter
  },
  mixins: [utilsMixin],
  data: () => ({
    dialog: false,
    headers: [
      { text: "Category", value: "category.name" },
      { text: "Group", value: "groupName" },
      { text: "Actions", value: "actions", sortable: false }
    ],
    category: [],
    group: [],
    editedIndex: -1,
    editedItem: {
      category: "",
      groupName: ""
    },
    defaultItem: {
      category: "",
      groupName: ""
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
    this.getGroupDb();
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
    getGroupDb: function() {
      console.log("getCategoryDb start");
      this.db = firebase.firestore();
      const _this = this;

      this.db
        .collection(this.company)
        .doc("group")
        .onSnapshot(function(doc) {
          _this.group = [];
          let data = doc.data();
          Object.keys(data).forEach(key => {
            let value = data[key];
            let pushData = {
              category: {
                id: value.categoryId,
                name: value.categoryName
              },
              groupId: value.groupId,
              groupName: value.groupName
            };
            _this.group.push(pushData);
          });
        });
      console.log("getCategoryDb end");
    },
    editItem(item) {
      console.log(item);
      this.editedIndex = this.group.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialog = true;
    },
    deleteItem(item) {
      let deleteField = {};
      (deleteField[item.groupId] = firebase.firestore.FieldValue.delete()),
        confirm("Are you sure you want to delete this item?") &&
          this.db
            .collection(this.company)
            .doc("group")
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
      let newGroupRef = this.db.collection(this.company).doc("group");
      let updateId = "";
      if (this.editedIndex > -1) {
        // Add a new document with a generated id.
        updateId = this.editedItem.groupId;
      } else {
        updateId = this.createRandomId();
      }

      // later...

      let updateData = {};
      updateData[updateId] = {
        categoryId: this.editedItem.category.id,
        categoryName: this.editedItem.category.name,
        groupId: updateId,
        groupName: this.editedItem.groupName
      };

      console.log(updateData);
      newGroupRef.set(updateData, { merge: true });
      this.close();
    }
  }
};
</script>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
.v-dialog .v-select__selections {
  line-height: 30px;
}

#text-input-dialog-group {
  margin: 0;
  padding: 0;
}
</style>

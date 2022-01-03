<template>
  <div>
    <v-text-field label="Фильтр пока не работает" solo>
      <v-icon slot="append" color="red" @click="handleInsert()">
        mdi-plus
      </v-icon>
    </v-text-field>
    <v-dialog
      v-model="showCreateDialog"
      transition="dialog-bottom-transition"
      max-width="600"
    >
      <template v-slot:default="dialog">
        <v-card>
          <v-toolbar color="primary" dark>Добавление данных</v-toolbar>
          <v-alert
            :value="alert"
            dense
            text
            type="success"
            transition="scale-transition"
          >
            Данные успешно добавлены
          </v-alert>
          <v-card-text>
            <v-select
              :items="listOfParentCategories"
              label="категория родитель"
              solo
              item-text="name"
              item-value="id"
              v-model="newItem.parentId"
            >
              <template v-slot:item="{ item }">
                <span>{{ item.name }}</span>
              </template>
            </v-select>
          </v-card-text>
          <v-text-field
            v-model="newItem.name"
            label="Наименование"
            required
          ></v-text-field>
          <v-text-field
            v-model="newItem.price"
            label="Цена"
            required
          ></v-text-field>
          <v-card-actions class="justify-end">
            <v-btn text @click="createNewItem()" color="primary"
              >Добавить</v-btn
            >
            <v-btn text @click="dialog.value = false">Закрыть</v-btn>
          </v-card-actions>
        </v-card>
      </template>
    </v-dialog>
    <v-treeview dense :items="items">
      <template v-slot:label="{ item }">
        <!-- , leaf, selected, indeterminate, active, open  -->
        <span
          >{{ item.name }}{{ " |  Цена:"
          }}{{ item.price }}</span
        >
      </template>
    </v-treeview>
  </div>
</template>

<script>
import axios from "axios";
export default {
  data: () => ({
    items: [],
    listOfParentCategories: [],
    showCreateDialog: false,
    newItem: {
      id: 0,
      name: "",
      price: 0,
      parentId: 0,
      children: null,
    },
    alert: false,
  }),
  methods: {
    resetNewItem() {
      this.newItem = {
        id: 0,
        name: "",
        price: 0,
        parentId: 0,
        children: null,
      };
    },
    async getCatalogList() {
      await axios.get(`catalog`).then((resp) => {
        this.items = this.listToTree(resp.data);
      });
    },
    async getCatalogList_2() {
      await axios.get(`catalog`).then((resp) => {
        this.listOfParentCategories = resp.data;
      });
    },
    async createNewItem() {
      await axios.post(`catalog`, this.newItem).then((resp) => {
        this.alert = true;
        setTimeout(() => {
          this.alert = false;
        }, 2000);
        this.resetNewItem();
      });
    },
    handleInsert() {
      this.showCreateDialog = true;
    },
    listToTree(list) {
      var map = {},
        node,
        roots = [],
        i;

      for (i = 0; i < list.length; i += 1) {
        map[list[i].id] = i;
        list[i].children = [];
      }

      for (i = 0; i < list.length; i += 1) {
        node = list[i];
        if (node.parentId !== 0) {
          list[map[node.parentId]].children.push(node);
        } else {
          roots.push(node);
        }
      }

      return roots;
    },
  },
  async created() {
    await this.getCatalogList();
  },
  watch: {
    showCreateDialog() {
      if (this.showCreateDialog) this.getCatalogList_2();
      else this.getCatalogList();
    },
  },
};
</script>
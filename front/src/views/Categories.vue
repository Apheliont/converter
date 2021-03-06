<template>
  <v-flex align-self-start>
    <v-card fill-height>
      <v-card-title class="my-title">
        Редактор категорий:
        <v-spacer></v-spacer>
        <v-dialog v-model="dialog" max-width="500px" persistent>
          <v-btn slot="activator" color="#ce4b6d" dark class="mb-2">Добавить категорию</v-btn>
          <v-card>
            <v-card-title>
              <span class="headline">{{ formTitle }}</span>
            </v-card-title>
            <v-card-text>
              <v-form v-model="valid">
                <v-container grid-list-md>
                  <v-layout column>
                    <v-flex xs12 sm6 md4>
                      <v-tooltip right v-model="tooltipName" color="white">
                        <template v-slot:activator="{ on }">
                          <span>
                            <v-text-field
                              append-outer-icon="help_outline"
                              @click:append-outer="tooltipName = !tooltipName"
                              v-model="editedItem.name"
                              @input="hasChanged = true"
                              :rules="commonRules"
                              label="Название"
                              clearable
                            ></v-text-field>
                          </span>
                        </template>
                        <span>Служит в качестве лейбла при сопостовление с выходным путем</span>
                      </v-tooltip>
                    </v-flex>
                    <v-flex xs12 sm6 md4>
                      <v-tooltip right v-model="tooltipPath" color="white">
                        <template v-slot:activator="{ on }">
                          <span>
                            <v-text-field
                              append-outer-icon="help_outline"
                              @click:append-outer="tooltipPath = !tooltipPath"
                              v-model="editedItem.path"
                              @input="hasChanged = true"
                              :rules="commonRules"
                              label="Путь для сохранения конечного результата"
                              clearable
                            ></v-text-field>
                          </span>
                        </template>
                        <span>То, куда будет помещен итоговый результат кодирования. ВАЖНО! Путь должен быть доступен со стороны обработчиков! Сервер по данному пути обращаться не будет</span>
                      </v-tooltip>
                    </v-flex>
                    <v-flex>
                      <v-layout row nowrap>
                        <v-flex grow>
                          <v-tooltip left v-model="tooltipPreset" color="white">
                            <template v-slot:activator="{ on }">
                              <span>
                                <v-text-field
                                  append-outer-icon="help_outline"
                                  @click:append-outer="tooltipPreset = !tooltipPreset"
                                  v-model="editedItem.preset"
                                  @input="hasChanged = true"
                                  :rules="commonRules"
                                  label="Пресет"
                                  clearable
                                ></v-text-field>
                              </span>
                            </template>
                            <span>Пресеты находятся в директории presets модуля обработчика. Расширение файла указывать необязательно</span>
                          </v-tooltip>
                        </v-flex>
                        <v-flex shrink class="my-priority">
                          <v-tooltip right color="white">
                            <template v-slot:activator="{ on }">
                              <span v-on="on">
                                <v-select
                                  v-model="editedItem.priority"
                                  @input="hasChanged = true"
                                  :items="Array.from({length: 10}, (i, v) => v + 1)"
                                  dense
                                  type="number"
                                  label="Приоритет"
                                ></v-select>
                              </span>
                            </template>
                            <span>Чем меньше значение, тем выше приоритет заданий для этой категории</span>
                          </v-tooltip>
                        </v-flex>
                      </v-layout>
                    </v-flex>
                  </v-layout>
                </v-container>
              </v-form>
            </v-card-text>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="error" @click="close">Отмена</v-btn>
              <v-btn color="success" :disabled="!valid || !hasChanged" @click="save">Сохранить</v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-card-title>
        <v-data-table
          ref="categories-data-table"
          no-data-text
          :headers="headers"
          :items="categories"
          hide-actions
          class="my-data-table"
          item-key="props.item.id"
          disable-initial-sort
        >
          <template slot="items" slot-scope="props">
            <td class="text-xs-left">{{ props.item.id }}</td>
            <td class="text-xs-left">{{ props.item.name }}</td>
            <td class="text-xs-left">{{ props.item.path }}</td>
            <td class="text-xs-left">{{ props.item.preset }}</td>
            <td class="text-xs-left">{{ props.item.priority }}</td>
            <td class="text-xs-left">
              <v-icon class="mr-2" @click="editItem(props.item)">edit</v-icon>
              <v-icon @click="deleteItem(props.item)">delete</v-icon>
            </td>
          </template>
        </v-data-table>
    </v-card>
  </v-flex>
</template>

<script>
import { mapActions } from "vuex";
import { mapGetters } from "vuex";
export default {
  data: () => ({
    tooltipName: false,
    tooltipPath: false,
    tooltipPreset: false,
    dialog: false,
    hasChanged: false,
    valid: false,
    headers: [
      {
        text: "ID",
        align: "left",
        value: "id",
        width: 50
      },
      {
        text: "Название",
        align: "left",
        value: "name"
      },
      { text: "Путь", align: "left", value: "path" },
      {
        text: "Пресет",
        align: "left",
        value: "preset",
        width: 150
      },
      {
        text: "Приоритет",
        align: "left",
        value: "priority",
        width: 150
      },
      {
        text: "Действия",
        value: "name",
        align: "left",
        sortable: false,
        width: 150
      }
    ],
    editedIndex: -1,
    editedItem: {
      name: "",
      path: "",
      preset: "",
      priority: 5
    },
    defaultItem: {
      name: "",
      path: "",
      preset: "",
      priority: 5
    },
    commonRules: [v => (v && v.length > 0) || "Поле не может быть пустым"]
  }),

  computed: {
    ...mapGetters({
      categories: "categories/categories"
    }),
    formTitle() {
      return this.editedIndex === -1 ? "Новая категория" : "Редактирование";
    }
  },

  watch: {
    dialog(val) {
      val || this.close();
    }
  },

  methods: {
    ...mapActions("categories", [
      "addCategory",
      "updateCategory",
      "deleteCategory"
    ]),
    editItem(item) {
      this.editedIndex = this.categories.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialog = true;
    },

    deleteItem(item) {
      confirm("Вы уверены что хотите удалить категорию?") &&
        this.deleteCategory(item.id);
    },

    close() {
      this.dialog = false;
      this.hasChanged = false;
      setTimeout(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      }, 300);
    },

    save() {
      this.hasChanged = false;
      if (this.editedIndex > -1) {
        this.updateCategory(this.editedItem);
      } else {
        this.addCategory(this.editedItem);
      }
      this.close();
    },
    resize() {
      const viewportHeight = window.innerHeight;
      const dtHeight = viewportHeight - 250 < 100 ? 100 : viewportHeight - 250;
      this.$refs["categories-data-table"].$el.style.height = `${dtHeight}px`;
    }
  },
  mounted() {
    this.resize();
    window.addEventListener("resize", this.resize);
  },
  beforeDestroy() {
    window.removeEventListener("resize", this.resize);
  }
};
</script>

<style scoped>
.my-data-table {
  margin-top: -4px;
  overflow-y: scroll;
}

.my-title {
  background-color: #ececec;
  padding: 10px 20px;
}

.my-priority {
  width: 30%;
}
</style>



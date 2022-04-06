<template>
  <div class="table-list-container">
    <div class="flex flex--justify-space-between">
      <div>
        <h2 class="mainContent__heading mainContent__heading--inProduct">
          {{ $root.translate.menu["design-modules"].items.layouts }}
        </h2>
      </div>
    </div>
    <div class="singleForm singleForm--inModulesTpl" v-if="layouts.length">
      <div class="moduleTemplates">
        <menu class="menu menu--inModulesTpl">
          <ul class="menu__list">
            <li
              class="menu__item"
              v-for="(layout, index) in layouts"
              :key="index"
            >
              <div
                class="menu__heading"
                :class="{ 'menu__heading--active': openLayout === index }"
                @click="openLayout = index"
              >
                <span>
                  {{ layout.name }}
                </span>
              </div>
            </li>
          </ul>
        </menu>
        <main
          class="mainContent mainContent--noHeight"
          v-bar="{ useScrollbarPseudo: true, preventParentScroll: false }"
        >
          <div>
            <div class="grid grid--3 grid--gap_25">
              <div
                class="layout"
                v-for="position in positions"
                :key="openLayout.index + '-' + position"
              >
                <div class="layout__title">
                  {{ $root.translate.columns[position] }}
                </div>
                <div class="layout__content">
                  <draggable
                    class="layout__modules"
                    :list="layouts[openLayout].modules"
                    v-bind="dragOptions"
                    @start="drag = true"
                    @end="drag = false"
                    handle=".icon-move"
                  >
                    <div
                      class="layoutModule"
                      v-for="(module, moduleIndex) in layouts[openLayout]
                        .modules"
                      :key="
                        layouts[openLayout].name +
                        ' ' +
                        position +
                        ' ' +
                        moduleIndex
                      "
                      v-show="module.position === position"
                    >
                      <div class="flex flex--align-center">
                        <icon
                          class="icon icon-delete mr--8"
                          icon="cancel"
                          @click.native="
                            layouts[openLayout].modules.splice(moduleIndex, 1)
                          "
                        ></icon>
                        {{ module.name }}
                      </div>
                      <icon class="icon icon-move" icon="move"></icon>
                    </div>
                  </draggable>

                  <v-select
                    :components="{ Deselect, OpenIndicator }"
                    :multiple="false"
                    :clearable="true"
                    :searchable="true"
                    :options="modules"
                    @input="
                      (module) => addModuleToSelectedList(module, position)
                    "
                    class="input vs--modules vs--multiply mb--45"
                    label="id"
                  >
                    <template #search="{ attributes, events }">
                      <div class="flex vs__searchContainer">
                        <div class="vs__searchIcon">+</div>

                        <input
                          class="vs__search"
                          :placeholder="$root.translateWords('Add module')"
                          v-bind="attributes"
                          v-on="events"
                        />
                      </div>
                    </template>

                    <template v-slot:option="option">
                      {{ option.name }}
                    </template>

                    <div slot="no-options">
                      {{ $root.translateWords("Data not found") }}
                    </div>
                  </v-select>
                </div>
              </div>
            </div>
          </div>
        </main>
      </div>
    </div>
    <widget-actions
      :store="isChangedLayout ? 'update' : false"
    ></widget-actions>
  </div>
</template>

<script>
import draggable from "vuedraggable";

export default {
  name: "LayoutsPage",
  components: {
    draggable,
  },
  computed: {
    dragOptions() {
      return {
        animation: 200,
        group: "description",
        disabled: false,
        ghostClass: "ghost",
      };
    },
    isChangedLayout() {
      let current = this.$root.copy(this.layouts[this.openLayout]);

      delete current.refreshing;

      let saved = this.$root.copy(this.savedOriginal[this.openLayout]);

      delete saved.refreshing;

      return JSON.stringify(current) !== JSON.stringify(saved);
    },
  },
  data() {
    return {
      refreshing: false,
      openLayout: 0,
      layouts: [],
      savedOriginal: [],
      modules: [],
      Deselect: {
        render: (createElement) =>
          createElement("icon", {
            class: "icon",
            props: {
              icon: "error",
            },
          }),
      },
      OpenIndicator: {
        render: (createElement) => createElement("span", ""),
      },
      drag: false,
      positions: [
        "left_column",
        "right_column",
        "content_top",
        "content_bottom",
        "bellow_header",
      ],
    };
  },
  created() {
    axios.get("/admin/layouts/").then((httpResponse) => {
      this.$set(this, "layouts", httpResponse.data);

      this.savedOriginal = this.$root.copy(this.layouts);
    });

    axios
      .get("/admin/modules/", {
        params: {
          autocomplete: true,
        },
      })
      .then((httpResponse) => {
        for (let module_index in httpResponse.data) {
          this.modules.push({
            id: httpResponse.data[module_index].id,
            name: httpResponse.data[module_index].name,
            value: httpResponse.data[module_index].id,
            text: httpResponse.data[module_index].name,
          });
        }
      });
  },
  methods: {
    addModuleToSelectedList(module, position) {
      this.layouts[this.openLayout].modules.push({
        id: module.id,
        value: module.id,
        name: module.name,
        position: position,
      });
    },
    update() {
      this.refreshing = true;

      axios
        .put(
          "/admin/layouts/" + this.layouts[this.openLayout].id,
          this.layouts[this.openLayout]
        )
        .then((httpResponse) => {
          this.$root.notify(httpResponse.data);

          this.savedOriginal = this.$root.copy(this.layouts);
        })
        .catch((error) => {
          if (error.response) this.$root.notify(error.response.data);
        })
        .finally(() => {
          this.refreshing = false;
        });
    },
  },
};
</script>

<style scoped></style>

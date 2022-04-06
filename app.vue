<template>
  <div>
    <div
      style="position: relative"
      v-if="Object.keys($root.translate).length && isAuthorized"
    >
      <nav class="topNav">
        <router-link :to="{ name: 'dashboard' }" class="topNav__item">
          <img
            src="/core-static/images/logo-admin.png"
            alt=""
            class="img-responsive"
          />
        </router-link>
        <div class="topNav__item">
          <a href="/" class="topNav__link" target="_blank">
            <icon icon="shopping-cart" class="icon topNav__icon"></icon>
            <span>{{ $root.translate.menu.my_shop }}</span>
          </a>
        </div>
        <div class="topNav__item">
          <router-link class="topNav__link" :to="{ name: 'settings' }">
            <icon icon="settings" class="icon topNav__icon"></icon>
            <span>{{ $root.translate.menu.settings.heading }}</span>
          </router-link>
        </div>
        <div
          class="topNav__item"
          v-click-outside="
            () =>
              $root.popups.support
                ? $root.changePopupShowStatus('support', false)
                : false
          "
        >
          <a
            href="javascript:void(0)"
            class="topNav__link"
            @click="$root.changePopupShowStatus('support')"
          >
            <icon icon="support" class="icon topNav__icon"></icon>
            <span>{{ $root.translate.menu.support }}</span>
          </a>
          <div
            class="topNav__supportInfo supportInfo"
            v-if="$root.popups.support"
          >
            <div class="supportInfo__main">
              <div class="supportInfo__heading">
                {{ $root.translateWords("Do you need help?") }}
              </div>
              <div class="supportInfo__manual">
                <a
                  class="supportInfo__link flex flex--align-center"
                  href="/core-static/files/instruction-wizorCMS.pdf"
                  target="_blank"
                >
                  {{ $root.translateWords("Open short manual") }}
                  <icon icon="foreign" class="icon"></icon>
                </a>
                <p>
                  {{
                    $root.translateWords(
                      "Manual may help you to work with admin panel"
                    )
                  }}
                </p>
              </div>
              <ul class="supportInfo__list supportInfoList">
                <li class="supportInfoList__item flex flex--align-center">
                  <icon icon="email" class="icon supportInfoList__icon"></icon>
                  {{ $root.translateWords("Send us an email") }}
                  <a
                    href="mailto:help.wizor@gmail.com"
                    class="supportInfo__link supportInfo__link--withText"
                  >
                    help.wizor@gmail.com
                  </a>
                </li>
                <!-- <li class="supportInfoList__item flex flex--align-center">
                                    <icon icon="support" class="icon supportInfoList__icon"></icon>
                                    <a href="/" class="supportInfo__link">
                                        {{$root.translateWords('Online chat')}}
                                    </a>
                                </li> -->
                <li class="supportInfoList__item flex flex--align-center">
                  <icon icon="" class="icon supportInfoList__icon"></icon>
                  {{ $root.translateWords("Call us at") }}
                  <a
                    href="tel:+380636315051"
                    class="supportInfo__link supportInfo__link--withText"
                  >
                    +38 (063) 631-50-51
                  </a>
                </li>
              </ul>
            </div>
            <div class="supportInfoSchedule">
              <div class="supportInfoSchedule__heading">
                {{ $root.translateWords("Support work schedule") }}
              </div>
              <p class="supportInfoSchedule__info">
                {{ $root.translateWords("Monday-Friday from 10am to 5pm") }}
                <br />
                {{ $root.translateWords("Saturday, sunday - free days") }}
              </p>
            </div>
          </div>
        </div>
        <div class="topNav__extra">
          <div
            class="topNav__search"
            v-click-outside="
              function () {
                searchResultsMayShow = false;
              }
            "
          >
            <div class="topNav__searchFieldContainer">
              <input
                v-model="searchPhrase"
                type="text"
                :placeholder="
                  $root.translateWords('Search products, categories, etc.')
                "
                class="topNav__searchField"
                @focus="searchResultsMayShow = true"
              />
              <icon icon="search" class="icon"></icon>
            </div>
            <div
              class="topNav__searchResults searchResults"
              v-if="
                searchResultsMayShow &&
                searchPhrase.length > 2 &&
                Object.keys(searchResults).length
              "
            >
              <div
                class="searchResults__group"
                v-for="(searchGroup, index) in searchResults"
                :key="index"
              >
                <div class="searchResults__heading">
                  {{ searchGroup.name }}:
                </div>
                <ul class="searchResults__list" v-if="searchGroup.items.length">
                  <li v-for="item in searchGroup.items" :key="item.id">
                    <router-link
                      :to="{ name: searchGroup.type, params: { id: item.id } }"
                      class="flex flex--align-center searchResultsItem"
                      :class="{
                        'searchResultsItem--product':
                          searchGroup.type === 'product',
                      }"
                    >
                      <img
                        v-if="searchGroup.type === 'product'"
                        class="searchResultsItem__img"
                        :src="item.image"
                        alt=""
                      />
                      <div
                        class="flex flex--column flex--justify-space-between"
                      >
                        <div class="searchResultsItem__name">
                          {{ item.name }}
                        </div>
                        <div
                          class="searchResultsItem__sku"
                          v-if="searchGroup.type === 'product'"
                        >
                          {{ $root.translate.columns.sku }}:
                          {{ item.sku }}
                        </div>
                      </div>
                    </router-link>
                  </li>
                </ul>
              </div>
            </div>
          </div>
          <div
            class="flex flex--1 flex--align-center flex--justify-space-between w100perc"
          >
            <div class="topNav__notification">
              <icon icon="email" class="icon"></icon>
              <span class="topNav__notification_text">10</span>
            </div>
            <a href="/account/logout" class="topNav__logout">
              <icon icon="turn-off" class="icon"></icon>
              <span>{{ $t("columns.logout") }}</span>
            </a>
          </div>
        </div>
      </nav>
      <div class="mainContainer">
        <menu class="menu">
          <perfect-scrollbar>
            <div class="menu__scroll">
              <ul class="menu__list">
                <li
                  class="menu__item"
                  v-for="(menuItem, index) in menu"
                  :key="index"
                >
                  <div
                    class="menu__heading"
                    :class="{ 'menu__heading--active': isActive(index) }"
                    v-if="menuItem.children.length"
                    @click="toggleSubMenu(index)"
                  >
                    <span>
                      <icon
                        :icon="menuItem.icon"
                        class="icon menu__icon"
                      ></icon>
                      {{ menuItem.name }}
                      <span
                        class="menu__notification"
                        v-if="
                          Object(menuItem).hasOwnProperty('notification') &&
                          notifications[menuItem.notification] > 0
                        "
                      >
                        +{{ notifications[menuItem.notification] }}
                      </span>
                    </span>
                    <icon
                      icon="play-button"
                      class="icon menu__icon"
                      :class="{ 'menu__icon--opened': menuItem.openedSubMenu }"
                    ></icon>
                  </div>
                  <router-link
                    v-else
                    class="menu__heading menu__link"
                    :class="{ 'menu__heading--active': isActive(index) }"
                    :to="{ name: menuItem.to }"
                  >
                    <span>
                      <icon
                        :icon="menuItem.icon"
                        class="icon menu__icon"
                      ></icon>
                      {{ menuItem.name }}
                    </span>
                  </router-link>
                  <ul
                    class="menu__list"
                    v-if="menuItem.children.length"
                    v-show="menuItem.openedSubMenu"
                  >
                    <li
                      class="menu__item menu__item--sub"
                      v-for="(menuSubItem, index) in menuItem.children"
                      :key="index"
                    >
                      <router-link
                        class="menu__link"
                        :class="{
                          'menu__link--active': menuSubItem.to === $route.name,
                        }"
                        :to="{ name: menuSubItem.to }"
                      >
                        {{ menuSubItem.name }}
                        <span
                          class="menu__notification"
                          v-if="
                            Object(menuSubItem).hasOwnProperty(
                              'notification'
                            ) && notifications[menuSubItem.notification] > 0
                          "
                        >
                          +{{ notifications[menuSubItem.notification] }}
                        </span>
                      </router-link>
                    </li>
                  </ul>
                </li>
              </ul>
            </div>
          </perfect-scrollbar>
          <div class="menu__tenant tenantInfo mt--20">
            <p class="flex flex--align-center">
              {{ $root.translateWords("Your tariff is:") }}
              <span class="tenantInfo__tariff">{{
                $root.tenantTariffInfo.name
              }}</span>
              <icon icon="icon" class="tenantInfo__icon icon"></icon>
            </p>
            <p
              class="flex flex--align-center"
              v-if="$root.tenantTariffInfo.expiredAt.length"
            >
              {{ $root.translateWords("Expired at:") }}
              <time>{{ $root.tenantTariffInfo.expiredAt }}</time>
            </p>
          </div>
        </menu>

        <perfect-scrollbar
          :min-scrollbar-length="200"
          id="perfect-scroll-offset"
        >
          <main
            class="mainContent"
            :class="{ 'mainContent--product': $route.name === 'product' }"
          >
            <div>
              <keep-alive
                v-if="$route.name === 'product' || $route.name === 'products'"
              >
                <router-view
                  @start-loading="loadingContent = true"
                  @stop-loading="loadingContent = false"
                ></router-view>
              </keep-alive>
              <router-view
                v-else
                @start-loading="loadingContent = true"
                @stop-loading="loadingContent = false"
              ></router-view>
            </div>
            <!-- <div class="preloader" v-if="loadingContent">
                                <icon class="icon preloader__icon" icon="waiting"></icon>
                            </div> -->
          </main>
        </perfect-scrollbar>
      </div>
      <div v-if="$root.waiting_server_action.status" class="global_preloader">
        <div class="preloader">
          <icon class="icon preloader__icon" icon="waiting"></icon>
          <div class="massage">
            {{ $root.waiting_server_action.text }}.
            {{ $t("words.Wait please") }}
          </div>
        </div>
      </div>
    </div>
    <div v-else>
      <authorization @authorized="afterAuthorized"></authorization>
    </div>
    <notifications
      group="success"
      position="bottom right"
      animation-type="velocity"
      :speed="1000"
    />

    <notifications
      group="error"
      position="bottom right"
      animation-type="velocity"
      :speed="1000"
    />
  </div>
</template>

<script>
import VueRouter from "vue-router";
import AttributesComponent from "./AttributesComponent";

Vue.use(VueRouter);

let CategoriesComponent = require("./CategoriesComponent").default;

import { Datetime } from "vue-datetime";

import "vue-datetime/dist/vue-datetime.css";

Vue.component("modal", require("./modalComponent").default);

Vue.use(Datetime);

Vue.component("datetime", Datetime);

import Vuebar from "vuebar";

Vue.use(Vuebar);

let router = new VueRouter({
  mode: "history",
  routes: [
    {
      path: "/admin/",
      component: require("./Dashboard").default,
      name: "dashboard",
    },
    {
      path: "/admin/categories",
      component: CategoriesComponent,
      name: "categories",
    },
    {
      path: "/admin/categories/:id/edit",
      component: require("./CategoryComponent").default,
      name: "category",
    },
    {
      path: "/admin/categories/create",
      component: require("./CategoryComponent").default,
      name: "category-create",
    },
    {
      path: "/admin/groups-attributes",
      component: require("./GroupsAttributesComponent").default,
      name: "groups-attributes",
    },
    {
      path: "/admin/languages",
      component: require("./LanguagesComponent").default,
      name: "languages",
    },
    {
      path: "/admin/attributes",
      component: AttributesComponent,
      name: "attributes",
    },
    {
      path: "/admin/attributes/:id/edit",
      component: require("./AttributeComponent").default,
      name: "attribute",
    },
    {
      path: "/admin/currencies/",
      component: require("./CurrenciesComponent").default,
      name: "currencies",
    },
    {
      path: "/admin/user-groups/",
      component: require("./UserGroupsComponent").default,
      name: "userGroups",
    },
    {
      path: "/admin/stock-statuses/",
      component: require("./StockStatusComponent").default,
      name: "stockStatuses",
    },
    {
      path: "/admin/price-units/",
      component: require("./PriceUnits").default,
      name: "priceUnits",
    },
    {
      path: "/admin/products/",
      component: require("./ProductsComponent").default,
      name: "products",
    },
    {
      path: "/admin/products/:id/edit",
      component: require("./ProductComponent").default,
      name: "product",
    },
    {
      path: "/admin/users/",
      component: require("./UsersComponent").default,
      name: "users",
    },
    {
      path: "/admin/settings/",
      component: require("./SettingsComponent").default,
      name: "settings",
    },
    {
      path: "/admin/locations/",
      component: require("./Locations").default,
      name: "locations",
    },
    {
      path: "/admin/locations/:id/edit",
      component: require("./Location").default,
      name: "location",
    },
    {
      path: "/admin/order-statuses/",
      component: require("./OrderStatuses").default,
      name: "orderStatuses",
    },
    {
      path: "/admin/orders/",
      component: require("./Orders").default,
      name: "orders",
    },
    {
      path: "/admin/orders/create",
      component: require("./CreateOrder").default,
      name: "create-order",
    },
    {
      path: "/admin/orders/:id/",
      component: require("./OrderInfo").default,
      name: "orderInfo",
    },
    {
      path: "/admin/posters/",
      component: require("./Banners").default,
      name: "banners",
    },
    {
      path: "/admin/posters/:id/edit",
      component: require("./BannerSlides").default,
      name: "bannerSlides",
    },
    {
      path: "/admin/informations/",
      component: require("./Informations").default,
      name: "informations",
    },
    {
      path: "/admin/informations/:id/edit",
      component: require("./Information").default,
      name: "information",
    },
    {
      path: "/admin/articles/",
      component: require("./Articles").default,
      name: "articles",
    },
    {
      path: "/admin/articles/:id/edit",
      component: require("./Article").default,
      name: "article",
    },
    {
      path: "/admin/testimonials/",
      component: require("./Testimonials").default,
      name: "testimonials",
    },
    {
      path: "/admin/articles/:id/edit",
      component: require("./Testimonial").default,
      name: "testimonial",
    },
    {
      path: "/admin/marketing-seo/",
      component: require("./Marketing_SEO").default,
      name: "marketing-seo",
    },
    {
      path: "/admin/modules/",
      component: require("./ModulesPage").default,
      name: "modules",
    },
    {
      path: "/admin/layouts/",
      component: require("./LayoutsPage").default,
      name: "layouts",
    },
    {
      path: "/admin/totals/",
      component: require("./TotalsPage").default,
      name: "totals",
    },
    {
      path: "/admin/excel/",
      component: require("./Excel").default,
      name: "excel",
    },
    {
      path: "/admin/xml-import/",
      component: require("./XmlImport").default,
      name: "xml-import",
    },
    {
      path: "/admin/xml-export/",
      component: require("./XmlExport").default,
      name: "xml-export",
    },
    {
      path: "/admin/externals-api/",
      component: require("./External_API").default,
      name: "external-api",
    },
    {
      path: "/admin/externals-api/moy-sklad/edit",
      component: require("./MoySklad").default,
      name: "external-service",
    },
  ],
});
import ClickOutside from "vue-click-outside";

import authorization from "./Authorization.vue";

import { PerfectScrollbar } from "vue2-perfect-scrollbar";

Vue.use(PerfectScrollbar);

export default {
  name: "app",
  router,
  directives: {
    ClickOutside,
  },
  components: {
    authorization,
    PerfectScrollbar,
  },
  data() {
    return {
      menu: [],
      searchPhrase: "",
      searchResults: {},
      searchResultsMayShow: true,
      loadingContent: false,
      notifications: {
        "orders-reviews": 0,
        orders: 0,
      },
    };
  },
  created() {
    let self = this;

    this.menu = [
      {
        name: this.$root.translate.columns.home,
        icon: "shop",
        to: "dashboard",
        openedSubMenu: false,
        children: [],
      },
      {
        name: this.$root.translate.menu.catalog.heading,
        icon: "package",
        to: false,
        openedSubMenu: false,
        children: [
          {
            name: this.$root.translate.menu.catalog.items.categories,
            to: "categories",
          },
          {
            name: this.$root.translate.menu.catalog.items.products,
            to: "products",
          },
          {
            name: this.$root.translate.menu.catalog.items.attributes,
            to: "attributes",
          },
          {
            name: this.$root.translate.menu.catalog.items.information,
            to: "informations",
          },
          {
            name: this.$root.translate.menu.catalog.items.articles,
            to: "articles",
          },
        ],
      },
      {
        name: this.$root.translate.menu.users.heading,
        icon: "people",
        to: false,
        openedSubMenu: false,
        children: [
          {
            name: this.$root.translate.menu.users.items.users,
            to: "users",
          },
          {
            name: this.$root.translate.menu.users.items.groups,
            to: "userGroups",
          },
        ],
      },
      {
        name: this.$root.translate.menu["orders-callback"].heading,
        icon: "shopping-cart",
        to: false,
        openedSubMenu: false,
        notification: "orders-reviews",
        children: [
          {
            name: this.$root.translate.menu["orders-callback"].items.orders,
            to: "orders",
            notification: "orders",
          },
          {
            name: this.$root.translate.menu["orders-callback"].items
              .testimonials,
            to: "testimonials",
          },
          {
            name: this.$root.translate.menu["orders-callback"].items[
              "cart-modules"
            ],
            to: "totals",
          },
        ],
      },
      {
        name: this.$root.translate.menu["design-modules"].heading,
        icon: "paint-brush",
        to: false,
        openedSubMenu: false,
        children: [
          {
            name: this.$root.translate.menu["design-modules"].items.banners,
            to: "banners",
          },
          {
            name: this.$root.translate.menu["design-modules"].items.modules,
            to: "modules",
          },
          {
            name: this.$root.translate.menu["design-modules"].items.templates,
            to: "layouts",
          },
        ],
      },
      {
        name: this.$root.translate.menu.localisation.heading,
        icon: "language",
        to: false,
        openedSubMenu: false,
        children: [
          {
            name: this.$root.translate.menu.localisation.items.currencies,
            to: "currencies",
          },
          {
            name: this.$root.translate.menu.localisation.items.languages,
            to: "languages",
          },
          {
            name: this.$root.translate.menu.localisation.items.locations,
            to: "locations",
          },
          {
            name: this.$root.translate.menu.localisation.items[
              "order-statuses"
            ],
            to: "orderStatuses",
          },
          {
            name: this.$root.translate.menu.localisation.items[
              "product-available-statuses"
            ],
            to: "stockStatuses",
          },
          {
            name: this.$root.translate.menu.localisation.items[
              "product-price-unit"
            ],
            to: "priceUnits",
          },
        ],
      },

      {
        name: this.$root.translate.menu.synchronizations.heading,
        icon: "share",
        to: false,
        openedSubMenu: false,
        children: [
          /*{
                            name: 'EXCEL',
                            to: 'excel',
                        },*/
          {
            name: "Xml import",
            to: "xml-import",
          },
          {
            name: "Xml export",
            to: "xml-export",
          },
          {
            name: "Внешние API",
            to: "external-api",
          },
        ],
      },
      {
        name: this.$root.translate.menu.marketing_seo.heading,
        icon: "loupe",
        to: "marketing-seo",
        openedSubMenu: false,
        children: [],
      },
    ];

    this.debouncedSearchGetItems = _.debounce(this.search, 800);

    if (this.isAuthorized) this.afterAuthorized();
  },
  methods: {
    afterAuthorized() {
      Echo.private(window.tenant_domain_hash + "-order").listen(
        "NewOrderEvent",
        () => {
          self.notifications["orders-reviews"]++;

          self.notifications["orders"]++;
        }
      );

      this.setUnViewedOrderAndReviews();
    },
    toggleSubMenu(index) {
      this.menu[index].openedSubMenu = this.menu[index].children.length
        ? !this.menu[index].openedSubMenu
        : false;
    },
    search() {
      axios
        .get("/admin/search", {
          params: {
            phrase: this.searchPhrase,
          },
        })
        .then((httpResponse) => {
          this.$set(this, "searchResults", httpResponse.data);
        });
    },
    setUnViewedOrderAndReviews() {
      axios.get("/admin/orders/not-viewed").then((httpResponse) => {
        this.notifications["orders"] += httpResponse.data;

        this.notifications["orders-reviews"] = this.notifications["orders"];
      });
    },
  },
  computed: {
    isAuthorized() {
      return this.$root.is_admin;
    },
    isActive() {
      return (index) => {
        return (
          this.menu[index].openedSubMenu ||
          this.menu[index].children.findIndex((child) => {
            return child.to === this.$route.name;
          }) !== -1 ||
          this.menu[index].to === this.$route.name
        );
      };
    },
  },
  watch: {
    searchPhrase: function () {
      if (this.searchPhrase.length > 2) {
        this.debouncedSearchGetItems();
      }
    },
    $route(to, from) {
      this.loadingContent = false;
    },
  },
};
</script>

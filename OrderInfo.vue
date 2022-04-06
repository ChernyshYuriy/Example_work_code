<template>
  <div class="table-list-container" v-if="Object.keys(order).length">
    <div>
      <div class="breadcrumbs">
        <router-link class="breadcrumbs__link" :to="{ name: 'dashboard' }">
          {{ $root.translate.columns.home }}
        </router-link>
        -
        <router-link class="breadcrumbs__link" :to="{ name: 'orders' }">
          {{ $root.translate.menu["orders-callback"].items.orders }}
        </router-link>
      </div>
      <h2
        class="mainContent__heading flex flex--justify-space-between flex--align-center"
      >
        <span class="flex">
          {{ $root.translate.columns.order }} №{{ order.id }}
          <span
            class="breadcrumbs flex flex--align-flex-end ml--16"
            v-if="order.type === 'fast'"
          >
            {{ $t("order.quick_order") }}
          </span>
        </span>

        <span class="order__date">
          {{ $root.translate.columns.created_at }}: {{ order.created_at }}
        </span>
      </h2>
    </div>
    <div class="grid grid--2 grid--gap_25">
      <div class="widget widget--startTop">
        <div class="flex flex--align-center">
          <icon
            class="icon widget__icon widget__icon--people"
            icon="people"
          ></icon>
          <div class="flex flex--column">
            <div class="widget__heading">
              {{ $root.translateWords("About customer") }}
            </div>
          </div>
          <span class="tenantInfo">
            <icon
              @click.native.stop="editModalCustomerInfo"
              icon="pencil-edit-button"
              class="tenantInfo__icon tenantInfo__icon--color"
            ></icon>
          </span>
        </div>
        <div class="orderTable">
          <div class="orderTable__row" v-if="order.FullName.length">
            <span class="orderTable__cell orderTable__name">
              {{ $root.translate.columns.fullname }}:
            </span>
            <span class="orderTable__cell orderTable__value">
              {{ order.FullName }}
            </span>
          </div>
          <div class="orderTable__row" v-if="order.email">
            <span class="orderTable__cell orderTable__name"> Email: </span>
            <span class="orderTable__cell orderTable__value">
              {{ order.email }}
            </span>
          </div>
          <div class="orderTable__row" v-if="order.telephone">
            <span class="orderTable__cell orderTable__name">
              {{ $root.translate.columns.telephone }}:
            </span>
            <span class="orderTable__cell orderTable__value">
              {{ order.telephone }}
            </span>
          </div>
        </div>
      </div>
      <div class="widget widget--startTop">
        <div class="flex flex--align-center">
          <icon class="icon widget__icon" icon="shopping-cart"></icon>
          <div class="flex flex--column">
            <div class="widget__heading">
              {{ $root.translateWords("Shipping and payment") }}
            </div>
          </div>
          <icon
            @click.native.stop="editModalShippingAndPayment"
            icon="pencil-edit-button"
            class="tenantInfo__icon tenantInfo__icon--color"
          ></icon>
        </div>
        <div class="orderTable" v-if="isShoppingHaveValue">
          <div class="orderTable__row" v-if="order.shipping">
            <span class="orderTable__cell orderTable__name">
              {{ $root.translate.columns.shipping_method }}:
            </span>
            <span class="orderTable__cell orderTable__value">
              {{ order.shipping.name }}
            </span>
          </div>
          <div class="orderTable__row" v-if="order.payment">
            <span class="orderTable__cell orderTable__name">
              {{ $root.translate.columns.payment_method }}:
            </span>
            <span class="orderTable__cell orderTable__value">
              {{ order.payment.name }}
            </span>
          </div>
          <template v-if="order.decodeAddressInfo">
            <div
              class="orderTable__row"
              v-for="(addressItem, index) in order.decodeAddressInfo"
              :key="index"
            >
              <span class="orderTable__cell orderTable__name">
                {{ addressItem.name }}:
              </span>
              <span class="orderTable__cell orderTable__value">
                {{ addressItem.value }}
              </span>
            </div>
          </template>
        </div>
        <div v-else class="widget__heading widget__heading--no-data">
          {{ $t("order.no_data") }}
        </div>
      </div>
    </div>
    <div>
      <div class="widget" v-if="order.comment">
        <div class="widget__heading flex flex--align-center">
          <icon class="icon widget__icon" icon="quotation-marks"></icon>
          {{ $t("order.customer_comment") }}
        </div>
        <div class="mt--20 text">
          {{ order.comment }}
        </div>
      </div>
    </div>
    <div>
      <div class="widget">
        <div class="widget__heading">
          {{ $root.translate.menu.catalog.items.products }}
        </div>
        <div class="mt--20 listData listData--noBg w100perc">
          <vue-good-table
            :columns="productsColumns"
            :rows="products"
            styleClass="table"
            row-style-class="table__row"
          >
            <template slot="emptystate">
              <div
                class="table__heading table__heading--no-data flex flex--align-center flex--justify-center m--30"
              >
                {{ $t("order.no_products_in_order") }}
              </div>
            </template>
            <template slot="table-row" slot-scope="props">
              <div
                :class="{
                  'block-not-active':
                    products[props.index].product_id === null &&
                    props.column.field !== 'icons',
                }"
              >
                <template v-if="props.column.field === 'sku'">
                  <div>
                    {{ products[props.index].sku }}
                  </div>
                </template>
                <template v-else-if="props.column.field === 'name'">
                  <template v-if="products[props.index].href">
                    <a
                      class="table__action text-link"
                      :href="products[props.index].href"
                      target="_blank"
                      >{{ products[props.index].name }}
                      <icon
                        icon="foreign"
                        class="icon icon--foreign ml--5"
                      ></icon>
                    </a>
                  </template>
                  <span v-else>{{ products[props.index].name }}</span>
                </template>
                <template v-else-if="props.column.field === 'image'">
                  <div class="thumb">
                    <img
                      class="img-responsive"
                      :src="products[props.index].filemanager_thumb"
                    />
                  </div>
                </template>
                <template v-else-if="props.column.field === 'priceFormat'">
                  {{ productsPrice(products[props.index]) }}
                </template>
                <template v-else-if="props.column.field === 'quantity'">
                  <input
                    :class="{
                      'block-not-active':
                        products[props.index].product_id === null,
                    }"
                    :disabled="products[props.index].product_id === null"
                    type="text"
                    class="input input--sort-order"
                    v-model.number="products[props.index].quantity"
                  />
                </template>
                <template v-else-if="props.column.field === 'spec'">
                  <!-- <div v-if="products[props.index].specifications.length !== undefined">
                                            <div v-if="isProductSpecification(products[props.index].product_id, products[props.index].specifications)"> -->
                  <div
                    v-if="
                      products[props.index].specifications.length !== 0 &&
                      products[props.index].product_id !== null
                    "
                  >
                    <v-select
                      @input="
                        changeSpecificationProduct(0, products[props.index])
                      "
                      v-model="products[props.index].product_id"
                      label="text"
                      :close-on-select="true"
                      :options="products[props.index].specifications"
                      :reduce="(specification) => specification.product_id"
                      :searchable="false"
                      :clearable="false"
                      class="input"
                    >
                    </v-select>
                  </div>
                  <div v-else-if="products[props.index].product_id === null">
                    {{ Object.keys(products[props.index].specification)[0] }}
                    {{
                      products[props.index].specification[
                        Object.keys(products[props.index].specification)[0]
                      ]
                    }}
                  </div>
                  <div v-else>
                    {{ $t("order.no_specifications") }}
                  </div>
                  <!-- </div>
                                    </div> -->
                </template>
                <template v-else-if="props.column.field === 'total_format'">
                  <div>
                    {{ products[props.index].total_format }}
                  </div>
                </template>
                <template v-else-if="props.column.field === 'icons'">
                  <div>
                    <template v-if="products[props.index].refreshing">
                      <font-awesome-icon
                        class="text-warning color"
                        icon="circle-notch"
                        spin
                      ></font-awesome-icon>
                    </template>
                    <div v-else>
                      <icon
                        v-if="
                          (JSON.stringify(
                            order.products[props.index].quantity
                          ) !==
                            JSON.stringify(
                              order_products_save[props.index].quantity
                            ) &&
                            products[props.index].product_id !== null) ||
                          (JSON.stringify(
                            order.products[props.index].product_id
                          ) !==
                            JSON.stringify(
                              order_products_save[props.index].product_id
                            ) &&
                            products[props.index].product_id !== null)
                        "
                        @click.native.stop="
                          saveProductOrderChenges(
                            products[props.index],
                            props.index
                          )
                        "
                        icon="floppy-disk"
                        class="icon icon--export"
                      ></icon>
                      <icon
                        @click.native.stop="
                          deleteProductFromOrder(
                            products[props.index],
                            props.index
                          )
                        "
                        icon="delete"
                        class="icon icon--export"
                      ></icon>
                    </div>
                  </div>
                </template>
              </div>
            </template>
          </vue-good-table>

          <div class="flex flex--justify-space-between">
            <div class="mt--48">
              <a
                class="btn btn--cancel modal__btn js-open-modal"
                href="javascript:void(0)"
                @click.stop="editModalAddingProducts()"
                >{{ $t("order.аdd_product") }}</a
              >
            </div>
            <div>
              <table>
                <tbody>
                  <tr
                    class="table__row"
                    v-for="(subtotal, index) in subtotals"
                    :key="index"
                  >
                    <td class="table__value">
                      <div class="text">{{ subtotal.name }}</div>
                    </td>
                    <td class="table__value">
                      <input
                        v-model="subtotal.value"
                        type="text"
                        class="input input--sort-order"
                      />
                    </td>
                    <td class="table__value">
                      <v-select
                        v-model="subtotal.is_percent_effect"
                        label="name"
                        :options="subtotal_effect_types"
                        :searchable="false"
                        :clearable="false"
                        :reduce="(value) => value.value"
                        class="input input--sort-order vs--dropdown-options-subtotals"
                      >
                      </v-select>
                    </td>
                    <td class="table__value">
                      <template v-if="subtotal.refreshing">
                        <font-awesome-icon
                          class="text-warning color"
                          icon="circle-notch"
                          spin
                        ></font-awesome-icon>
                      </template>
                      <div v-else>
                        <icon
                          v-if="isChangedSubtotal(subtotal)"
                          @click.native="saveSubtotal(subtotal)"
                          icon="floppy-disk"
                          class="icon icon--export"
                        ></icon>
                      </div>
                    </td>
                  </tr>
                </tbody>
              </table>
              <!-- <div class="mt--48 singleFormGroup border-bottom">
                                <div class="flex flex--justify-space-between">
                                    <div class="text">{{$t("order.discount_on_order_amount")}}</div>
                                    <input type="text" class="input input--sort-order">
                                    <v-select
                                            v-model="discounts.sum_of_order"
                                            label="name"
                                            :close-on-select="true"
                                            :options="discounts_value"
                                            :searchable='false'
                                            :clearable="false"
                                            :reduce="value => value.type"
                                            class="input input--locale">
                                    </v-select> -->
              <!-- v-if="isChangedConfigurationRow(item.id, 1)"
                                      @click.native.stop="saveConfigurationsExportChenge(item)" -->
              <!-- <icon icon="floppy-disk" class="icon icon--export"></icon>
                        </div>
                        <div class="">
                            <div class="flex flex--justify-space-between">
                                <div class="text">{{$t("order.discount_from_the_manager")}}</div>
                                <input type="text" class="input input--sort-order">
                                <v-select
                                        v-model="discounts.manager"
                                        label="name"
                                        :close-on-select="true"
                                        :options="discounts_value"
                                        :searchable='false'
                                        :clearable="false"
                                        :reduce="value => value.type"
                                        class="input input--locale">
                                </v-select> -->
              <!-- v-if="isChangedConfigurationRow(item.id, 1)"
                                      @click.native.stop="saveConfigurationsExportChenge(item)" -->
              <!-- <icon icon="floppy-disk" class="icon icon--export"></icon>
                        </div>
                    </div>
                    </div>-->
              <div>
                <div
                  class="flex flex--justify-space-between singleFormGroup singleFormGroup--total-price mt--32"
                >
                  <div
                    class="singleFormGroup__title singleFormGroup__title--header"
                  >
                    {{ $t("columns.total") }}
                  </div>
                  <div
                    class="singleFormGroup__title singleFormGroup__title--header"
                  >
                    {{ order.total_format }}
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div>
      <div class="widget">
        <div class="tabs">
          <a
            @click="tab = 'status'"
            href="javascript:void(0)"
            class="tabs__heading"
            :class="{ 'tabs__heading--active': tab === 'status' }"
          >
            {{ $t("order.order_status") }}
          </a>
          <!--
                    <a @click="tab = 'history'" href="javascript:void(0)" class="tabs__heading"
                       :class="{'tabs__heading--active': tab === 'history'}">
                        {{$root.translateWords('Order history')}}
                    </a>
                    -->
        </div>
        <!--
                <div v-if="tab === 'history'" >
                     <div class="">
                        {{$root.translateWords('Order history')}}
                    </div>
                    <div class="mt--20 listData listData--noBg w100perc">
                        <vue-good-table
                            :columns="chengesColumns"
                            :rows="chenges"
                            styleClass="table"
                            row-style-class="table__row">
                            <template slot="table-row" slot-scope="props">
                                <template v-if="props.column.field === 'users'">
                                    {{chenges[props.index].user}}
                                </template>
                                <template v-if="props.column.field === 'data_time'">
                                   <span>
                                       {{chenges[props.index].data}} {{chenges[props.index].time}}
                                    </span> 
                                </template>
                                <template v-if="props.column.field === 'chenged'">
                                    <div>
                                        {{chenges[props.index].chenges_info}}
                                        <span class="text-link text-link--product-link text-link--underline">Lorem, ipsum.</span>
                                        <span class="text-link text-link--underline">Lorem, ipsum.</span>
                                    </div>
                                </template>
                            </template>
                        </vue-good-table>
                    </div>
                </div>
            -->
        <div
          v-if="tab === 'status'"
          class="mt--20 listData listData--noBg w100perc"
        >
          <vue-good-table
            :columns="historyColumns"
            :rows="histories"
            styleClass="table"
            row-style-class="table__row"
          >
            <template slot="table-row" slot-scope="props">
              <template v-if="props.column.field === 'status'">
                {{
                  histories[props.index].status !== null
                    ? histories[props.index].status.translate.name
                    : $t("order.missing_status")
                }}
              </template>
              <template v-else-if="props.column.field === 'notify'">
                <template v-if="parseInt(histories[props.index].notify)">
                  {{ $root.translate.columns.yes }}
                </template>
                <template v-else>
                  {{ $root.translate.columns.no }}
                </template>
              </template>
              <template v-else>
                {{ histories[props.index][props.column.field] }}
              </template>
            </template>
          </vue-good-table>
          <div class="w100perc mt--48 flex flex--justify-end">
            <a
              href="javascript:void(0)"
              class="btn btn--confirm js-open-modal"
              @click.stop="$root.changePopupShowStatus('history')"
            >
              {{ $root.translateWords("Add status") }}
            </a>
          </div>
        </div>
      </div>
    </div>

    <modal
      name="history"
      width="60%"
      :title="$root.translateWords('Add new status to order')"
    >
      <template v-slot>
        <div class="singleFormGroup">
          <div class="singleFormGroup__title">
            {{ $root.translate.columns.status }}:
          </div>
          <div class="singleFormGroup__field">
            <v-select
              :options="orderStatuses"
              v-model="newHistory.order_status_id"
              class="input"
              :reduce="(orderStatus) => orderStatus.id"
              label="name"
            >
              <div slot="no-options">
                {{ $root.translateWords("Data not found") }}
              </div>
            </v-select>
          </div>
        </div>
        <div class="singleFormGroup">
          <div class="singleFormGroup__title">
            {{ $root.translate.columns.comment }}:
          </div>
          <div class="singleFormGroup__field">
            <textarea
              class="input input--inForm w100perc"
              rows="10"
              v-model="newHistory.comment"
            ></textarea>
          </div>
        </div>
        <div class="flex flex--align-center">
          <div class="switcher">
            <check
              @click.native="newHistory.notify = !newHistory.notify"
              :checked="newHistory.notify === true"
              class="switcher__icon"
            ></check>
            <span
              @click="newHistory.notify = !newHistory.notify"
              class="switcher__label"
            >
              {{ $root.translateWords("Notify customer?") }}:
            </span>
            <icon icon="icon" class="icon switcher__help"></icon>
          </div>
        </div>
        <div class="flex mt--48">
          <a
            class="btn btn--cancel modal__btn js-open-modal"
            href="javascript:void(0)"
            @click="$root.changePopupShowStatus('history', false)"
            >{{ $root.translateWords("Cancel") }}</a
          >
          <a
            class="btn btn--confirm modal__btn"
            href="javascript:void(0)"
            @click="addHistory"
          >
            {{ $root.translateWords("Add status") }}
          </a>
        </div>
      </template>
    </modal>
    <!-- v-if="Object.keys(modalConfiguration).length" -->
    <modal
      name="customer-info"
      width="60%"
      v-if="!modal_customer_info.data.length"
      :title="$t('order.editing_customer_information')"
      @closed="closeModal"
    >
      <template v-slot>
        <div class="singleFormGroup">
          <h2 class="singleFormGroup__title singleFormGroup__title--header">
            {{ $t("order.registered_customer") }}
          </h2>

          <v-select
            @search="onSearchUsers"
            :multiple="false"
            :clearable="true"
            :searchable="true"
            :options="Users"
            :filterable="false"
            :slot-scope="Users"
            v-model="modal_customer_info.data.search_result"
            class="options vs--user border-bottom border-bottom--padding"
            placeholder="Поиск"
            label="email"
          >
            <template
              slot="option"
              slot-scope="option"
              class="vs__dropdown-option--height"
            >
              <div class="">
                <div>{{ option.email }}/{{ option.telephone }}</div>
                <div>{{ option.fullname }}</div>
              </div>
            </template>
            <template slot="selected-option" slot-scope="option">
              <div class="flex flex--justify-space-between flex--align-center">
                <div class="options">
                  <div>{{ option.email }}/{{ option.telephone }}</div>
                  <div>{{ option.fullname }}</div>
                </div>
              </div>
              <icon
                @click.native.stop="clearSearchResults()"
                icon="delete"
                class="icon icon--export"
              ></icon>
            </template>

            <!-- <template slot="option" slot-scope="option">
                            <div>
                                <div>{{option.email}}/{{option.telephone}}</div>
                                <div>{{option.fullname}}</div>
                            </div>
                        </template> -->
          </v-select>
        </div>

        <div class="singleFormGroup">
          <div
            v-if="!modal_customer_info.data.search_result"
            class="singleFormGroup__title singleFormGroup__title--header"
          >
            {{ $t("order.new_client") }}
          </div>
          <div class="flex flex--column singleFormGroup">
            <span class="singleFormGroup__title">
              {{ $t("columns.firstname") }}:
            </span>
            <input
              type="text"
              class="input"
              v-model="modal_customer_info.data.firstname"
            />
          </div>
          <div class="flex flex--column singleFormGroup">
            <span class="singleFormGroup__title">
              {{ $t("columns.surname") }}:
            </span>
            <input
              type="text"
              class="input"
              v-model="modal_customer_info.data.surname"
            />
          </div>
          <div class="flex flex--column singleFormGroup">
            <span class="singleFormGroup__title">
              {{ $t("columns.lastname") }}:
            </span>
            <input
              type="text"
              class="input"
              v-model="modal_customer_info.data.lastname"
            />
          </div>
          <div class="flex flex--column singleFormGroup">
            <span class="singleFormGroup__title"> Email: </span>
            <input
              type="text"
              class="input"
              v-model="modal_customer_info.data.email"
            />
          </div>
          <div class="flex flex--column singleFormGroup">
            <span class="singleFormGroup__title">
              {{ $root.translate.columns.telephone }}:
            </span>
            <input
              type="text"
              class="input"
              v-model="modal_customer_info.data.telephone"
            />
          </div>
        </div>
        <div
          class="flex flex--justify-end"
          v-if="
            JSON.stringify(modal_customer_info_save.data) !==
            JSON.stringify(modal_customer_info.data)
          "
        >
          <a
            @click.stop="saveChengesUser"
            href="javascript:void(0)"
            class="btn btn--confirm"
          >
            {{ $t("words.Save changes") }}
          </a>
        </div>
      </template>
    </modal>
    <modal
      name="shipping-and-payment"
      width="60%"
      :title="$t('order.editing_delivery_and_payment')"
      @closed="closeModal"
    >
      <template v-slot>
        <div class="">
          <div class="flex flex--column singleFormGroup">
            <span class="singleFormGroup__title">
              {{ $t("columns.shipping_method") }}:
            </span>
            <v-select
              :clearable="true"
              :searchable="true"
              :options="modal_shipping_and_payment.data.shipping"
              v-model="modal_shipping_and_payment.data.selected_shipping"
              :reduce="(shipping) => shipping.code"
              class="input input--shipping"
              label="decoded_method_name"
            />
          </div>

          <div
            v-for="shipping_method in modal_shipping_and_payment.data.shipping"
            :key="shipping_method.id"
          >
            <div
              v-if="
                shipping_method.code ===
                  modal_shipping_and_payment.data.selected_shipping &&
                shipping_method.has_api === false
              "
            >
              <div
                v-for="(item, index) in shipping_method.decoded_fields"
                :key="index"
              >
                <div class="flex flex--column singleFormGroup">
                  <span class="singleFormGroup__title">
                    {{ item.trans.name }}
                  </span>
                  <input
                    type="text"
                    class="input input--shipping"
                    v-model="item.value"
                  />
                </div>
              </div>
            </div>
            <div
              v-if="
                shipping_method.code ===
                  modal_shipping_and_payment.data.selected_shipping &&
                shipping_method.has_api === true
              "
            >
              <component
                :is="modal_shipping_and_payment.data.selected_shipping"
                :decode_value="shipping_method"
                :addres="order.address"
              ></component>
            </div>
          </div>

          <div class="flex flex--column singleFormGroup">
            <span class="singleFormGroup__title">
              {{ $t("columns.payment_method") }}:
            </span>
            <v-select
              v-model="modal_shipping_and_payment.data.payment_method"
              label="decoded_method_name"
              :close-on-select="true"
              :options="modal_shipping_and_payment.data.payment"
              :searchable="false"
              :clearable="false"
              class="input input--shipping select select--special"
            >
            </v-select>
          </div>
          <!-- <div class="form__validation form__validation--inline js-validation">
                        {{trans.errors.choose_single}}
                    </div> -->

          <div
            class="flex flex--justify-end"
            v-if="
              JSON.stringify(modal_shipping_and_payment.data) !==
              JSON.stringify(modal_shipping_and_payment_save.data)
            "
          >
            <a
              @click.stop="saveChengesShipping"
              href="javascript:void(0)"
              class="btn btn--confirm"
            >
              {{ $t("words.Save changes") }}
            </a>
          </div>
        </div>
      </template>
    </modal>
    <modal
      name="adding-products"
      width="80%"
      :title="$t('order.adding_products_to_order')"
      @closed="closeModal"
    >
      <template v-slot>
        <div class="filter">
          <div class="filter__name">
            <input
              @input="debounceFunction(onColumnFilter)"
              type="text"
              class="input"
              v-model="serverParams.name_sku"
              :placeholder="
                $root.translateWords('Search products by name or sku')
              "
            />
            <icon icon="search" class="icon filter__name_icon"></icon>
          </div>
          <advanced-filter
            :settings="{ offset_top: true }"
            :currencies="currencies"
            @filter="expandFilter"
            v-on:clear-filter-option="clearFilterParam"
          ></advanced-filter>
        </div>
        <vue-good-table
          :columns="productsListColumns"
          :rows="productsList"
          :isLoading.sync="isLoading"
          styleClass="table"
          row-style-class="table__row"
        >
          <template slot="table-row" slot-scope="props">
            <template v-if="props.column.field === 'sku'">
              <div>
                {{ productsList[props.index].sku }}
              </div>
            </template>
            <template v-else-if="props.column.field === 'name'">
              <template v-if="productsList[props.index].href">
                <a
                  class="table__action text-link"
                  :href="productsList[props.index].href"
                  target="_blank"
                  >{{ productsList[props.index].name }}
                  <icon icon="foreign" class="icon icon--foreign ml--5"></icon>
                </a>
              </template>
              <span v-else>{{ productsList[props.index].name }}</span>
            </template>
            <template v-else-if="props.column.field === 'image'">
              <div class="thumb">
                <img
                  class="img-responsive"
                  :src="productsList[props.index].filemanager_thumb"
                />
              </div>
            </template>
            <template v-else-if="props.column.field === 'product_balance'">
              {{ productsList[props.index].quantity }}
            </template>
            <template v-else-if="props.column.field === 'priceFormat'">
              {{
                productsList[props.index].vendor_price +
                " " +
                productsList[props.index].currency_code
              }}
            </template>
            <template v-else-if="props.column.field === 'quantity'">
              <div
                :class="{
                  'block-not-active': productsList[props.index].quantity <= 0,
                }"
              >
                <input
                  type="text"
                  class="input input--sort-order"
                  :disabled="productsList[props.index].quantity <= 0"
                  v-model.number="productsList[props.index].required_quantity"
                />
              </div>
            </template>
            <template v-else-if="props.column.field === 'spec'">
              <div class="block-not-active__not-disabled">
                <div
                  v-if="productsList[props.index].specifications.length !== 0"
                >
                  <v-select
                    @input="
                      changeSpecificationProduct(1, productsList[props.index])
                    "
                    v-model="productsList[props.index].id"
                    label="text"
                    :close-on-select="true"
                    :options="productsList[props.index].specifications"
                    :searchable="false"
                    :reduce="(specification) => specification.product_id"
                    :clearable="false"
                    class="input"
                  >
                  </v-select>
                </div>
                <div v-else>
                  {{ $t("order.no_specifications") }}
                </div>
              </div>
            </template>
            <!-- <template v-else-if="props.column.field === 'total_format'">
                            <div>
                                {{productsList[props.index].quantity * productsList[props.index].price }} грн
                            </div>
                        </template> -->
            <template v-else-if="props.column.field === 'icons'">
              <div v-if="productsList[props.index].quantity > 0">
                <icon
                  @click.native.stop="
                    addProductToOrder(productsList[props.index])
                  "
                  icon="plus"
                  class="icon icon--color icon--background-color"
                ></icon>
              </div>
              <div
                v-else
                :class="{
                  'block-not-active': productsList[props.index].quantity <= 0,
                }"
              >
                <icon
                  icon="plus"
                  class="icon icon--color icon--background-color icon--disable"
                ></icon>
              </div>
            </template>
          </template>
        </vue-good-table>
        <pagination
          v-if="serverParams.perPage < totalRecords"
          :current-page="serverParams.page"
          :per-page="serverParams.perPage"
          :total="totalRecords"
          :from-records="serverParams.fromRecords"
          :to-records="serverParams.toRecords"
          :pageChanged="onPageChange"
          :perPageChanged="onPerPageChange"
        >
        </pagination>
      </template>
    </modal>
    <widget-actions remove="deleteOrder"></widget-actions>
  </div>
</template>

<script>
import nova_poshta from "../admin/ComponentsOrderInfo/NewPost";

export default {
  name: "OrderInfo",
  components: {
    nova_poshta,
    "advanced-filter": require("./FilterComponent").default,
    pagination: require("./paginationComponent").default,
  },
  data() {
    return {
      isLoading: false,
      refreshing: false,
      tab: "status",
      order: {},
      order_products_save: [],
      orderStatuses: [],
      newHistory: {},
      Users: [],
      shipping: [],
      payment: [],
      productsColumns: [
        {
          label: this.$root.translate.columns.sku,
          field: "sku",
          thClass: "table__heading",
          tdClass: "table__value",
        },
        {
          label: this.$root.translate.columns.image_short,
          field: "image",
          thClass: "table__heading",
          tdClass: "table__value",
          sortable: false,
        },
        {
          label: this.$root.translate.columns.name,
          field: "name",
          thClass: "table__heading",
          tdClass: "table__value",
          sortable: false,
        },
        {
          label: this.$root.translate.columns.price,
          field: "priceFormat",
          thClass: "table__heading",
          tdClass: "table__value",
        },
        {
          label: this.$root.translate.columns.quantity,
          field: "quantity",
          thClass: "table__heading",
          tdClass: "table__value",
        },
        {
          label: this.$root.translate.columns.specification,
          field: "spec",
          thClass: "table__heading",
          tdClass: "table__value",
        },
        {
          label: this.$root.translate.columns.total,
          field: "total_format",
          thClass: "table__heading",
          tdClass: "table__value",
        },
        {
          label: "",
          field: "icons",
          thClass: "table__heading",
          tdClass: "table__value",
        },
      ],
      productsListColumns: [
        {
          label: this.$root.translate.columns.sku,
          field: "sku",
          thClass: "table__heading",
          tdClass: "table__value",
        },
        {
          label: this.$root.translate.columns.image_short,
          field: "image",
          thClass: "table__heading",
          tdClass: "table__value",
          sortable: false,
        },
        {
          label: this.$root.translate.columns.name,
          field: "name",
          thClass: "table__heading",
          tdClass: "table__value",
          sortable: false,
        },
        {
          label: this.$root.$t("order.residue"),
          field: "product_balance",
          thClass: "table__heading",
          tdClass: "table__value",
        },
        {
          label: this.$root.translate.columns.price,
          field: "priceFormat",
          thClass: "table__heading",
          tdClass: "table__value",
        },
        {
          label: this.$root.translate.columns.specification,
          field: "spec",
          thClass: "table__heading",
          tdClass: "table__value",
        },
        {
          label: this.$root.translate.columns.quantity,
          field: "quantity",
          thClass: "table__heading",
          tdClass: "table__value",
        },
        {
          label: "",
          field: "icons",
          thClass: "table__heading",
          tdClass: "table__value",
        },
      ],
      productsList: [],
      historyColumns: [
        {
          label: this.$root.translate.columns.status,
          field: "status",
          thClass: "table__heading",
          tdClass: "table__value",
        },
        {
          label: this.$root.translate.columns.created_at,
          field: "date_added",
          thClass: "table__heading",
          tdClass: "table__value",
        },
        {
          label: this.$root.translateWords("Notified?"),
          field: "notify",
          thClass: "table__heading",
          tdClass: "table__value",
        },
        {
          label: this.$root.translate.columns.comment,
          field: "comment",
          thClass: "table__heading",
          tdClass: "table__value",
        },
      ],
      chengesColumns: [
        {
          label: "Пользиватель",
          field: "users",
          thClass: "table__heading",
          tdClass: "table__value",
        },
        {
          label: "Дата / время",
          field: "data_time",
          thClass: "table__heading",
          tdClass: "table__value",
        },
        {
          label: "Изменения",
          field: "chenged",
          thClass: "table__heading",
          tdClass: "table__value",
        },
      ],
      chenges: [
        {
          user: "kort ons",
          data: "12.02.2020",
          time: "12:52:23",
          link: "www.google.com",
          chenges_info:
            "asdmkashduiashduihasuidhasiuhd CHENGET asdasdjasdh ashdh ahashd uasdh to sdasdasdasdasd",
        },
        {
          user: "kort HOST",
          data: "12.02.1800",
          time: "02:52:23",
          link: "www.google.com",
          chenges_info:
            "asdmkashduiashduihasuidhasiuhd CHENGET in 1800 asdasdjasdh ashdh ahashd uasdh to sdasdasdasdasd",
        },
        //class="text-link text-link--product-link text-link--underline" клас на просто підкреслене слово стандартно кольору тексту
        //class="text-link text-link--underline"  клас на link
      ],
      modal_customer_info: {
        data: {},
        edit: false,
        open: false,
      },
      modal_customer_info_save: {},
      modal_shipping_and_payment: {
        edit: false,
        open: false,
      },
      modal_shipping_and_payment_save: {},
      modal_adding_products: {
        edit: false,
        open: false,
      },
      subtotal_effect_types: [
        {
          name: this.$root.currency_code,
          value: false,
        },
        {
          name: "%",
          value: true,
        },
      ],
      subtotals: [],
      subtotals_saved: [],
      totalRecords: 0,
      serverParams: {
        columnFilters: {},
        sort_column: null,
        sort_direction: null,
        page: 1,
        perPage: 6,
        fromRecords: null,
        toRecords: null,
        with_specifications: true,
      },
      currencies: [],
    };
  },
  created() {
    let self = this;

    axios.get("/admin/currencies").then((httpResponse) => {
      this.currencies = httpResponse.data.currencies;
    });

    axios.get("/admin/orders/" + this.$route.params.id).then((httpResponse) => {
      //this.order = httpResponse.data;

      //this.$set(this.order, "address", this.order.address);

      self.$set(self, "order", httpResponse.data);

      self.order.products.forEach((item) => {
        self.$set(item, "refreshing", false);
      });

      this.$set(
        this,
        "order_products_save",
        this.$root.copy(this.order.products)
      );

      this.getSubtotalTemplates();
    });

    axios
      .get("/admin/order-statuses/", {
        params: {
          autocomplete: true,
        },
      })
      .then((httpResponse) => {
        this.orderStatuses = httpResponse.data.order_statuses;
      });
    this.getPayment();
    this.getShipping();
    this.getProductsList();
    this.clearValues();
  },
  computed: {
    products() {
      return this.order.products;
    },
    histories() {
      return this.order.histories;
    },
    isClientNew() {
      return this.modal_customer_info.data.search_result.length == null;
    },
    calculationProductSum(quantity, price) {
      return quantity * price;
    },
    isProductSpecification(id, specifications) {
      //console.log(specifications);
      //return id
      // if (specifications.length !== 0) {
      // let status = false;
      // specifications.forEach(item => {
      //     if (item.product_id === id) {
      //         status = true;
      //     }
      // })
      //     if (status === true) {
      //         return true
      //     }
      //     else{
      //        return false
      //     }
      // }
    },
    isShoppingHaveValue() {
      if (this.order.shipping || this.order.payment) {
        return true;
      }
    },
  },
  methods: {
    clearValues() {
      this.newHistory = {
        order_status_id: null,
        notify: false,
        comment: "",
        refreshing: false,
      };
    },
    addHistory() {
      this.refreshing = true;

      axios
        .post("/admin/orders/" + this.order.id + "/histories", this.newHistory)
        .then((httpResponse) => {
          this.$root.notify(httpResponse.data);

          axios
            .get("/admin/orders/" + this.order.id + "/histories")
            .then((httpResponse) => {
              this.$set(this.order, "histories", httpResponse.data.histories);

              this.clearValues();
            });

          this.$root.changePopupShowStatus("history", false);
        })
        .catch((error) => {
          if (error.response) {
            this.$root.notify(error.response.data);
          }
        })
        .finally(() => (this.refreshing = false));
    },
    deleteOrder() {
      this.$dialog
        .confirm(
          {
            title: this.$root.translateWords("Are you sure?"),
            body: this.$root.translateWords("This action cannot be undone"),
          },
          {
            view: "confirm-window", // can be set globally too
          }
        )
        .then(() => {
          this.refreshing = true;

          axios
            .delete("/admin/orders/" + this.order.id)
            .then((httpResponse) => {
              this.$router.push({ name: "orders" });
            });
        });
    },
    closeModal() {
      this.$root.changePopupShowStatus("customer-info", false);
      this.$root.changePopupShowStatus("shipping-and-payment", false);
      this.$root.changePopupShowStatus("adding-products", false);
      this.modal_customer_info.open = false;
      this.modal_adding_products.open = false;
      this.modal_shipping_and_payment.open = false;
    },
    editModalAddingProducts() {
      this.modal_adding_products.edit = true;
      this.modal_adding_products.open = true;
      this.$root.changePopupShowStatus("adding-products", true);
    },
    editModalCustomerInfo() {
      let self = this;
      //console.log(this.order.user.length);

      let configuration = {
        firstname: this.order.firstname,
        surname: this.order.surname,
        lastname: this.order.lastname,
        email: this.order.email,
        telephone: this.order.telephone,
        search_result: this.order.user,
        user_group_id: this.order.user_group_id,
        user_id: this.order.user_id,
      };
      // configuration[search_result] = "";
      if (this.order.user !== null) {
        console.log("null");
        this.Users.push(this.order.user);
      }
      this.modal_customer_info.edit = true;
      this.modal_customer_info.open = true;
      this.$set(this.modal_customer_info, "data", configuration);
      this.$root.changePopupShowStatus("customer-info", true);
      this.$set(
        this.modal_customer_info_save,
        "data",
        this.$root.copy(this.modal_customer_info.data)
      );
    },
    editModalShippingAndPayment() {
      let self = this;
      let shipping_method = "";
      if (this.order.shipping !== null) {
        shipping_method = this.order.shipping.code;
      }
      let configuration = {
        shipping: this.shipping,
        payment: this.payment,
        selected_shipping: shipping_method,
      };
      console.log(configuration, "conf");

      this.$root.copy(this.shipping);
      this.modal_shipping_and_payment.edit = true;
      this.modal_shipping_and_payment.open = true;
      this.$set(this.modal_shipping_and_payment, "data", configuration);
      this.$set(this.modal_shipping_and_payment.data, "bonus", "");
      this.$set(
        this.modal_shipping_and_payment.data,
        "payment",
        this.$root.copy(this.payment)
      );

      this.order.decoded_fields;

      this.modal_shipping_and_payment.data.shipping.forEach((item) => {
        if (item.code === self.order.shipping_code) {
          item.decoded_fields.forEach((elem) => {
            self.order.address.forEach((adderss_item) => {
              if (adderss_item.key === elem.key) {
                self.$set(elem, "value", adderss_item.value);
              }
            });
          });
        }
      });
      this.modal_shipping_and_payment.data.payment.forEach((item) => {
        if (item.code === self.order.payment_code) {
          self.$set(
            self.modal_shipping_and_payment.data,
            "payment_method",
            item
          );
        }
      });
      this.$set(
        this.modal_shipping_and_payment_save,
        "data",
        this.$root.copy(this.modal_shipping_and_payment.data)
      );
      this.$root.changePopupShowStatus("shipping-and-payment", true);
    },

    getSearchUsers: _.debounce((phrase, self, loading) => {
      axios
        .get("/admin/users", {
          params: {
            phrase: phrase,
            autocomplete: true,
          },
        })
        .then((Response) => {
          console.log(Response, "USERS");

          self.$set(self, "Users", Response.data.users);
          if (loading !== null) loading(false);
        });
    }, 500),

    onSearchUsers(phrase = null, loading = null) {
      if (loading !== null) loading(true);
      this.getSearchUsers(phrase, this, loading);
    },
    getPayment() {
      let self = this;
      axios.get("/admin/payment-methods").then((Response) => {
        console.log(Response, "getPayment");
        self.$set(self, "payment", Response.data);
      });
    },
    getShipping() {
      let self = this;
      axios.get("/admin/shipping-methods").then((Response) => {
        self.$set(self, "shipping", Response.data);
        self.shipping.forEach((element) => {
          element.decoded_fields.forEach((item) => {
            self.$set(item, "value", "");
          });
        });
      });
    },
    getProductsList(params = null) {
      let self = this;

      if (params !== null) {
        this.serverParams.sort_column = params[0].field;
        this.serverParams.sort_direction = params[0].type;
      }

      return axios
        .get("/admin/products", {
          params: this.serverParams,
        })
        .then((response) => {
          this.totalRecords = response.data.products.total;

          this.serverParams.fromRecords = response.data.products.from;
          this.serverParams.toRecords = response.data.products.to;

          if (response.data.products.data.length) {
            response.data.products.data.forEach((product) => {
              product.refreshing = false;

              product.translates.forEach((translates_elem) => {
                if (translates_elem.locale === self.$root.adminLanguage) {
                  product.name = translates_elem.name;
                }
              });
            });
          }

          this.isLoaded = true;
          this.$set(this, "productsList", response.data.products.data);
          self.productsList.forEach((item) => {
            // self.order.products.forEach(order_product => {
            //     if (order_product.product_id === item.id) {
            self.$set(item, "required_quantity", 1);
            console.log(0);
            // } else {
            //     console.log(1);
            //     self.$set(item, "required_quantity", 1);
            // } })
          });
        })
        .catch((error) => {
          if (error.response) this.$root.notify(error.response.data);
        });
    },
    saveChengesUser() {
      let self = this;

      let push_data = {
        user_id: this.modal_customer_info.data.search_result
          ? this.modal_customer_info.data.search_result.id
          : null,
        // user_group_id : self.modal_customer_info.data.user_group_id,
        firstname: self.modal_customer_info.data.firstname,
        surname: self.modal_customer_info.data.surname,
        lastname: self.modal_customer_info.data.lastname,
        email: self.modal_customer_info.data.email,
        telephone: self.modal_customer_info.data.telephone,
      };
      console.log(push_data, "put ");

      axios
        .put("/admin/orders/" + this.order.id + "/customer", push_data)
        .then((Response) => {
          this.$set(this.order, "user", Response.data.data.user);
          this.order.firstname = Response.data.data.firstname;
          this.order.lastname = Response.data.data.lastname;
          this.order.surname = Response.data.data.surname;
          this.order.email = Response.data.data.email;
          this.order.telephone = Response.data.data.telephone;
          this.order.FullName = Response.data.data.FullName;

          this.$root.notify(Response.data);
          this.closeModal();
        });
    },
    saveChengesShipping() {
      let self = this;
      let push_address = [];
      this.modal_shipping_and_payment.data.shipping.forEach((elem) => {
        if (
          elem.code === self.modal_shipping_and_payment.data.selected_shipping
        ) {
          if (elem.has_api === true) {
            push_address = elem.decoded_fields;
          } else {
            elem.decoded_fields.forEach((item) => {
              push_address.push({
                key: item.key,
                value: item.value,
              });
            });
          }
        }
      });
      axios
        .put("/admin/orders/" + this.order.id + "/shipping-payment", {
          payment_code:
            this.modal_shipping_and_payment.data.payment_method.code,
          shipping_code: this.modal_shipping_and_payment.data.selected_shipping,
          address: push_address,
        })
        .then((Response) => {
          self.order.decodeAddressInfo = Response.data.data.DecodeAddressInfo;
          self.shipping = self.modal_shipping_and_payment.data.shipping;
          if (self.order.shipping !== null) {
            self.order.shipping.code =
              self.modal_shipping_and_payment.data.selected_shipping;
          } else {
            self.$set(self.order, "shipping", {
              code: self.modal_shipping_and_payment.data.selected_shipping,
              name: self.modal_shipping_and_payment.data.payment_method
                .decoded_method_name,
            });
          }
          if (Response.data.data.address !== null) {
            self.$set(self.order, "address", Response.data.data.address);
          }
          self.order.payment =
            self.modal_shipping_and_payment.data.payment_method;
          self.order.payment.name = self.order.payment.decoded_method_name;
          self.order.payment_code =
            self.modal_shipping_and_payment.data.payment_method.code;
          self.shipping.forEach((elem) => {
            if (self.order.shipping.code === elem.code) {
              self.$set(self.order.shipping, "name", elem.decoded_method_name);
            }
          });
          this.$root.notify(Response.data);
          this.closeModal();
        });
      //console.log(address, "addres");
    },
    clearSearchResults() {
      this.modal_customer_info.data.search_result = "";
    },
    addProductToOrder(item) {
      let self = this;
      let status = false;
      let order_product_id;
      let quantity_in_order = 0;
      let index = this.order.products.findIndex(
        (product_elem) => product_elem.product_id === item.id
      );
      this.order.products.forEach((elem) => {
        if (item.id === elem.product_id) {
          status = true;
          order_product_id = elem.id;
          quantity_in_order = elem.quantity;
        }
      });
      if (status === false) {
        axios
          .post("/admin/orders/" + this.order.id + "/products", {
            product_id: item.id,
            required_quantity: item.required_quantity,
          })
          .then((Response) => {
            self.order.products.push(Response.data.data);
            self.order_products_save.push(self.$root.copy(Response.data.data));
            console.log(Response.data.total_format);
            item.quantity = Response.data.new_quantity;
            self.order.total_format = Response.data.total_format;
            this.$root.notify(Response.data);
          })
          .catch((error) => {
            if (error.response) this.$root.notify(error.response.data);
          })
          .finally(() => {
            item.refreshing = false;
          });
      } else if (status === true) {
        axios
          .put(
            "/admin/orders/" + this.order.id + "/products/" + order_product_id,
            {
              product_id: item.id,
              required_quantity: item.required_quantity + quantity_in_order,
            }
          )
          .then((Response) => {
            let id = Response.data.data.product_id;
            self.$set(
              self.order.products,
              index,
              this.$root.copy(Object.assign(item, Response.data.data))
            );
            item.id = id;
            self.$set(
              self.order_products_save,
              index,
              this.$root.copy(self.order.products[index])
            );
            self.order.total_format = Response.data.total_format;
            item.quantity = Response.data.new_quantity;
            this.$root.notify(Response.data);
          })
          .catch((error) => {
            if (error.response) this.$root.notify(error.response.data);
          })
          .finally(() => {
            item.refreshing = false;
          });
      }
      //                item.quantity = item.quantity - item.required_quantity;
    },
    deleteProductFromOrder(item, index) {
      let self = this;
      this.$dialog
        .confirm(
          {
            title: this.$root.translateWords("Are you sure?"),
            body: this.$root.translateWords("This action cannot be undone"),
          },
          {
            view: "confirm-window", // can be set globally too
          }
        )
        .then(() => {
          item.refreshing = true;
          axios
            .delete("/admin/orders/" + this.order.id + "/products/" + item.id)
            .then((Response) => {
              self.order.products.splice(index, 1);
              self.order_products_save.splice(index, 1);
              item.refreshing = false;
              self.order.total_format = Response.data.total_format;
              this.$root.notify(Response.data);
              self.productsList.forEach((product) => {
                if (product.id === item.product_id) {
                  product.quantity = Response.data.new_quantity;
                }
              });
            })
            .catch((error) => {
              if (error.response) this.$root.notify(error.response.data);
            })
            .finally(() => {
              item.refreshing = false;
            });
        });
    },
    changeSpecificationProduct(type, product) {
      let self = this;

      let product_id = type === 0 ? product.product_id : product.id;

      axios
        .get("/admin/orders/" + this.order.id + "/products/" + product_id, {
          params: {
            required_quantity: product.quantity,
          },
        })
        .then((Response) => {
          console.log(Response.data);
          let quantity = product.quantity;
          Object.assign(product, Response.data);
          if (type === 0) {
            product.quantity = quantity;
          }
          if (type === 1) {
            product.id = Response.data.product_id;

            self.order.products.forEach((item) => {
              if (Response.data.product_id === item.product_id) {
                product.required_quantity = item.quantity;
              } else {
                product.required_quantity = 1;
              }
            });
          }
        });
    },
    saveProductOrderChenges(item, index) {
      let self = this;
      item.refreshing = true;
      axios
        .put("/admin/orders/" + this.order.id + "/products/" + item.id, {
          product_id: item.product_id,
          required_quantity: item.quantity,
        })
        .then((Response) => {
          self.$set(
            self.order.products,
            index,
            Object.assign(item, Response.data.data)
          );
          console.log(self.order.products[index]);
          item.refreshing = false;
          self.$set(
            self.order_products_save,
            index,
            this.$root.copy(self.order.products[index])
          );
          //self.$set(self, "order_products_save[index]",self.$root.copy(item))
          self.order.total_format = Response.data.total_format;
          this.$root.notify(Response.data);
          self.productsList.forEach((product) => {
            if (product.id === item.product_id) {
              product.quantity = Response.data.new_quantity;
            }
          });
        })
        .catch((error) => {
          if (error.response) this.$root.notify(error.response.data);
        })
        .finally(() => {
          item.refreshing = false;
        });
    },
    productsPrice(item) {
      if (item.special !== 0 && item.special !== null) {
        return item.specialFormat;
      } else {
        return item.priceFormat;
      }
    },
    getSubtotalTemplates() {
      let self = this;

      axios
        .get("/admin/subtotals", {
          params: {
            only_templates: true,
          },
        })
        .then((httpResponse) => {
          httpResponse.data.forEach((subtotals_template) => {
            let subtotal_from_order = self.order.subtotals.find((subtotal) => {
              return subtotal.subtotal_template_id === subtotals_template.id;
            });

            let subtotal = {
              id: subtotal_from_order ? subtotal_from_order.id : null,
              subtotal_template_id: subtotal_from_order
                ? subtotal_from_order.subtotal_template_id
                : subtotals_template.id,
              order_id: subtotal_from_order
                ? subtotal_from_order.order_id
                : null,
              code: subtotal_from_order
                ? subtotal_from_order.code
                : subtotals_template.code,
              name: subtotal_from_order
                ? subtotal_from_order.name
                : subtotals_template.decoded_title,
              value: subtotal_from_order ? subtotal_from_order.value : null,
              is_percent_effect: subtotal_from_order
                ? subtotal_from_order.is_percent_effect
                : false,
              direction: subtotal_from_order
                ? subtotal_from_order.direction
                : subtotals_template.direction,
              refreshing: false,
            };

            self.subtotals.push(subtotal);

            if (subtotal.id)
              self.subtotals_saved.push(self.$root.copy(subtotal));
          });
        })
        .catch((error) => {
          if (error.response) this.$root.notify(error.response.data);
        });
    },
    isChangedSubtotal(subtotal) {
      let subtotal_saved = this.subtotals_saved.find((subtotal_saved) => {
        return subtotal_saved.id === subtotal.id;
      });

      return JSON.stringify(subtotal) !== JSON.stringify(subtotal_saved);
    },
    saveSubtotal(subtotal) {
      let is_new = subtotal.id === null;

      let request;

      subtotal.refreshing = true;

      if (is_new) {
        request = axios.post(
          "/admin/orders/" + this.order.id + "/subtotals",
          subtotal
        );
      } else {
        request = axios.put(
          "/admin/orders/" + this.order.id + "/subtotals/" + subtotal.id,
          subtotal
        );
      }

      request
        .then((httpResponse) => {
          subtotal.refreshing = false;

          if (is_new) {
            subtotal.id = httpResponse.data.id;

            this.subtotals_saved.push(this.$root.copy(subtotal));
          } else {
            let subtotal_saved_index = this.subtotals_saved.findIndex(
              (subtotal_saved) => {
                return subtotal_saved.id === subtotal.id;
              }
            );

            this.$set(
              this.subtotals_saved,
              subtotal_saved_index,
              this.$root.copy(subtotal)
            );
          }

          this.order.total = httpResponse.data.total;
          this.order.total_format = httpResponse.data.total_format;
        })
        .catch((error) => {
          if (error.response) this.$root.notify(error.response.data);
        })
        .finally(() => {
          subtotal.refreshing = false;
        });
    },
    updateParams(newProps) {
      this.serverParams = Object.assign({}, this.serverParams, newProps);
    },

    onPageChange(params) {
      this.updateParams({ page: params.currentPage });

      this.getProductsList();
    },

    onPerPageChange(params) {
      this.updateParams({ perPage: params.currentPerPage });
      this.getProductsList();
    },

    onSortChange(params) {
      this.serverParams.sort_direction = params[0].type;
      this.serverParams.sort_column = params[0].field;

      this.getProductsList();
    },

    onColumnFilter(params) {
      this.updateParams(params);
      this.getProductsList();
    },
    debounceFunction: _.debounce(function (userFunction) {
      userFunction();
    }, 1500),

    expandFilter(attributes = {}) {
      this.updateParams(attributes);

      this.getProductsList();
    },
    clearFilterParam(param) {
      if (Object(this.serverParams).hasOwnProperty(param)) {
        delete this.serverParams[param];
      }
    },

    //debounceFunction: _.debounce(function (userFunction) {
    //    userFunction();
    //}, 1500),
    //onColumnFilter(params) {
    //    this.updateParams(params);
    //    this.loadItems();
    //},
    //  getSearchProducts: _.debounce((phrase, self, loading) => {
    //              axios.get('/admin/users', {
    //                  params: {
    //                      phrase: phrase,
    //                  }
    //              }).then(Response => {
    //                  console.log(Response, "USERS");

    //                     self.$set(self, "productsList", Response.data.users);
    //                     if (loading !== null) loading(false);
    //                 });
    //         }, 500),
    // onSearchProducts(phrase = null, loading = null) {
    //             if (loading !== null) loading(true);
    //             this.getSearchProducts(phrase, this, loading);
    // },
  },
};
</script>

<style scoped></style>

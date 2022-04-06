<template>
    <div class="table-list-container" v-if="Object.keys(product).length">
        <div class="flex flex--justify-space-between">
            <div>
                <div class="breadcrumbs">
                    <router-link class="breadcrumbs__link" :to="{name: 'dashboard'}">{{$root.translate.columns.home}}
                    </router-link>
                    -
                    <router-link class="breadcrumbs__link" :to="{name: 'products'}">
                        {{$root.translate.menu.catalog.items.products}}
                    </router-link>
                </div>
                <template v-if="product.type === 1">
                    <h2 class="mainContent__heading mainContent__heading--inProduct">
                        {{product.translates.find(translate => {return translate.locale ==
                        $root.adminLanguage}).name}}</h2>
                    <div class="flex flex--align-center">
                        <div class="switcher">
                            <check @click.native="product.type = 1" :checked="product.type === 1"
                                   class="switcher__icon"></check>
                            <span @click="product.type = 1"
                                  class="switcher__label">{{$root.translateWords('Simple product')}}</span>
                            <icon icon="icon" class="icon switcher__help"></icon>
                        </div>
                        <div class="switcher">
                            <check @click.native="checkProductTypeForVariants"
                                   class="switcher__icon"></check>
                            <span @click="checkProductTypeForVariants" class="switcher__label">{{$root.translateWords('With variants')}}</span>

                            <v-popover offset="6" placement="right" popoverClass="popover">

                                <icon icon="icon" class="icon switcher__help popover__icon"></icon>

                                <template slot="popover">
                                    <p class="popover__content">
                                        content
                                    </p>
                                    <a class="popover__close sb-icon-cancel" v-close-popover></a>
                                </template>

                            </v-popover>

                        </div>
                    </div>
                </template>
            </div>
            <div>
                <div class="singleFormGroup">
                    <div class="singleFormGroup__title">
                        {{$root.translate.columns.status}}:
                    </div>
                    <div class="singleFormGroup__field">
                        <div class="switcherStatus">
                            <div @click="product.status = false" class="switcherStatus__value"
                                 :class="{'switcherStatus__value--active switcherStatus__value--active_off': product.status === false}">
                                {{$root.translate.columns.disabled_short}}
                            </div>
                            <div @click="product.status = true" class="switcherStatus__value"
                                 :class="{'switcherStatus__value--active': product.status === true}">
                                {{$root.translate.columns.enabled_short}}
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <div class="singleForm">
            <div class="tabs">
                <a @click="tab = 'main'" href="javascript:void(0)" class="tabs__heading"
                   :class="{'tabs__heading--active': tab === 'main'}">
                    {{$root.translate.columns.main}}
                </a>
                <a v-if="product.type === 2" @click="tab = 'variants'" href="javascript:void(0)" class="tabs__heading"
                   :class="{'tabs__heading--active': tab === 'variants'}">
                    {{$root.translate.columns.variants}}
                </a>
                <a @click="tab = 'relations'" href="javascript:void(0)" class="tabs__heading"
                   :class="{'tabs__heading--active': tab === 'relations'}">
                    {{$root.translate.columns.relations}}
                </a>
                <a @click="tab = 'attributes'" href="javascript:void(0)" class="tabs__heading"
                   :class="{'tabs__heading--active': tab === 'attributes'}">
                    {{$root.translate.columns.attributes}}
                </a>
                <a v-if="product.type !== 2" @click="tab = 'specials'" href="javascript:void(0)" class="tabs__heading"
                   :class="{'tabs__heading--active': tab === 'specials'}">
                    {{$root.translate.columns.specials}}
                </a>
                <a v-if="product.type !== 2" @click="tab = 'discounts'" href="javascript:void(0)" class="tabs__heading"
                   :class="{'tabs__heading--active': tab === 'discounts'}">
                    {{$root.translate.columns.discounts}}
                </a>
                <a v-if="product.type !== 2" @click="tab = 'seo'" href="javascript:void(0)" class="tabs__heading"
                   :class="{'tabs__heading--active': tab === 'seo'}">
                    Seo
                </a>
            </div>
            <div class="singleForm__content">

                <div v-show="tab === 'variants'">
                    <products-variations-component
                            ref="productVariants"
                            :product="product"
                            :user-groups="userGroups"
                            :currencies="currencies"
                            :attributes="attributes"
                            :attribute_values="attribute_values"
                            :add-row-to-attributes="addToAttribute"
                            :check-attribute-exist="checkAttributeExist"
                            :check-attribute-value-exist="checkAttributeValueExist"
                            :set-attribute-search-phrase="setAttributeSearchPhrase"
                            :create-attribute="createAttribute"
                            :set-attribute-value-search-phrase="setAttributeValueSearchPhrase"
                            :create-attribute-value="createAttributeValue"
                            :remove-to-attribute="removeToAttribute"
                            :update-product-original="updateProductOriginal"
                            :drag-options="dragOptions"
                            :drag="drag"
                            :is-changed-info="isChangedInfo"
                            :update-product="updateProduct"
                            :delete-product="deleteProduct"
                            :columns-for-specials="columnsForSpecials"
                            :columns-for-discounts="columnsForDiscounts"
                            :clear-attribute-search-phrase="clearAttributeSearchPhrase"
                            :clear-attribute-value-search-phrase="clearAttributeValueSearchPhrase"

                    ></products-variations-component>
                </div>

                <template v-if="tab === 'main'">
                    <div class="flex flex--justify-space-between">
                        <div>
                            <template v-if="product.type !== 2">
                                <div class="singleFormGroup">
                                    <div class="singleFormGroup__title">
                                        {{$root.translate.columns.name}}:
                                    </div>
                                    <div class="singleFormGroup__field inputWithTranslates" v-for="translate in product.translates">
                                        <div class="flex flex--align-center">
                                            {{$root.languages.find((language) => {return language.locale ===
                                            translate.locale}).name}}:
                                            <input type="text" class="input input--inForm input--label_left"
                                                   v-model="translate.name">
                                        </div>
                                    </div>
                                </div>
                                <div class="singleFormGroup">
                                    <div class="singleFormGroup__title">
                                        {{$root.translate.columns.sku}}:
                                    </div>
                                    <div class="singleFormGroup__field">
                                        <div class="flex flex--align-center">
                                            <input type="text" class="input input--inForm"
                                                   v-model="product.sku">
                                        </div>
                                    </div>
                                </div>
                            </template>

                            <div class="singleFormGroup singleFormGroup--inlineSet">
                                <div class="singleFormGroup__set" v-if="product.type !== 2">
                                    <div class="singleFormGroup__title">
                                        {{$root.translate.columns.price}}:
                                    </div>
                                    <div class="singleFormGroup__field">
                                        <div class="flex flex--align-center">
                                            <input type="text" class="input input--price_inProduct"
                                                   v-model.number="product.vendor_price">
                                        </div>
                                    </div>
                                </div>
                                <div class="singleFormGroup__set">
                                    <div class="singleFormGroup__title">
                                        {{$root.translate.columns.currency}}:
                                        <icon icon="icon" class="icon singleFormGroup__title_helper"></icon>
                                    </div>
                                    <div class="singleFormGroup__field">
                                        <div class="flex flex--align-center">
                                            <v-select
                                                    :clearable="false"
                                                    :searchable="false"
                                                    :options="currencies"
                                                    v-model="product.currency_code"
                                                    :reduce="currency => currency.code"
                                                    class="input input--currency"
                                                    label="code">
                                            </v-select>
                                        </div>
                                    </div>
                                </div>
                                <div class="singleFormGroup__set">
                                    <div class="singleFormGroup__title">
                                        {{$root.translateWords('Price unit')}}:
                                        <icon icon="icon" class="icon singleFormGroup__title_helper"></icon>
                                    </div>
                                    <div class="singleFormGroup__field">
                                        <div class="flex flex--align-center">
                                            <v-select
                                                    :clearable="true"
                                                    :searchable="false"
                                                    :options="priceUnits"
                                                    v-model="product.price_unit_id"
                                                    :reduce="priceUnit => priceUnit.id"
                                                    class="input input--inForm input--price-unit"
                                                    label="name">
                                                <div slot="no-options">{{$root.translateWords('Data not found')}}</div>
                                            </v-select>
                                        </div>
                                    </div>
                                </div>
                            </div>
                            <div class="singleFormGroup" v-if="product.type !== 2">
                                <div class="singleFormGroup__title">
                                    {{$root.translateWords('Stock quantity')}}:
                                    <icon icon="icon" class="icon singleFormGroup__title_helper"></icon>
                                </div>
                                <div class="singleFormGroup__field">
                                    <div class="flex flex--align-center">
                                        <input type="number" class="input input--inForm"
                                               v-model.number="product.quantity">
                                    </div>
                                </div>
                            </div>
                            <div class="singleFormGroup">
                                <div class="singleFormGroup__title">
                                    {{$root.translateWords('Min product quantity for order')}}:
                                    <icon icon="icon" class="icon singleFormGroup__title_helper"></icon>
                                </div>
                                <div class="singleFormGroup__field">
                                    <div class="flex flex--align-center">
                                        <input type="number" class="input input--inForm"
                                               v-model.number="product.minimum">
                                    </div>
                                </div>
                            </div>
                            <div class="singleFormGroup">
                                <div class="singleFormGroup__title">
                                    {{$root.translateWords('Stock status')}}:
                                    <icon icon="icon" class="icon singleFormGroup__title_helper"></icon>
                                </div>
                                <div class="singleFormGroup__field">
                                    <div class="flex flex--align-center">
                                        <v-select
                                                :clearable="true"
                                                :searchable="false"
                                                :options="stockStatuses"
                                                v-model="product.stock_status_id"
                                                :reduce="stockStatus => stockStatus.id"
                                                class="input input--inForm"
                                                label="title">
                                            <div slot="no-options">{{$root.translateWords('Data not found')}}</div>
                                        </v-select>
                                    </div>
                                </div>
                            </div>
                            <div class="singleFormGroup">
                                <div class="singleFormGroup__title">
                                    {{$root.translate.columns['sort-order']}}:
                                    <icon icon="icon" class="icon singleFormGroup__title_helper"></icon>
                                </div>
                                <div class="singleFormGroup__field">
                                    <div class="flex flex--align-center">
                                        <input type="number" class="input input--inForm"
                                               v-model.number="product.sort_order">
                                    </div>
                                </div>
                            </div>
                            <div class="singleFormGroup" v-if="product.type !== 2">
                                <div class="singleFormGroup__title">
                                    {{$root.translate.columns.description}}:
                                </div>
                                <div class="singleFormGroup__field inputWithTranslates" v-for="translate in product.translates">
                                    <div class="flex">
                                <span class="mr--8">
                                    {{$root.languages.find((language) => {return language.locale == translate.locale}).name}}:
                                </span>
                                        <vue-ckeditor
                                                :config="$root.editorConfig"
                                                v-model="translate.description">
                                        </vue-ckeditor>
                                    </div>
                                </div>
                            </div>
                            <div class="singleFormGroup">
                                <div class="singleFormGroup__title">
                                    {{$root.translate.columns.date_available}}:
                                </div>
                                <div class="singleFormGroup__field">
                                    <div class="flex flex--align-center">
                                        <datetime input-class="input input--date"
                                                  v-model="product.date_available_js"
                                                  type="datetime"
                                                  :phrases="datetime.phrases"
                                                  format="dd.MM.yyyy HH:mm"/>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div v-if="product.type !== 2">
                            <div class="imagesSet">
                                <div class="imagesSet__title">
                                    {{$root.translateWords('Product images')}}:
                                    <icon icon="icon" class="icon singleFormGroup__title_helper"></icon>
                                </div>
                                <draggable
                                        v-model="product.images"
                                        v-bind="dragOptions"
                                        @start="drag = true"
                                        @end="drag = false"
                                        tag="transition-group"
                                        class="imagesSet__items"
                                        :componentData="{props: {
                                            type: 'transition',
                                            name: !drag ? 'flip-list' : null
                                        }}">

                                    <div class=""
                                         v-for="(element, number) in product.images"
                                         :key="number">
                                        <upload-thumb
                                                items_type="products"
                                                :item="$root.encodeId('products',product.id)"
                                                :ref="'additionalThumb'"
                                                :data="element"
                                                attribute-src-name="src"
                                                :file_path="element.src"
                                                :thumb_path="element.filemanager_thumb"
                                                @remove="product.images.splice(number, 1)"
                                        ></upload-thumb>
                                    </div>

                                    <div @click="openUploadWindow" class="imagesSet__new" slot="footer" key="footer">
                                        <icon icon="plus" class="icon"></icon>
                                    </div>

                                </draggable>
                            </div>
                        </div>
                    </div>
                </template>

                <template v-else-if="tab === 'relations'">
                    <div class="singleFormGroup">
                        <div class="singleFormGroup__title">
                            {{$root.translateWords('Show in categories:')}}
                            <icon icon="icon" class="icon singleFormGroup__title_helper"></icon>
                        </div>
                        <div class="singleFormGroup__field">
                            <div class="flex flex--align-center">
                                <v-select
                                        @input="changeCategoriesList"
                                        @search="onSearchCategories"
                                        :components="{Deselect, OpenIndicator}"
                                        :multiple="true"
                                        :clearable="true"
                                        :searchable="true"
                                        :options="categories"
                                        v-model="product.categories"
                                        class="input input--inForm vs--multiply"
                                        label="name">
                                    <template #search="{attributes, events}">
                                        <input
                                                class="vs__search"
                                                :placeholder="$root.translateWords('Add category')"
                                                v-bind="attributes"
                                                v-on="events"
                                        />
                                    </template>
                                    <div slot="no-options">{{$root.translateWords('Data not found')}}</div>
                                </v-select>
                            </div>
                        </div>
                    </div>
                    <div class="singleFormGroup">
                        <div class="singleFormGroup__title">
                            {{$root.translateWords('Compare in categories:')}}
                            <icon icon="icon" class="icon singleFormGroup__title_helper"></icon>
                        </div>
                        <div class="singleFormGroup__field">
                            <div class="flex flex--align-center">
                                <v-select
                                        @search="onSearchCategories"
                                        :components="{Deselect, OpenIndicator}"
                                        :multiple="true"
                                        :clearable="true"
                                        :searchable="true"
                                        :options="categories"
                                        v-model="product.compare_categories"
                                        class="input input--inForm vs--multiply"
                                        label="name">
                                    <template #search="{attributes, events}">
                                        <input
                                                class="vs__search"
                                                :placeholder="$root.translateWords('Add category')"
                                                v-bind="attributes"
                                                v-on="events"
                                        />
                                    </template>
                                    <div slot="no-options">{{$root.translateWords('Data not found')}}</div>
                                </v-select>
                            </div>
                        </div>
                    </div>
                    <div class="singleFormGroup">
                        <div class="singleFormGroup__title">
                            {{$root.translateWords('Main category')}}:
                            <icon icon="icon" class="icon singleFormGroup__title_helper"></icon>
                        </div>
                        <div class="singleFormGroup__field">
                            <div class="flex flex--align-center">
                                <v-select
                                        :clearable="true"
                                        :searchable="true"
                                        :options="product.categories"
                                        v-model="product.category_id"
                                        class="input input--inForm"
                                        :reduce="category => category.id"
                                        label="name">
                                    <div slot="no-options">{{$root.translateWords('Data not found')}}</div>
                                </v-select>
                            </div>
                        </div>
                    </div>
                    <div class="singleFormGroup">
                        <div class="singleFormGroup__title">
                            {{$root.translate.columns.vendor}}:
                            <icon icon="icon" class="icon singleFormGroup__title_helper"></icon>
                        </div>
                        <div class="singleFormGroup__field">
                            <div class="flex flex--align-center">
                                <input type="text" class="input input--inForm"
                                       v-model="product.vendor">
                            </div>
                        </div>
                    </div>

                    <div class="singleFormGroup">
                        <div class="singleFormGroup__title">
                            {{$root.translate.columns.assortment.list.related}}
                            <icon icon="icon" class="icon singleFormGroup__title_helper"></icon>
                        </div>
                        <div class="singleFormGroup__field">

                            <div class="flex flex--align-center searchResultsItem searchResultsItem--related"
                                 v-for="(related, rel_index) in product.relateds">
                                <div class="flex flex--align-center">
                                    <img class="searchResultsItem__img"
                                         :src="related.image" alt="" style="max-width: 41px">
                                    <div class="flex flex--column flex--justify-space-between">
                                        <div class="searchResultsItem__name">{{related.name}}</div>
                                        <div class="searchResultsItem__sku">
                                            {{$root.translate.columns.sku}}: {{related.sku}}
                                        </div>
                                    </div>
                                </div>
                                <div>
                                    <icon @click.native.stop="deleteProductParameters('relateds',rel_index)" icon="delete"
                                          class="icon searchResultsItem__clear"></icon>
                                </div>
                            </div>

                            <div class="flex flex--align-center">

                                <v-select
                                        @search="onSearchProducts"
                                        :components="{Deselect, OpenIndicator}"
                                        :multiple="true"
                                        :clearable="true"
                                        :options="products"
                                        :filterable="false"
                                        v-model="product.relateds"
                                        class="input input--inForm vs--products vs--multiply"
                                        label="id">

                                    <template #search="{attributes, events}">

                                        <div class="flex vs__searchContainer">

                                            <div class="vs__searchIcon">
                                                +
                                            </div>

                                            <input class="vs__search"
                                                   :placeholder="$root.translateWords('Add')"
                                                   v-bind="attributes"
                                                   v-on="events"/>
                                        </div>
                                    </template>

                                    <template v-slot:selected-option-container>
                                        <slot name="selected-option">
                                            <div></div>
                                        </slot>
                                    </template>

                                    <template v-slot:option="option">
                                        <div class="flex flex--align-center searchResultsItem searchResultsItem--product">
                                            <img class="searchResultsItem__img"
                                                 :src="option.image" style="max-width: 39px" alt="">
                                            <div class="flex flex--column flex--justify-space-between">
                                                <div class="searchResultsItem__name">{{option.name}}</div>
                                                <div class="searchResultsItem__sku">
                                                    {{$root.translate.columns.sku}}: {{option.sku}}
                                                </div>
                                            </div>
                                        </div>
                                    </template>

                                    <div slot="no-options">{{$root.translateWords('Data not found')}}</div>

                                </v-select>
                            </div>
                        </div>
                    </div>

                    <div class="singleFormGroup">
                        <div class="singleFormGroup__title">
                            {{$root.translate.columns.source_id}}:
                            <icon icon="icon" class="icon singleFormGroup__title_helper"></icon>
                        </div>
                        <div class="singleFormGroup__field">
                            <div class="flex flex--align-center">
                                <input type="text" class="input input--inForm"
                                       v-model="product.extra_1">
                            </div>
                        </div>
                    </div>

                    <div class="singleFormGroup">
                        <div class="singleFormGroup__title">
                            {{$root.translate.columns.source_name}}:
                            <icon icon="icon" class="icon singleFormGroup__title_helper"></icon>
                        </div>
                        <div class="singleFormGroup__field">
                            <div class="flex flex--align-center">
                                <input type="text" class="input input--inForm"
                                       v-model="product.extra_2">
                            </div>
                        </div>
                    </div>

                </template>

                <template v-else-if="tab === 'attributes'">
                    <div class="grid grid--2 grid--gap_40"
                         v-for="to_attribute  in product.to_attributes.filter(to_attribute => to_attribute.has_variant === false)">
                        <div>
                            <div class="singleFormGroup mb--50">
                                <div class="singleFormGroup__title">
                                    {{$root.translate.columns.attributes}}:
                                    <icon icon="icon" class="icon singleFormGroup__title_helper"></icon>
                                </div>
                            </div>
                            <div class="singleFormGroup">
                                <div class="singleFormGroup__field">
                                    <div class="flex flex--align-center mb--10">
                                        <v-select
                                                :clear-search-on-select="true"
                                                taggable
                                                :clearable="true"
                                                :searchable="true"
                                                :options="attributes.filter(attribute => product.to_attributes.map(p_to_attribute => p_to_attribute.attribute).indexOf(attribute) === -1)"
                                                v-model="to_attribute.attribute"
                                                @input="clearAttributeSearchPhrase(to_attribute.id)"
                                                class="input input--inForm"
                                                label="name">
                                            <template #search="{attributes, events}">
                                                <input
                                                        @input="setAttributeSearchPhrase(to_attribute.id)"
                                                        class="vs__search"
                                                        v-bind="attributes"
                                                        v-on="events"
                                                />
                                            </template>
                                            <div slot="no-options">{{$root.translateWords('Data not found')}}</div>
                                        </v-select>
                                        <a v-if="!checkAttributeExist(to_attribute.id)"
                                           v-tooltip.top-start="'You have new messages.'"
                                           class="singleFormGroup__action" href="javascript:void(0)"
                                           @click.stop="createAttribute(to_attribute.id)">
                                            <icon icon="floppy-disk" class="icon"></icon>
                                        </a>
                                        <a v-tooltip.top-start="'You have new messages.'"
                                           class="singleFormGroup__action" href="javascript:void(0)"
                                           @click.stop="removeToAttribute(to_attribute.id)">
                                            <icon icon="delete" class="icon"></icon>
                                        </a>

                                    </div>
                                    <div class="flex mb--10">
                                        <div class="switcher">
                                            <check @click.native="to_attribute.main = !to_attribute.main"
                                                   :checked="to_attribute.main === true"
                                                   class="switcher__icon"></check>
                                            <span @click="to_attribute.main = !to_attribute.main"
                                                  class="switcher__label">
                                                {{$root.translateWords('Use in list of primary attributes?')}}
                                            </span>
                                            <icon icon="icon" class="icon switcher__help"></icon>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div>
                            <div class="singleFormGroup mb--50">
                                <div class="singleFormGroup__title">
                                    {{$root.translate.columns.values}}:
                                    <icon icon="icon" class="icon singleFormGroup__title_helper"></icon>
                                </div>
                            </div>
                            <div class="singleFormGroup">
                                <div class="singleFormGroup__field">
                                    <div class="flex flex--align-center">
                                        <template v-if="to_attribute.attribute !== null && to_attribute.attribute.id">
                                            <v-select
                                                    multiple
                                                    :clearable="true"
                                                    :searchable="true"
                                                    :clear-search-on-select="true"
                                                    :options="attribute_values.filter(attribute_value => attribute_value.attribute_id === to_attribute.attribute.id && to_attribute.values.indexOf(attribute_value) === -1)"
                                                    @input="clearAttributeValueSearchPhrase(to_attribute.id)"
                                                    v-model="to_attribute.values"
                                                    class="input input--inForm vs--multiply pt--0"
                                                    :components="{Deselect, OpenIndicator}"
                                                    :ref="'to_attributes_v'+to_attribute.id"
                                                    label="value">
                                                <template #search="{attributes, events}">
                                                    <input
                                                            :placeholder="$root.translateWords('Create value')"
                                                            @input="setAttributeValueSearchPhrase(to_attribute.id)"
                                                            class="vs__search"
                                                            v-bind="attributes"
                                                            v-on="events"
                                                    />
                                                </template>
                                                <div slot="no-options">{{$root.translateWords('Data not found')}}</div>
                                            </v-select>
                                            <template
                                                    v-if="to_attribute.values.length || (Object(to_attribute).hasOwnProperty('attributeValueSearchPhrase') && to_attribute.attributeValueSearchPhrase.length)">
                                                <a v-if="!checkAttributeValueExist(to_attribute.id)"
                                                   v-tooltip.top-start="'You have new messages.'"
                                                   class="singleFormGroup__action" href="javascript:void(0)"
                                                   @click.stop="createAttributeValue(to_attribute.id, 'to_attributes_v'+to_attribute.id, false)">
                                                    <icon icon="floppy-disk" class="icon"></icon>
                                                </a>
                                            </template>
                                        </template>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </template>

                <template v-else-if="tab === 'specials'">
                    <div class="listData listData--noBg" v-if="product.specials.length">
                        <vue-good-table
                                :columns="columnsForSpecials"
                                :rows="product.specials"
                                styleClass="table"
                                row-style-class="table__row">
                            <template slot="table-row" slot-scope="props">
                                <template v-if="props.column.field === 'user_group_id'">
                                    <v-select
                                            :clearable="false"
                                            :searchable="true"
                                            :options="userGroups"
                                            v-model="product.specials[props.index].user_group_id"
                                            class="input input--inForm"
                                            :reduce="userGroup => userGroup.id"
                                            label="name">
                                    </v-select>
                                </template>
                                <template v-else-if="props.column.field === 'price'">
                                    <div class="flex flex--align-center">
                                        <input type="number" class="input input--price"
                                               v-model.number="product.specials[props.index].price">
                                    </div>
                                </template>
                                <template v-else-if="props.column.field === 'date_start'">
                                    <div class="flex flex--align-center">
                                        <datetime input-class="input input--date"
                                                  v-model="product.specials[props.index].date_start_js"
                                                  type="datetime"
                                                  :phrases="datetime.phrases"
                                                  format="dd.MM.yyyy HH:mm"/>
                                    </div>
                                </template>
                                <template v-else-if="props.column.field === 'date_end'">
                                    <div class="flex flex--align-center">
                                        <datetime input-class="input input--date"
                                                  v-model="product.specials[props.index].date_end_js"
                                                  type="datetime"
                                                  :phrases="datetime.phrases"
                                                  format="dd.MM.yyyy HH:mm"/>
                                    </div>
                                </template>
                                <template v-else-if="props.column.field === 'action'">
                                    <a v-tooltip.top-start="'You have new messages.'" class="table__action"
                                       href="javascript:void(0)" @click.stop="deleteProductParameters('specials',props.index)">
                                        <icon icon="delete" class="icon"></icon>
                                    </a>
                                </template>
                            </template>
                        </vue-good-table>
                    </div>
                </template>

                <template v-else-if="tab === 'discounts'">
                    <div class="listData listData--noBg" v-if="product.discounts.length">
                        <vue-good-table
                                :columns="columnsForDiscounts"
                                :rows="product.discounts"
                                styleClass="table"
                                row-style-class="table__row">
                            <template slot="table-row" slot-scope="props">
                                <template v-if="props.column.field === 'user_group_id'">
                                    <v-select
                                            :clearable="false"
                                            :searchable="true"
                                            :options="userGroups"
                                            v-model="product.discounts[props.index].user_group_id"
                                            class="input input--inForm"
                                            :reduce="userGroup => userGroup.id"
                                            label="name">
                                    </v-select>
                                </template>
                                <template v-else-if="props.column.field === 'price'">
                                    <div class="flex flex--align-center">
                                        <input type="number" class="input input--price"
                                               v-model.number="product.discounts[props.index].price">
                                    </div>
                                </template>
                                <template v-else-if="props.column.field === 'quantity'">
                                    <div class="flex flex--align-center">
                                        <input type="number" class="input input--qty"
                                               v-model.number="product.discounts[props.index].quantity">
                                    </div>
                                </template>
                                <template v-else-if="props.column.field === 'date_start'">
                                    <div class="flex flex--align-center">
                                        <datetime input-class="input input--date"
                                                  v-model="product.discounts[props.index].date_start_js"
                                                  type="datetime"
                                                  :phrases="datetime.phrases"
                                                  format="dd.MM.yyyy HH:mm"/>
                                    </div>
                                </template>
                                <template v-else-if="props.column.field === 'date_end'">
                                    <div class="flex flex--align-center">
                                        <datetime input-class="input input--date"
                                                  v-model="product.discounts[props.index].date_end_js"
                                                  type="datetime"
                                                  :phrases="datetime.phrases"
                                                  format="dd.MM.yyyy HH:mm"/>
                                    </div>
                                </template>
                                <template v-else-if="props.column.field === 'action'">
                                    <a v-tooltip.top-start="'You have new messages.'" class="table__action"
                                       href="javascript:void(0)" @click.stop="deleteProductParameters('discounts',props.index)">
                                        <icon icon="delete" class="icon"></icon>
                                    </a>
                                </template>
                            </template>
                        </vue-good-table>
                    </div>
                </template>

                <template v-else-if="tab === 'seo'">

                    <div class="singleFormGroup">
                        <div class="singleFormGroup__title">
                            {{$root.translate.columns.slug}}:
                            <icon icon="icon" class="icon singleFormGroup__title_helper"></icon>
                        </div>
                        <div class="singleFormGroup__field">
                            <div class="flex flex--align-center">
                                <input type="text" class="input input--inForm"
                                       v-model="product.slug">
                            </div>
                        </div>
                    </div>
                    <div class="singleFormGroup">
                        <div class="singleFormGroup__title">
                            {{$root.translate.columns.title}} H1:
                        </div>
                        <div class="singleFormGroup__field inputWithTranslates" v-for="translate in product.translates">
                            <div class="flex flex--align-center">
                                {{$root.languages.find((language) => {return language.locale ===
                                translate.locale}).name}}:
                                <input type="text" class="input input--inForm input--label_left"
                                       v-model="translate.meta_h1">
                            </div>
                        </div>
                    </div>
                    <div class="singleFormGroup">
                        <div class="singleFormGroup__title">
                            {{$t("marketing_seo.seo_meta_title")}}:
                        </div>
                        <div class="singleFormGroup__field inputWithTranslates" v-for="translate in product.translates">
                            <div class="flex">
                                {{$root.languages.find((language) => {return language.locale ===
                                translate.locale}).name}}:
                                <textarea rows="3" class="input input--meta-tegs input--inForm input--label_left"
                                v-model="translate.meta_title"></textarea>
                            </div>
                        </div>
                    </div>
                    <div class="singleFormGroup">
                        <div class="singleFormGroup__title">
                            {{$t("marketing_seo.seo_meta_description")}}:
                        </div>
                        <div class="singleFormGroup__field inputWithTranslates" v-for="translate in product.translates">
                            <div class="flex">
                                {{$root.languages.find((language) => {return language.locale ===
                                translate.locale}).name}}:
                                <textarea rows="3" class="input input--meta-tegs input--inForm input--label_left"
                                v-model="translate.meta_description"></textarea>

                            </div>
                        </div>
                    </div>
                    <div class="singleFormGroup">
                        <div class="singleFormGroup__title">
                            {{$t("marketing_seo.seo_meta_keywords")}}:
                        </div>
                        <div class="singleFormGroup__field inputWithTranslates" v-for="translate in product.translates">
                            <div class="flex">
                                {{$root.languages.find((language) => {return language.locale ===
                                translate.locale}).name}}:
                                <textarea rows="3" class="input input--meta-tegs input--inForm input--label_left"
                                v-model="translate.meta_keywords"></textarea>

                            </div>
                        </div>
                    </div>
                </template>

            </div>
        </div>

        <widget-actions
                v-if="!blockedScreen && (tab !== 'variants' || (product.type === 2 && !product.variants.length && !$root.popups['variants-configuration'] && !$root.popups['variants-filling']))"
                :trans="{add: getTransForAddAction}"
                :add="getActionForTab"
                :store="isChangedInfo ? 'updateProduct': false"
                remove="deleteProduct"
                :foreign="product.href"
                :additional-styles="['add']"
        ></widget-actions>

    </div>
</template>

<script>

    let UploadThumb = require('./UploadThumbComponent').default;

    let ProductsVariationsComponent = require('./ProductsVariationsComponent').default;

    import draggable from 'vuedraggable'

    import VueCkeditor from 'vue-ckeditor2';

    export default {
        name: "ProductComponent",
        components: {
            VueCkeditor,
            UploadThumb,
            ProductsVariationsComponent,
            draggable,
        },
        data() {
            return {
                tab: 'main',
                blockedScreen: false,
                datetime: {
                    phrases: {ok: this.$root.translateWords('Apply'), cancel: this.$root.translateWords('Reset')},
                },
                activeTab: 0,
                refreshing: false,
                drag: false,
                Deselect: {
                    render: createElement => createElement('icon', {
                        class: 'icon',
                        props: {
                            icon: 'error'
                        }
                    }),
                },
                DeselectInProducts: {
                    render: createElement => createElement('icon', {
                        class: 'icon',
                        props: {
                            icon: 'delete'
                        }
                    }),
                },
                OpenIndicator: {
                    render: createElement => createElement('span', ''),
                },
                product: {},
                categories: [],
                products: [],
                currencies: [],
                stockStatus: {},
                stockStatuses: [],
                priceUnits: [],
                attributes: [],
                attribute_values: [],
                userGroups: [],
                informations: [],
                types: [
                    {
                        id: 1,
                        title: this.$root.translateWords('Simple product')
                    },
                    {
                        id: 2,
                        title: this.$root.translateWords('With variants')
                    }
                ],
                new_attribute_to_list: {},
                savedOriginal: {},
                columnsForSpecials: [
                    {
                        label: this.$root.translate.columns['user-group'],
                        field: 'user_group_id',
                        thClass: 'table__heading',
                        tdClass: 'table__value',
                        sortable: false,
                    },
                    {
                        label: this.$root.translate.columns.price,
                        field: 'price',
                        thClass: 'table__heading',
                        tdClass: 'table__value',
                        sortable: false,
                    },
                    {
                        label: this.$root.translate.columns['date-start'],
                        field: 'date_start',
                        thClass: 'table__heading',
                        tdClass: 'table__value',
                        sortable: false,
                    },
                    {
                        label: this.$root.translate.columns['date-end'],
                        field: 'date_end',
                        thClass: 'table__heading',
                        tdClass: 'table__value',
                        sortable: false,
                    },
                    {
                        label: this.$root.translate.columns.action,
                        field: 'action',
                        thClass: 'table__heading',
                        tdClass: 'table__value',
                        sortable: false,
                    },
                ],
                columnsForDiscounts: [
                    {
                        label: this.$root.translate.columns['user-group'],
                        field: 'user_group_id',
                        thClass: 'table__heading',
                        tdClass: 'table__value',
                        sortable: false,
                    },
                    {
                        label: this.$root.translate.columns.quantity,
                        field: 'quantity',
                        thClass: 'table__heading',
                        tdClass: 'table__value',
                        sortable: false,
                    },
                    {
                        label: this.$root.translate.columns.price,
                        field: 'price',
                        thClass: 'table__heading',
                        tdClass: 'table__value',
                        sortable: false,
                    },
                    {
                        label: this.$root.translate.columns['date-start'],
                        field: 'date_start',
                        thClass: 'table__heading',
                        tdClass: 'table__value',
                        sortable: false,
                    },
                    {
                        label: this.$root.translate.columns['date-end'],
                        field: 'date_end',
                        thClass: 'table__heading',
                        tdClass: 'table__value',
                        sortable: false,
                    },
                    {
                        label: this.$root.translate.columns.action,
                        field: 'action',
                        thClass: 'table__heading',
                        tdClass: 'table__value',
                        sortable: false,
                    },
                ],
                actionProduct:{}
            }
        },
        created() {

            this.getProduct();
            this.onSearchCategories();
            this.onSearchProducts();
            this.getCurrencies();
            this.getStockStatuses();
            this.getAttributes();
            this.getAttributesValues();
            this.getUserGroups();
            this.getPriceUnits();
            this.getInformations();

            let self = this;

            this.debouncedStoreProduct = _.debounce(function () {
                self.updateProduct();
            }, 1000 * 60 * 2);

        },
        computed: {
            isEditProduct(){
                if (this.$route.params !== undefined && this.$route.params) {
                    return this.$route.params;
                }
            },
            isChangedInfo() {

                let current = this.$root.copy(this.product);

                if (Object.keys(current).length) {

                    delete current.refreshing;

                    current.translates.forEach(translate => translate.description = translate.description.trim());
                    current.variants.forEach(variant => {
                        variant.translates.forEach(translate => translate.description = translate.description.trim())
                    });


                    let saved = this.$root.copy(this.savedOriginal);

                    saved.translates.forEach(translate => translate.description = translate.description.trim());
                    saved.variants.forEach(variant => {
                        variant.translates.forEach(translate => translate.description = translate.description.trim())
                    });

                    delete saved.refreshing;

                    return JSON.stringify(current) !== JSON.stringify(saved);
                }

            },
            dragOptions() {
                return {
                    animation: 200,
                    group: "description",
                    disabled: false,
                    ghostClass: "ghost"
                };
            },
            languageLinks() {
                let languages = [];

                for (let language of this.$root.languages) {
                    languages.push({
                        text: language.name,
                        locale: language.locale
                    });
                }

                return languages;
            },
            getActionForTab() {

                let action;

                switch (this.tab) {
                    case 'attributes':
                        action = 'addToAttribute';
                        break;
                    case 'specials':
                        action = 'addSpecial';
                        break;
                    case 'discounts':
                        action = 'addDiscount';
                        break;
                    case 'variants':

                        if (!this.product.variants.length) {
                            action = 'showConfiguration';
                        }

                        break;
                }

                return action || null;
            },
            getTransForAddAction() {
                let trans = '';

                switch (this.tab) {
                    case 'attributes':
                        trans = this.$root.translateWords('Add attribute');
                        break;
                    case 'specials':
                        trans = this.$root.translateWords('Create special');
                        break;
                    case 'discounts':
                        trans = this.$root.translateWords('Create wholesale price');
                        break;
                    case 'variants':

                        if (!this.product.variants.length) {
                            trans = this.$root.translateWords('Make variants');
                        }

                        break;


                }

                return trans;
            }
        },
        watch: {
            isChangedInfo: function (value) {
                if (value) {
                    //this.debouncedStoreProduct();
                }
            },
            isEditProduct: function (value) {
                if (value.id) {
                    this.getProduct();
                }
                document.getElementById('perfect-scroll-offset').scrollTop = 0;
            },
        },
        methods: {
            clearAttributeSearchPhrase(id) {

                let to_attribute_index = this.product.to_attributes.findIndex(to_attribute => to_attribute.id === id);

                this.$set(this.product.to_attributes[to_attribute_index], 'attributeSearchPhrase', '');

                this.$set(this.product.to_attributes[to_attribute_index], 'values', []);

            },
            clearAttributeValueSearchPhrase(id) {

                let to_attribute_index = this.product.to_attributes.findIndex(to_attribute => to_attribute.id === id);

                this.$set(this.product.to_attributes[to_attribute_index], 'attributeValueSearchPhrase', '');

            },
            updateProductOriginal() {
                this.$set(this, 'savedOriginal', this.$root.copy(this.product));
            },
            checkProductTypeForVariants() {

                this.blockedScreen = true;

                this.$dialog.confirm({
                    title: this.$root.translateWords('Are you sure?'),
                    body: this.$root.translateWords('This action cannot be undone')
                }, {
                    view: 'confirm-window', // can be set globally too
                }).then(() => {
                    this.tab = 'variants';
                    this.product.type = 2;

                    this.addToAttribute(true);
                    this.$refs.productVariants.showConfiguration();
                    this.blockedScreen = false;
                }).catch(() => this.blockedScreen = false);
            },
            setAttributeSearchPhrase(id) {

                let to_attribute_index = this.product.to_attributes.findIndex(to_attribute => to_attribute.id === id);

                this.$set(this.product.to_attributes[to_attribute_index], 'attributeSearchPhrase', event.target.value);
            },
            checkAttributeExist(id) {

                let result = true;

                let to_attribute_index = this.product.to_attributes.findIndex(to_attribute => to_attribute.id === id);

                let phrase = this.product.to_attributes[to_attribute_index].attributeSearchPhrase || '';

                if ((phrase && phrase.length) || (this.product.to_attributes[to_attribute_index].attribute !== null && !this.product.to_attributes[to_attribute_index].attribute.id)) {
                    if (!this.attributes.filter(attribute => {
                        return attribute.name === phrase;
                    }).length) {
                        result = false;
                    }
                }

                return result;
            },
            setAttributeValueSearchPhrase(id) {

                let to_attribute_index = this.product.to_attributes.findIndex(to_attribute => to_attribute.id === id);

                this.$set(this.product.to_attributes[to_attribute_index], 'attributeValueSearchPhrase', event.target.value);

            },
            checkAttributeValueExist(id) {

                let result = true;

                let to_attribute_index = this.product.to_attributes.findIndex(to_attribute => to_attribute.id === id);

                let to_attribute = this.product.to_attributes[to_attribute_index];

                if (to_attribute.attribute !== null) {

                    let phrase = to_attribute.attributeValueSearchPhrase || '';

                    let values = this.product.to_attributes[to_attribute_index].values;

                    if ((phrase && phrase.length) || (values.length && values[values.length - 1] !== null && !values[values.length - 1].id)) {
                        if (!this.attribute_values.filter(attribute_value => {
                            return (attribute_value.attribute_id === to_attribute.attribute.id && attribute_value.value === phrase);
                        }).length) {
                            result = false;
                        }
                    }
                }

                return result;
            },
            addToAttribute(has_variant = false) {
                this.product.to_attributes.push({
                    id: 'tmp-' + Math.random(100000, 10000000),
                    attribute: null,
                    values: [],
                    main: false,
                    has_variant: has_variant
                });
            },
            createAttributeValue(id, ref, inVariant = true) {

                let translates = [];

                let to_attribute_index = this.product.to_attributes.findIndex(to_attribute => to_attribute.id === id);

                let name = this.product.to_attributes[to_attribute_index].attributeValueSearchPhrase.length ? this.product.to_attributes[to_attribute_index].attributeValueSearchPhrase : this.product.to_attributes[to_attribute_index].values[this.product.to_attributes[to_attribute_index].values.length - 1].value;

                this.$root.languages.forEach(language => {
                    translates.push({
                        locale: language.locale,
                        value: name
                    });
                });

                this.refreshing = true;

                axios.post('/admin/attribute-values', {
                    attribute_id: this.product.to_attributes[to_attribute_index].attribute.id,
                    slug: null,
                    sort_order: 0,
                    status: true,
                    translates: translates
                }).then(httpResponse => {

                    this.attribute_values.push({
                        id: httpResponse.data.id,
                        value: name,
                        attribute_id: this.product.to_attributes[to_attribute_index].attribute.id
                    });

                    this.product.to_attributes[to_attribute_index].values.push({
                        id: httpResponse.data.id,
                        value: name,
                        attribute_id: this.product.to_attributes[to_attribute_index].attribute.id
                    });

                    if (inVariant) {
                        this.$refs.productVariants.$refs[ref][0].search = '';
                    } else {
                        this.$refs[ref][0].search = '';
                    }

                    this.$root.notify(httpResponse.data);

                }).catch(error => {
                    if (error.response) this.$root.notify(error.response.data);
                }).finally(() => {
                    this.refreshing = false;
                });
            },
            createAttribute(id) {

                let to_attribute_index = this.product.to_attributes.findIndex(to_attribute => to_attribute.id === id);

                let name = this.product.to_attributes[to_attribute_index].attributeSearchPhrase.length ? this.product.to_attributes[to_attribute_index].attributeSearchPhrase : this.product.to_attributes[to_attribute_index].attribute.name;

                let translates = [];

                this.$root.languages.forEach(language => {
                    translates.push({
                        locale: language.locale,
                        name: name
                    });
                });

                this.refreshing = true;

                axios.post('/admin/attributes', {
                    status: true,
                    sort_order: null,
                    translates: translates
                }).then(httpResponse => {

                    this.attributes.push({
                        id: httpResponse.data.id,
                        name: name
                    });

                    this.$set(this.product.to_attributes[to_attribute_index], 'attribute', {
                        id: httpResponse.data.id,
                        name: name
                    });

                    this.$set(this.product.to_attributes[to_attribute_index], 'values', []);

                    this.$root.notify(httpResponse.data);

                }).catch(error => {
                    if (error.response) this.$root.notify(error.response.data);
                }).finally(() => {
                    this.refreshing = false;
                });
            },
            removeToAttribute(id) {
                this.$dialog.confirm({
                    title: this.$root.translateWords('Are you sure?'),
                    body: this.$root.translateWords('This action cannot be undone')
                }, {
                    view: 'confirm-window', // can be set globally too
                }).then(() => {
                    this.product.to_attributes.splice(this.product.to_attributes.findIndex(to_attribute => to_attribute.id === id), 1);
                });
            },
            onSearchProducts(phrase = null, loading = null) {
                if (loading !== null) loading(true);
                this.getProducts(phrase, this, loading);
            },
            changeCategoriesList(values) {
                if (this.product.category_id !== null) {
                    let index = values.findIndex(value => {
                        return value.id === this.product.category_id;
                    });

                    if (index === -1) this.product.category_id = null;
                }
            },
            onSearchCategories(phrase = null, loading = null) {
                if (loading !== null) loading(true);
                this.getCategories(phrase, this, loading);
            },
            updateProduct() {

                this.$root.waitingServerActionInfo(true, this.$t("words.Saveing changes in product"));

                this.refreshing = true;

                if (this.product.variants.length && this.tab === 'variants') this.$refs.productVariants.refreshing = true;

                if (this.product.type === 2 && !this.product.variants.length) {
                    this.deleteProduct();
                } else {
                    axios.put('/admin/products/' + this.product.id, this.product).then(
                        httpResponse => {

                            this.$root.notify(httpResponse.data);

                            let i;

                            for (i in this.product.variants) {
                                this.$set(this.product.variants[i], 'id', httpResponse.data.variants_info[i].id);
                                this.$set(this.product.variants[i], 'href', httpResponse.data.variants_info[i].href);
                            }

                            this.product.primary_variant_id = httpResponse.data.primary_variant_id;
                            this.product.href = httpResponse.data.href;
                            this.product.visual_labels = httpResponse.data.visual_labels;
                            this.product.date_available_day = httpResponse.data.date_available_day;

                            this.$set(this, 'savedOriginal', this.$root.copy(this.product));

                            if (this.product.variants.length) {
                                this.product.variants.forEach(elem => {
                                    if (elem.id === this.product.primary_variant_id) {
                                        alert('warians');
                                        this.actionProduct.type = "save";
                                        this.actionProduct.name = elem.translates;
                                        this.actionProduct.price = elem.vendor_price;
                                        this.actionProduct.quantity = elem.quantity;
                                        this.actionProduct.id = elem.id;
                                        this.actionProduct.filemanager_thumb = elem.images[0].filemanager_thumb;
                                        this.actionProduct.image = elem.images[0].src;
                                        this.actionProduct.visual_labels = elem.visual_labels;
                                        this.actionProduct.date_available_day = elem.date_available_day;
                                        this.actionProduct.href = elem.href;
                                        this.actionProduct.sku = elem.sku;
                                        this.actionProduct.main_id = elem.main_id;
                                        this.actionProduct.quantity_other_variants = this.product.variants.length;
                                    }
                                })
                            }else{
                                this.actionProduct.type = "save";
                                this.actionProduct.name = this.product.translates;
                                this.actionProduct.price = this.product.vendor_price;
                                this.actionProduct.quantity = this.product.quantity;
                                this.actionProduct.id = this.product.id;
                                this.actionProduct.sku = this.product.sku;
                                this.actionProduct.href = this.product.href;
                                this.actionProduct.visual_labels = this.product.visual_labels;
                                this.actionProduct.date_available_day = this.product.date_available_day;
                                this.actionProduct.filemanager_thumb = null;
                                this.actionProduct.image = null;
                                if (this.product.images[0].length !== 0) {
                                    this.actionProduct.filemanager_thumb = this.product.images[0].filemanager_thumb;
                                    this.actionProduct.image = this.product.images[0].src;
                                }
                            }
                        }).catch(error => {
                        if (error.response) this.$root.notify(error.response.data);
                    }).finally(() => {
                        this.refreshing = false;
                        this.$root.waitingServerActionInfo(false);
                        if (this.product.variants.length && this.tab === 'variants') this.$refs.productVariants.refreshing = false;
                    });
                }
            },
            getProduct() {
                this.$root.waitingServerActionInfo(true, this.$t("words.receiving data"));
                axios.get('/admin/products/' + this.$route.params.id + '/edit')
                    .then(httpResponse => {
                        this.product = httpResponse.data.product;

                        this.product.to_attributes.forEach(function (to_attribute) {
                            to_attribute.values.forEach(function (value) {
                                delete value.laravel_through_key;
                            });
                        });

                        let images = [];

                        if (this.product.image) {
                            images.push({
                                src: this.product.image,
                                filemanager_thumb: this.product.filemanager_thumb
                            });
                        }

                        this.product.images.forEach(image => {
                            images.push({
                                src: image.src,
                                filemanager_thumb: image.filemanager_thumb
                            });
                        });

                        this.$set(this.product, 'images', images);

                        this.product.variants.forEach(variant => {
                            let images = [];

                            images.push({
                                src: variant.image,
                                filemanager_thumb: variant.filemanager_thumb
                            });

                            variant.images.forEach(image => {
                                images.push({
                                    src: image.src,
                                    filemanager_thumb: image.filemanager_thumb
                                });
                            });

                            this.$set(variant, 'images', images);
                        });

                        this.savedOriginal = this.$root.copy(this.product);

                    }).catch(error => {
                    if (error.response) this.$root.notify(error.response.data);

                    }).finally(() => {
                        this.$root.waitingServerActionInfo(false);
                    });

            },
            getCategories: _.debounce((phrase, self, loading) => {
                axios.get('/admin/categories', {
                    params: {
                        phrase: phrase,
                        autocomplete: true
                    }
                }).then(httpResponse => {
                    self.$set(self, 'categories', httpResponse.data.categories);
                    if (loading !== null) loading(false);
                });
            }, 500),
            getProducts: _.debounce((phrase, self, loading) => {
                axios.get('/admin/products', {
                    params: {
                        phrase: phrase,
                        autocomplete: true,
                        exclude: self.$route.params.id
                    }
                }).then(
                    httpResponse => {
                        self.$set(self, 'products', httpResponse.data);
                        if (loading !== null) loading(false);
                    }
                )
            }),
            getCurrencies() {
                axios.get('/admin/currencies', {
                    params: {
                        autocomplete: true
                    }
                }).then(httpResponse => {
                    this.currencies = httpResponse.data.currencies;
                });
            },
            getStockStatuses() {

                let self = this;

                axios.get('/admin/stock-statuses/', {
                    params: {
                        autocomplete: true
                    }
                }).then(
                    httpResponse => {
                        this.stockStatuses = httpResponse.data.stock_statuses;

                        if (this.product.stock_status_id) {
                            this.stockStatus = this.stockStatuses.find(stock_status => {
                                return stock_status.id === self.product.stock_status_id;
                            }) || {};
                        }
                    });
            },
            getPriceUnits() {
                axios.get('/admin/price-units/', {
                    params: {
                        autocomplete: true
                    }
                }).then(
                    httpResponse => {
                        this.priceUnits = httpResponse.data.price_units;
                    });
            },
            getAttributes() {
                axios.get('/admin/attributes/', {
                    params: {
                        with_translate: true
                    }
                }).then(
                    httpResponse => {
                        this.attributes = httpResponse.data.attributes;
                    });
            },
            getAttributesValues() {
                axios.get('/admin/attribute-values/', {
                    params: {
                        autocomplete: true,
                    }
                }).then(httpResponse => {
                    this.attribute_values = httpResponse.data.values;
                })
            },
            getUserGroups() {
                axios.get('/admin/user-groups/', {
                    params: {
                        autocomplete: true
                    }
                }).then(httpResponse => {
                    this.$set(this, 'userGroups', httpResponse.data.user_groups);
                });
            },
            getInformations() {
                axios.get('/admin/informations/', {
                    params: {
                        autocomplete: true,
                    }
                }).then(httpResponse => {
                    this.informations = httpResponse.data.informations;
                })
            },
            addSpecial() {
                this.product.specials.push({
                    user_group_id: null,
                    price: null,
                    date_start: null,
                    date_end: null
                });
            },
            deleteSpecial(index) {
                this.product.specials.splice(index, 1);
            },
            addDiscount() {
                this.product.discounts.push({
                    user_group_id: null,
                    quantity: null,
                    price: null,
                    date_start: null,
                    date_end: null
                });
            },
            deleteDiscount(index) {
                this.product.discounts.splice(index, 1);
            },
            openUploadWindow() {
                let self = this;
                var route_prefix = '/admin/filemanager';

                window.open(route_prefix + '?items_type=products&item=' + btoa('products-' + this.product.id) + '&type=' + (self.type ? self.type : 'image') + 'FileManager', 'width=900,height=600');

                window.SetUrl = function (info) {

                    info.forEach(image => {
                        self.product.images.push({
                            'src': image.name,
                            'filemanager_thumb': image.thumb_url
                        });
                    });
                };
            },
            deleteProduct() {
                this.$dialog.confirm({
                    title: this.$root.translateWords('Are you sure?'),
                    body: this.$root.translateWords('This action cannot be undone')
                }, {
                    view: 'confirm-window', // can be set globally too
                }).then(() => {
                    this.$root.waitingServerActionInfo(true, this.$t("words.deleted product"));

                    this.refreshing = true;

                    if (this.product.variants.length && this.tab === 'variants') this.$refs.productVariants.refreshing = true;

                    axios.delete('/admin/products/' + this.product.id).then(() => {
                        this.$set(this, "actionProduct", {   type: "delete",
                                                             id: this.product.id
                        });
                        this.$router.push({'name': 'products'})
                    }).catch(() => {
                        this.refreshing = false;

                        if (this.product.variants.length && this.tab === 'variants') this.$refs.productVariants.refreshing = false;
                    }).finally(() => {
                        this.refreshing = false;
                        this.$root.waitingServerActionInfo(false);
                    });
                });
            },
            addVariant() {
                this.$refs.productVariants.addVariant();
            },
            showConfiguration() {
                this.$refs.productVariants.showConfiguration();
            },
            deleteProductParameters(type, index){
                this.$dialog.confirm({
                    title: this.$root.translateWords('Are you sure?'),
                    body: this.$root.translateWords('This action cannot be undone')
                }, {
                    view: 'confirm-window', // can be set globally too
                }).then(() => {
                    this.product[type].splice(index, 1);
                });
            }
        },
        beforeRouteLeave (to, from, next) {
            if (this.actionProduct.data) {
                this.$set(this.actionProduct.data, 'refreshing', false);
            }
            to.params.product = this.actionProduct;
            to.params.scrollActivate = true;
            next()
        },
    }
</script>

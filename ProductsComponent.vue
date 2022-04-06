<template>
    <div class="table-list-container">

        <!-- <h2 class="mainContent__heading">{{$root.translate.menu.catalog.items.products}}</h2> -->

        <div class="filter">
            <div class="filter__name">
                <input @input="debounceFunction(onColumnFilter)" type="text" class="input"
                       v-model="serverParams.name_sku"
                       :placeholder="$root.translateWords('Search products by name or sku')">
                <icon icon="search" class="icon filter__name_icon"></icon>
            </div>
            <advanced-filter :criteria-name="['Categories', 'Price', 'Quantity', 'Status', 'Attributes']"
                             :settings="{offset_top: true}" @filter="expandFilter"
                             v-on:clear-filter-option="clearFilterParam"></advanced-filter>
        </div>

        <template v-if="isLoaded">
            <div class="listData" v-if="rows.length || totalRecords">

                <mass-actions
                        @clear-checked-items="clearCheckedItems"
                        :refresh-items="loadItems"
                        :products="selecteds"
                ></mass-actions>

                <vue-good-table
                        @on-page-change="onPageChange"
                        @on-sort-change="onSortChange"
                        @on-column-filter="onColumnFilter"
                        @on-per-page-change="onPerPageChange"
                        :pagination-options="{
                            enabled: false
                        }"
                        mode="remote"
                        :columns="columns"
                        :rows="rows"
                        :totalRows="totalRecords"
                        :isLoading.sync="isLoading"
                        styleClass="table"
                        :row-style-class="getValueRowClass">


                    <template slot="table-column" slot-scope="props">
                        <template v-if="props.column.label == 'check'">
                            <check :checked="selectedAll" @click.native="changeSelectAll"></check>
                        </template>
                        <span v-else>
                        {{props.column.label}}
                    </span>
                    </template>

                    <template slot="table-row" slot-scope="props">
                        <template v-if="props.column.field === 'check' && rows[props.index].type !== 3">
                            <check :checked="selecteds.indexOf(rows[props.index].id) !== -1"
                                   @click.native="changeSelect(rows[props.index].id)"></check>
                        </template>
                        <template v-else-if="props.column.field === 'sku'">
                            <div class="flex flex--align-center">
                                <input type="text" class="input input--price"
                                       v-model="rows[props.index].sku">
                            </div>
                            <router-link v-if="parseInt(rows[props.index].variants_count) > 1"
                                         :to="{name:'product', params: {id: rows[props.index].type === 2 ? rows[props.index].main_id : rows[props.index].id}}"
                                         class="text-link">
                                {{$root.translateWords('Has variants:') + ' ' +
                                (parseInt(rows[props.index].variants_count) - 1)}}
                            </router-link>
                        </template>
                        <template v-else-if="props.column.field === 'image'">

                            <upload-thumb
                                    :is_tmp="typeof rows[props.index].id !== 'number'"
                                    items_type="products"
                                    :item="$root.encodeId('products', rows[props.index].type === 2 ? rows[props.index].main_id : rows[props.index].id)"
                                    :data="rows[props.index]"
                                    :file_path="rows[props.index].image"
                                    :thumb_path="rows[props.index].filemanager_thumb"
                                    img-style="max-width: 57px; border-radius: 5px;"
                                    @remove="rows[props.index].image = ''"
                            ></upload-thumb>

                        </template>
                        <template v-else-if="props.column.field === 'name'">
                            <div v-for="translate in rows[props.index].translates" class="inputWithTranslates">
                                <div class="flex flex--align-center">
                                    {{$root.languages.find((language) => {return language.locale ===
                                    translate.locale}).name}}:
                                    <input type="text" class="input input--label_left"
                                           v-model="translate.name">
                                </div>
                            </div>
                        </template>
                        <template v-else-if="props.column.field == 'vendor_price'">
                            <div class="flex flex--align-center">
                                <input type="number" class="input input--price input--label_right"
                                       v-model.number="rows[props.index].vendor_price">
                                {{rows[props.index].currency_code}}
                            </div>
                        </template>
                        <template v-else-if="props.column.field == 'quantity'">

                            <div class="flex flex--align-center">
                                <input type="number" class="input input--qty"
                                       v-model.number="rows[props.index].quantity">
                            </div>
                        </template>
                        <template v-else-if="props.column.field === 'actions'">
                            <template v-if="rows[props.index].refreshing">
                                <font-awesome-icon class="text-warning" icon="circle-notch" spin></font-awesome-icon>
                            </template>
                            <template v-else>
                                <a v-if="isChangedRow(rows[props.index].id)" class="table__action"
                                   href="javascript:void(0)" @click.stop="storeProduct(props.index)">
                                    <icon icon="floppy-disk" class="icon"></icon>
                                </a>
                                <template v-if="typeof rows[props.index].id === 'number'">
                                    <a class="table__action" :href="rows[props.index].href" target="_blank">
                                        <icon icon="foreign" class="icon"></icon>
                                    </a>
                                    <a class="table__action"
                                       href="javascript:void(0)" @click.stop="openEditProduct(rows[props.index])">
                                        <icon icon="pencil-edit-button" class="icon"></icon>
                                    </a>
                                </template>
                                <a v-tooltip.top-start="'You have new messages.'" class="table__action"
                                   href="javascript:void(0)" @click.stop="removeProduct(props.index)">
                                    <icon icon="delete" class="icon"></icon>
                                </a>
                            </template>
                        </template>
                        <span v-else>
                        {{props.formattedRow[props.column.field]}}
                    </span>
                    </template>

                    <template slot="pagination-bottom" :slot-scope="props">
                    </template>

                </vue-good-table>

                <pagination
                        v-if="serverParams.perPage < totalRecords+1"
                        :current-page="serverParams.page"
                        :per-page="serverParams.perPage"
                        :total="totalRecords"
                        :from-records="serverParams.fromRecords"
                        :to-records="serverParams.toRecords"
                        :pageChanged="onPageChange"
                        :perPageChanged="onPerPageChange">
                </pagination>
            </div>
            <div v-else class="listEmpty">
                <div class="listEmpty__heading">{{$root.translateWords('Your products list is empty')}} :(</div>
                <div class="listEmpty__text">
                    {{$root.translateWords('You may add them')}}
                    <a class="listEmpty__link" href="javascript:void(0)" @click.stop="addProduct">{{$root.translateWords('manually')}}</a>
                    {{$root.translate.columns.or}}
                    <router-link class="listEmpty__link" :to="{name: 'xml-import'}">{{$root.translateWords('use import')}}
                    </router-link>
                </div>
            </div>
        </template>

        <!-- <widget-actions v-if="selecteds.length || !modalConfigurationProductsComponent.open" add="addProduct"
                        remove="removeProducts" copy="copyProducts"
                        :trans="{add: $root.translateWords('Create product')}"></widget-actions> -->
        <widget-actions add="addProduct" :trans="{add: $root.translateWords('Create product')}"></widget-actions>

    </div>
</template>

<script>
    //$root.translateWords(modalConfigurationProductsComponent.edit ? 'Editing configuration': 'Creating configuration')
    let UploadThumb = require('./UploadThumbComponent').default;

    export default {
        name: "ProductsComponent",
        components: {
            UploadThumb,
            'mass-actions': require('./Product/MassActions').default,
            'pagination': require('./paginationComponent').default,
            'advanced-filter': require('./FilterComponent').default
        },
        data() {
            return {
                scrollOffset: 0,
                totalRecords: 0,
                serverParams: {
                    columnFilters: {},
                    sort_column: null,
                    sort_direction: null,
                    page: 1,
                    perPage: 100,
                    fromRecords: null,
                    toRecords: null
                },
                isLoaded: false,
                columns: [
                    {
                        label: 'check',
                        field: 'check',
                        thClass: 'table__heading',
                        tdClass: 'table__value',
                        sortable: false,
                    },
                    {
                        label: this.$root.translate.columns.sku,
                        field: 'sku',
                        thClass: 'table__heading',
                        tdClass: 'table__value'
                    },
                    {
                        label: this.$root.translate.columns.image_short,
                        field: 'image',
                        thClass: 'table__heading',
                        tdClass: 'table__value',
                        sortable: false,
                    },
                    {
                        label: this.$root.translate.columns.name,
                        field: 'name',
                        thClass: 'table__heading',
                        tdClass: 'table__value',
                        sortable: false,
                    },
                    {
                        label: this.$root.translate.columns.price,
                        field: 'vendor_price',
                        thClass: 'table__heading',
                        tdClass: 'table__value'
                    },
                    {
                        label: this.$root.translate.columns.quantity,
                        field: 'quantity',
                        thClass: 'table__heading',
                        tdClass: 'table__value'
                    },
                    {
                        label: this.$root.translate.columns.actions,
                        field: 'actions',
                        thClass: 'table__heading table__heading--text_right',
                        tdClass: 'table__value table__value--text_right',
                        sortable: false,
                    },
                ],
                modalConfigurationProductsComponent: {
                    edit: false,
                    open: false
                },
                rows: [],
                isLoading: false,
                currencies: [],
                selecteds: [],
                refreshing: false,
                selectedAll: false,
                savedOriginal: [],
                modal_add_to_export_list: [],
                Deselect: {
                    render: createElement => createElement('icon', {
                        class: 'icon',
                        props: {
                            icon: 'error'
                        }
                    }),
                },
                OpenIndicator: {
                    render: createElement => createElement('span', ''),
                },
            }
        },
        computed: {
            countCheckedItems() {
                if (this.selecteds.length) {
                    return this.selecteds.length
                }
            },

            isSelected() {
                return (product_id) => {
                    return this.selecteds.indexOf(product_id) !== -1;
                }
            },
            isChangedRow() {
                return (product_id) => {
                    let originalPosition = this.savedOriginal.findIndex(product => {
                        return product.id === product_id
                    });

                    if (originalPosition === -1) {
                        return true;
                    }

                    let currentPosition = this.rows.findIndex(product => {
                        return product.id === product_id
                    });

                    let currentProduct = this.$root.copy(this.rows[currentPosition]);

                    delete currentProduct.refreshing;

                    let savedProduct = this.$root.copy(this.savedOriginal[originalPosition]);

                    delete savedProduct.refreshing;

                    return JSON.stringify(currentProduct) !== JSON.stringify(savedProduct);
                }
            },
            isProductsChanget() {
                if (this.$route.params !== undefined && this.$route.params) {
                    return this.$route.params;
                }
            },
        },
        created() {
            axios.get('/admin/currencies/').then(
                httpResponse => {
                    this.currencies = httpResponse.data.currencies;
                });

            let url = new URL(window.location.href);
            let page = parseInt(url.searchParams.get("page"));

            if (!page) page = 1;

            this.serverParams.page = page;

            history.pushState(null, null, '/admin/products/?page=' + page);

            this.loadItems();
        },
        methods: {
            addProduct() {
                let product = {
                    id: 'tmp-' + Math.random(100000, 10000000),
                    image: '',
                    vendor_price: 0,
                    quantity: 0,
                    sort_order: 0,
                    translates: [],
                    refreshing: false,
                    href: ''
                };

                for (let language of this.$root.languages) {
                    product.translates.push({
                        name: null,
                        locale: language.locale
                    });
                }

                this.$root.scrollToNewRow(this.rows, product);
            },
            storeProduct(index) {

                let product = this.rows[index];

                let product_id = product.id;

                let originalPosition = this.savedOriginal.findIndex(product => {
                    return product.id === product_id
                });

                let request;

                product.refreshing = true;

                if (typeof product.id === 'number') {

                    request = axios.put('/admin/products/' + product.id + '?fast=1', product);

                } else {
                    request = axios.post('/admin/products', product);
                }

                request.then(httpResponse => {

                    if (typeof product.id !== 'number') {

                        product.id = httpResponse.data.id;

                        product.sku = httpResponse.data.sku;

                        product.href = httpResponse.data.href;
                    }

                    if (originalPosition !== -1) {
                        this.$set(this.savedOriginal, originalPosition, this.$root.copy(product));
                    } else {
                        this.savedOriginal.push(this.$root.copy(product));
                    }

                    this.$root.notify(httpResponse.data);

                }).catch(error => {
                    if (error.response) this.$root.notify(error.response.data);
                }).finally(() => {
                    product.refreshing = false;
                });

            },
            removeProduct(index) {
                this.$dialog.confirm({
                    title: this.$root.translateWords('Are you sure?'),
                    body: this.$root.translateWords('This action cannot be undone')
                }, {
                    view: 'confirm-window', // can be set globally too
                }).then(() => {

                    let product = this.rows[index];

                    let product_id = product.id;

                    let originalPosition = this.savedOriginal.findIndex(product => {
                        return product.id === product_id
                    });

                    product.refreshing = true;

                    let idForRemove = product.type === 2 ? product.main_id : product.id;

                    if (typeof idForRemove === 'number') {

                        axios.delete('/admin/products/' + idForRemove).then(httpResponse => {

                            this.$root.notify(httpResponse.data);

                            this.rows.splice(index, 1);

                            this.savedOriginal.splice(originalPosition, 1);

                        });

                    } else {
                        this.rows.splice(index, 1);
                    }
                });
            },
            async loadItems(params = null) {
                this.$root.waitingServerActionInfo(true, this.$t("words.receiving data"));

                if (params !== null) {
                    this.serverParams.sort_column = params[0].field;
                    this.serverParams.sort_direction = params[0].type;
                }


                return axios.get('/admin/products', {
                    params: this.serverParams
                }).then(response => {

                    this.totalRecords = response.data.products.total;

                    if (response.data.products.data.length === 0 && this.totalRecords > 0) {
                        this.onPageChange({currentPage: 1});
                    } else {
                        this.serverParams.fromRecords = response.data.products.from;
                        this.serverParams.toRecords = response.data.products.to;

                        if (response.data.products.data.length) {
                            response.data.products.data.forEach(product => {
                                product.refreshing = false;
                            });
                        }

                        this.$set(this, 'rows', response.data.products.data);
                        this.$set(this, 'savedOriginal', this.$root.copy(response.data.products.data));
                        this.isLoaded = true;

                    }
                }).catch(error => {
                    if (error.response) this.$root.notify(error.response.data);
                    this.$root.waitingServerActionInfo(false);

                }).finally(() => {
                    this.$root.waitingServerActionInfo(false);
                });
            },
            changeSelectAll() {

                this.selectedAll = !this.selectedAll;

                this.$set(this, 'selecteds', []);

                if (this.selectedAll) {

                    for (let i in this.rows) {
                        this.selecteds.push(this.rows[i].id);
                    }
                }
            },
            changeSelect(product_id) {
                let pos = this.selecteds.indexOf(product_id);

                if (pos === -1) {
                    this.selecteds.push(product_id);
                } else {
                    this.selecteds.splice(pos, 1);
                }
            },
            getValueRowClass(row) {
                let rowClass = 'table__row';

                if (this.isSelected(row.id)) rowClass += ' table__row--selected';

                if (row.changed === true && row.changed !== undefined) rowClass += ' table__row--was-changet';

                return rowClass;

            },
            openMassEdit() {
                this.$refs.massEdit.open();
            },
            updateParams(newProps) {
                this.serverParams = Object.assign({}, this.serverParams, newProps);
            },

            onPageChange(params) {
                this.updateParams({page: params.currentPage});

                history.pushState(null, null, '/admin/products/?page=' + params.currentPage);

                this.loadItems();
            },

            onPerPageChange(params) {
                this.updateParams({perPage: params.currentPerPage});
                this.loadItems();
            },

            onSortChange(params) {
                this.serverParams.sort_direction = params[0].type;
                this.serverParams.sort_column = params[0].field;

                this.loadItems();
            },

            onColumnFilter(params) {
                this.updateParams(params);
                this.loadItems();
            },
            debounceFunction: _.debounce(function (userFunction) {
                userFunction();
            }, 1500),
            paramText(variant) {

                let names = [];

                variant.variant_attribute_values.forEach(function (value) {
                    names.push(value.value);
                });

                return names.join(' & ');
            },
            expandFilter(attributes = {}) {

                this.updateParams(attributes);

                this.loadItems();
            },
            clearFilterParam(param) {
                if (Object(this.serverParams).hasOwnProperty(param)) {
                    delete this.serverParams[param];
                }
            },
            clearCheckedItems() {
                this.selectedAll = false;
                this.selecteds = [];
            },
            getAllProductsId() {
                let self = this;
                axios.get('/admin/products/all-ids').then(Response => {
                    self.$set(self, 'selecteds', Response.data);
                    console.log(Response.data, "all id");
                })
            },
            openEditProduct(item) {
                this.scrollOffset = document.getElementById('perfect-scroll-offset').scrollTop;
                this.$router.push({name: 'product', params: {id: item.type === 2 ? item.main_id : item.id}});
            },
        },
        watch: {
            isProductsChanget(value){
                if (value !== undefined && value.product !== undefined) {
                    let index
                    let originalPosition
                    if (value.product.main_id !== undefined) {
                        index = this.rows.findIndex(product => {
                            return product.main_id === value.product.main_id;
                        });
                        originalPosition = this.savedOriginal.findIndex(product => {
                            return product.main_id === value.product.main_id;
                        });
                    }else{
                        index = this.rows.findIndex(product => {
                            return product.id === value.product.id;
                        });
                        originalPosition = this.savedOriginal.findIndex(product => {
                            return product.id === value.product.id;
                        });
                    }
                    if (value.product.type === "delete") {
                        this.rows.splice(index, 1);
                        this.savedOriginal.splice(originalPosition, 1);
                    }else if (value.product.type === "save") {
                        this.rows[index].translates = value.product.name;
                        this.rows[index].vendor_price = value.product.price;
                        this.rows[index].quantity = value.product.quantity;
                        this.rows[index].filemanager_thumb = value.product.filemanager_thumb;
                        this.rows[index].image = value.product.image;
                        this.rows[index].visual_labels = value.product.visual_labels;
                        this.rows[index].date_available_day = value.product.date_available_day;
                        this.rows[index].sku = value.product.sku;
                        this.rows[index].href = value.product.href;
                        if (value.product.quantity_other_variants !== undefined) {
                            this.rows[index].variants_count = value.product.quantity_other_variants.toString();
                        }
                        this.$set(this.savedOriginal, originalPosition, this.$root.copy(this.rows[index]));
                        this.rows[index].changed = true;
                        this.savedOriginal[originalPosition].changed = true;
                        setTimeout(() => {
                            this.rows[index].changed = false;
                            this.savedOriginal[originalPosition].changed = false;
                        }, 4000);
                    }
                }
                if (document.getElementById('perfect-scroll-offset') && value.scrollActivate === true) {
                    let offset = this.scrollOffset;
                    setTimeout(() => {
                        document.getElementById('perfect-scroll-offset').scrollTop = offset;
                        value.scrollActivate = false;
                    }, 100);
                }
            },
        }
    }
</script>

<style scoped>

</style>
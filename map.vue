<template>
    <div> <!-- without an outer container div this component template will not render -->
        <loading-spinner v-if="!dataLoaded"></loading-spinner>
        <transition name="fade">
            <div v-if="dataLoaded" v-cloak>
                <div class="inside_header_background" v-if="pageBanner" v-bind:style="{ backgroundImage: 'url(' + pageBanner.image_url + ')' }">
                    <div class="main_container">
                        <div class="page_container">
                            <h2>Center Map</h2>
                        </div>
                    </div>
                </div>
                <div class="main_container margin_30">
                    <div class="details_row">
                        <div class="details_col_3">
                            <div class="hidden_phone">
                                <h3 class="inside_page_title">Find Store</h3>
                                <div class="store_list_container hidden-mobile" v-if="alphaStores" >
                                    <span v-for="(stores, key) in alphaStores">
                                        <p class="store_heading">{{ key }}</p>
                                        <p class="store_name" v-for="store in stores" v-on:click="dropPin(store)">{{store.name}}</p>
                                    </span>
                                </div>
                            </div>
                            <div class="visible_phone">
                                <div class="position_relative">
                                    <search-component v-model="storeSearch" :list="processedStores" :suggestion-attribute="suggestionAttribute" @select="onOptionSelect" :threshold=".5">
                                        <template slot="item" scope="option">
                                            <article class="media">
                                                <p>{{ option.data.name }}</p>
                                            </article>
                                        </template>
                                    </search-component>
                                    <i id="store-search-icon" class="fa fa-search" aria-hidden="true"></i>
                                </div>
                            </div>
                        </div>
                        <div class="details_col_9">
                            <mapplic-png-map ref="pngmap_ref" :height="700" :hovertip="true" :storelist="allStores" :floorlist="floorList" :bindLocationOpened="true" :svgWidth="property.map_image_width" :svgHeight="property.map_image_height" :showPin="true" tooltiplabel="View Store Details"></mapplic-png-map>
                        </div>
                    </div>
                </div>
            </div>
        </transition>
    </div>
</template>
<script>
    define(["Vue", "vuex", "vue-select", "vue!mapplic-png-map", "vue!search-component"], function(Vue, Vuex, VueSelect, MapplicComponent, SearchComponent) {
        Vue.component('v-select', VueSelect.VueSelect);
        return Vue.component("stores-component", {
            template: template, // the variable template will be injected
            data: function() {
                return {
                    dataLoaded: false,
                    pageBanner: "",
                    suggestionAttribute: "name",
                    storeSearch: null,
                }
            },
            created (){
                this.loadData().then(response => {
                    var temp_repo = this.findRepoByName('Map Banner');
                    if(temp_repo) {
                        this.pageBanner = temp_repo.images[0];
                    }
                    
                    this.dataLoaded = true;
                });
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'findRepoByName',
                    'processedStores',
                    'storesByAlphaIndex'
                ]),
                alphaStores() {
                    console.log(this.storesByAlphaIndex);
                    return this.storesByAlphaIndex
                },
                allStores() {
                    
                    this.processedStores.map(function(store){
                        store.zoom = 1;
                    })
                    return this.processedStores;
                    
                    
                },
                getPNGurl() {
                    return "https://www.mallmaverick.com" + this.property.map_url;
                },
                pngMapRef() {
                    return this.$refs.pngmap_ref;
                },
                floorList () {
                    var floor_list = [];
                    
                    var floor_1 = {};
                    floor_1.id = "first-floor";
                    floor_1.title = "Floor 1";
                    floor_1.map = this.getPNGurl;
                    floor_1.z_index = 1;
                    floor_1.show = true;
                    
                    floor_list.push(floor_1);
                    return floor_list;
                }
            },
            methods: {
                loadData: async function () {
                    try {
                        let results = await Promise.all([this.$store.dispatch("getData", "repos")]);
                        return results;
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                },
                onOptionSelect(store) {
                    this.$nextTick(function() {
                        this.storeSearch = ""
                    });
                    // this.$refs.mapplic_ref.showLocation(option.svgmap_region);
                    this.pngMapRef.showLocation(store.id);
                },
                dropPin(store) {
                    this.pngMapRef.showLocation(store.id);
                }
            }
        });
    });
</script>
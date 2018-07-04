<template>
    <div><!-- without an outer container div this component template will not render -->
        <loading-spinner v-if="!dataLoaded"></loading-spinner>
        <transition name="fade">
            <div v-if="dataLoaded" v-cloak>
                <div class="home_banner_container">
                    <slick ref="slick" :options="slickOptions">
                        <div v-if="homeBanners" v-for="banner in homeBanners">
                            <a v-if="banner.url" :href="banner.url" class="">
                                <div class="banner_image" v-bind:style="{ backgroundImage: 'url(' + banner.image_url + ')' }"></div>
                            </a>
                            <div v-else class="banner_image" v-bind:style="{ backgroundImage: 'url(' + banner.image_url + ')' }"></div>
                        </div>
                    </slick>
                </div>
                <div class="main_container">
                    <div v-if="featureItems" class="row">
                        <div v-for="item in featureItems" class="col-sm-4 feature_item">
                            <a :href="item.url">
                                <img :src="item.image_url" :alt="item.name" />
                            </a>
                        </div>
                    </div>
                </div>
            </div>
        </transition>
    </div>
</template>
<script>
    define(["Vue", "vuex", "vue!vue-slick"], function (Vue, Vuex, slick) {
        return Vue.component("home-component", {
            template: template, // the variable template will be injected
            data: function() {
                return {
                    dataLoaded: false,
                    slickOptions: {
                        arrows: false,
                        autoplay: true,
                        autoplaySpeed: 6000,
                        cssEase: 'linear',
                        dots: true,
                        fade: false,
                        infinite: true,
                        slidesToShow: 1,
                        speed: 1000
                    }
                }
            },
            created(){
                this.loadData().then(response => {
                    this.dataLoaded = true;  
                });
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'timezone',
                    'getPropertyHours'
                ]),
                homeBanners() {
                    var banners = [];
                    _.forEach(this.$store.state.banners, function (value, key) {
                        var today = new Date();
                        var start = new Date (value.start_date);
                        if (start <= today){
                            if (value.end_date){
                                var end = new Date (value.end_date);
                                if (end >= today){
                                    banners.push(value);  
                                }
                            } else {
                                banners.push(value);
                            }
                        }
                    });
                    return banners
                },
                featureItems() {
                    var features =  _.slice(this.$store.state.feature_items, 0, 3);
                    _.forEach(features, function(value, key) {
                        if( _.includes(value.name, 'Dining')) {
                            value.prop = 'dining';
                        }
                    });
                    return features;
                }
            },
            methods: {
                loadData: async function() {
                    try {
                        let results = await Promise.all([this.$store.dispatch("getData", "banners"), this.$store.dispatch("getData", "feature_items")]);
                        return results;
                    } catch(e) {
                        console.log("Error loading data: " + e.message);    
                    }
                }
            }
        })
    })
</script>
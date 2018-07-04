<template>
    <div> <!-- Without an outer container div this component template will not render -->
        <loading-spinner v-if="!dataLoaded"></loading-spinner>
        <transition name="fade">
            <div v-if="dataLoaded" v-cloak>
                <div class="inside_header_background" v-if="pageBanner" v-bind:style="{ backgroundImage: 'url(' + pageBanner.image_url + ')' }">
                    <div class="main_container">
                        <h2>Jobs</h2>
                    </div>
                </div>
                <div class="main_container mobile_padding margin_30">
                    <div class="details_row">
                        <div class="details_col_3 hidden_phone">
                            <img class="img_max" src="//codecloud.cdn.speedyrails.net/sites/5b2960236e6f647f54020000/image/jpeg/1530645301000/side_banner1.jpg" alt="" />    
                        </div>
                        <div class="details_col_9" v-if="currentJob">
                            <router-link to="/jobs">
                                <div class="inside_page_header"><i class="fa fa-caret-left"></i> Back to List</div>
                            </router-link>
                            <h3 class="promo_name">{{ currentJob.name }}</h3>
                            <p class="promo_store_name">
                                <router-link v-if="currentJob.jobable_type == 'Store'" :to="'/stores/'+ currentJob.store.slug">
                                    {{ currentJob.store.name }}
                                </router-link>
                                <span v-else>{{ property.name }}</span>
                                <span>| </span>
                                <span v-if="isMultiDay(currentJob)" class="promo_date">{{ currentJob.start_date | moment("MMMM D", timezone)}} to {{ currentJob.end_date | moment("MMMM D", timezone)}}</span>
                                <span v-else class="promo_date">{{ currentJob.start_date | moment("MMMM D", timezone)}}</span>
                            </p>
                            <div class="promo_desc" v-html="currentJob.rich_description"></div>
                            <social-sharing v-if="currentJob" :url="shareURL(currentJob.slug)" :title="currentJob.title" :description="currentJob.body" :quote="truncate(currentJob.body)" :twitter-user="siteInfo.twitterHandle" :media="currentJob.image_url" inline-template>
                                <div class="social_share">
                                    <p>Share</p>
                                    <network network="facebook">
                                        <i class="fab fa-facebook"></i>
                                    </network>
                                    <network network="twitter">
                                        <i class="fab fa-twitter"></i>
                                    </network>
                                </div>
                            </social-sharing>
                        </div>
                    </div>
                </div>
            </div>
        </transition>
    </div>
</template>

<script>
    define(["Vue", "vuex", "moment", "moment-timezone", "vue-moment", "vue-lazy-load",  "vue-social-sharing", "json!site.json"], function(Vue, Vuex, moment, tz, VueMoment, VueLazyload, SocialSharing, site) {
        Vue.use(VueLazyload);
        Vue.component('social-sharing', SocialSharing);
        return Vue.component("job-details-component", {
            template: template, // the variable template will be injected,
            props: ['id'],
            data: function() {
                return {
                    dataLoaded: false,
                    pageBanner: "",
                    currentJob: null,
                    siteInfo: site,
                }
            },
            created() {
                this.$store.dispatch("getData", "repos").then(response => {
					var temp_repo = this.findRepoByName('Jobs Banner');
                    if(temp_repo) {
                        this.pageBanner = temp_repo.images[0];
                    }
				}, error => {
					console.error("Could not retrieve data from server. Please check internet connection and try again.");
				});
				
				this.$store.dispatch("getData", "jobs").then(response => {
					this.currentJob = this.findJobBySlug(this.id);
					if (this.currentJob === null || this.currentJob === undefined) {
						this.$router.replace({ path: '/jobs' });
					}
					this.dataLoaded = true;
				}, error => {
					console.error("Could not retrieve data from server. Please check internet connection and try again.");
				});
			},
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'timezone',
                    'findRepoByName',
                    'findJobBySlug'
                ])
            },
            methods: {
				isMultiDay(currentJob) {
					var timezone = this.timezone
					var start_date = moment(currentJob.start_date).tz(timezone).format("MM-DD-YYYY")
					var end_date = moment(currentJob.end_date).tz(timezone).format("MM-DD-YYYY")
					if (start_date === end_date) {
						return false
					} else {
						return true
					}
				},
				truncate(val_body) {
                    var truncate = _.truncate(val_body, { 'length': 99, 'separator': ' ' });
                    return truncate;
                },
				shareURL(slug) {
                    var share_url = window.location.href
                    return share_url
                }
			}
        });
    });
</script>
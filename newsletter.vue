<template>
    <div> <!-- without an outer container div this component template will not render -->
        <loading-spinner v-if="!dataLoaded"></loading-spinner>
        <transition name="fade">
            <div v-if="dataLoaded" v-cloak>
                <div class="inside_header_background" v-if="pageBanner" v-bind:style="{ background: '#000 url(' + pageBanner.image_url + ')' }">
                    <div class="main_container">
                        <div class="page_container">
                            <h2>Newsletter</h2>
                        </div>
                    </div>
                </div>
                <div class="main_container margin_30">
                    <div class="details_row">
                        <div class="details_col_3 hidden_phone">
                            <h3 class="inside_page_title">Address</h3>
                            <p class="inside_page_link">
                                {{ property.address1 }} <br>
                                {{ property.city }}, {{ property.province_state }} <br>
                                {{ property.postal_code }}
                            </p>
                        </div>
                        <div class="details_col_9">
                            <p class="inside_page_link">Be the first to know about upcoming events and special announcements from {{ property.name }}!</p>
                            <form class="form-horizontal js-cm-form" id="subForm" action="https://www.createsend.com/t/subscribeerror?description=" method="post" data-id="92D4C54F0FEC16E5ADC2B1904DE9ED1A2D25E274BC1E4651B49B12DDAABE44B43B30C41BA099348D43D606789D4B459471C241D925F2A0191E4DACEF3F4F11E8" >
                                <div class="row">
                                    <div class="col-sm-8">
                                        <label for="cm-name">Name </label>
                                        <input class="margin_20 form-control" aria-label="Name" id="cm-name" maxlength="200" name="cm-name">
                                    </div>
                                    <div class="col-sm-8">
                                        <label for="newsletter_email">Email </label>
                                        <input autocomplete="Email" aria-label="Email" class="js-cm-email-input qa-input-email margin_20 form-control" id="newsletter_email" maxlength="200" name="cm-vhtsh-vhtsh" required="" type="email">
                                    </div>
                                    <div class="col-sm-8">
                                        <div style="margin-left: 20px">
                                            
                                                
                                            <label class="checkbox" for="cm-privacy-consent">
                                                <input aria-required="" id="cm-privacy-consent" name="cm-privacy-consent" required="" type="checkbox">
                                                I agree to receive communications from Fox Run Square.
                                                    
                                            </label>
                                
                                            <input id="cm-privacy-consent-hidden" name="cm-privacy-consent-hidden" type="hidden" value="true">
                                        </div>
                                    </div>
                                    <div class="margin_20 clearfix"></div>
                                    <div class="col-xs-12">
                                        <button class="animated_btn" type="submit" >Subscribe</button>
                                    </div>
                                </div>
                            </form>
                        </div>
                    </div>
                </div>
            </div>
        </transition>
    </div>
</template>
<script>
    define(["Vue", "vuex", "jquery", "vee-validate", "json!site.json", "campaignMonitor"], function(Vue, Vuex, $, VeeValidate, site, campaignMonitor) {
        Vue.use(VeeValidate);
        return Vue.component("newsletter-component", {
            template: template, // the variable template will be injected
            data: function() {
                return {
                    dataLoaded: true,
                    pageBanner: "",
                    siteInfo: site,
                    form_data : {},
                    formSuccess : false,
                    formError: false
                }
            },
            created (){
                this.loadData().then(response => {
                    var temp_repo = this.findRepoByName('Newsletter Banner');
                    if(temp_repo) {
                        this.pageBanner = temp_repo.images[0];
                    }

                    this.dataLoaded = true;
                });
            },
            mounted () {
                this.form_data.email = this.$route.query.email;
                $("#newsletter_email").val(this.form_data.email);
            },
            watch : {
                $route () {
                    this.form_data.email = this.$route.query.email;
                    $("#newsletter_email").val(this.form_data.email);
                }
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'findRepoByName'
                ])
            },
            methods: {
                loadData: async function () {
                    try {
                        let results = await Promise.all([this.$store.dispatch("getData", "repos")]);
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                },
                validateBeforeSubmit(form) {
                    this.$validator.validateAll().then((result) => {
                        if (result) {
                            let errors = this.errors;
                            
                            if(errors.length > 0) {
                                console.log("Error");
                            } else {
                                console.log("No Error");
                                // return true;
                                form.target.submit();
                            }
                        }
                    })
                }
            }
        });
    });
</script>
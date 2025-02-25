<template>

            <v-card-text style="color: #333;">
                <div class="pa-3" v-if="data">
                    <v-row>
                        <v-col>

                            <div :class="{editable: !editable}" class="text-summary">
                                <p>
                                    At a projected annual spend of <strong>{{this.cost | currency}},</strong> subscribing to

                                    <span v-if="subscribedJournals.length">
                                        only the <strong>{{ subscribedJournals.length}}</strong> most <a
                                            href="https://support.unpaywall.org/support/solutions/articles/44001822684-cost-effectiveness" target="_blank">cost-effective</a> journals
                                    </span>
                                    <span v-if="!subscribedJournals.length">
                                        <em>none</em> of your baseline journals
                                    </span>
                                     saves <strong>{{100 - (subrCostPercent + illCostPercent) | round}}%</strong> off your current package subscription cost, while providing instant fulfillment for <strong>{{instantUsage | round}}%</strong> of (weighted) usage.
                                </p>

                                <p>
                                    <v-btn
                                            @click='$store.commit("setEditMode")'
                                            v-if="!$store.state.editMode"
                                            class="mt-4"
                                            depressed color="info">edit</v-btn>
                                </p>



                                <v-alert text type="info" v-if="false && journalsCheaperToSubscribe.length">
                                    <div>
                                        You could save money by subscribing to the <strong>{{ journalsCheaperToSubscribe.length }}</strong> journals where subscription is cheaper than ILL.
                                    </div>
                                    <div>
                                        <v-btn @click='$store.dispatch("openWizard")' class="mt-4" depressed color="info">show me</v-btn>
                                    </div>
                                </v-alert>



                            </div>

                            <div v-if="$store.state.editMode" class="mt-8">
                                <v-btn depressed
                                       :loading="makeItSoLoading"
                                       @click="makeItSo"
                                       class="mr-2"
                                       color="info">
                                    Save subscriptions</v-btn>
                                <v-btn depressed
                                       @click="$store.commit('clearEditMode')"

                                       outlined>cancel</v-btn>
                            </div>

                        </v-col>
                        <v-col cols="1 slider-col lift" >
                            <v-slider
                                    v-model="sliderPercent"
                                    color="info"
                                    vertical
                                    :disabled="!$store.state.editMode"
                                    @end="sliderEnd"
                            ></v-slider>
                        </v-col>
                        <v-col :cols="barCols" class="currency-area lift">
                            <div class="under-bar">
                                <div class="num font-weight-bold main-number ">
                                    {{(subrCostPercent+illCostPercent) | round}}%
                                </div>
                                <div class="label">
                                        of base cost
                                </div>
                            </div>
                            <div class="bar-wrapper">
                                <div class="bar-fill"></div>
                                <div class="bar cost" :style="{height: subrCostPercent+'%'}">
                                    <strong>{{subrCostPercent | round}}%</strong>
                                    Subscription
                                </div>
                                <div class="bar cost" :style="{height: illCostPercent +'%'}">
                                    <strong>{{illCostPercent | round}}%</strong>
                                    ILL
                                </div>
                            </div>

                        </v-col>


                        <v-col :cols="barCols" class="usage-area lift">
                            <div class="under-bar">
                                <div class="num font-weight-bold main-number ">
                                    {{instantUsage | round}}%
                                </div>
                                <div class="label">
                                        Instant Usage
                                </div>
                            </div>

                            <div class="bar-wrapper">
                                <div class="bar delayed bar-fill">
                                    <strong>{{usage.ill + usage.otherDelayed | round}}%</strong>
                                    ILL and other
                                </div>
                                <div class="bar paid instant" :style="{height: usage.subr+'%'}">
                                    <strong>{{usage.subr | round}}%</strong>
                                    Subscription
                                </div>
                                <div class="bar free instant" :style="{height: usage.oa+'%'}">
                                    <strong>{{usage.oa | round}}%</strong>
                                    OA
                                </div>
                                <div class="bar free instant" :style="{height: usage.backfile+'%'}">
                                    <strong>{{usage.backfile | round}}%</strong>
                                    Backfile
                                </div>
                                <div class="bar free instant" :style="{height: usage.asn+'%'}">
                                    <strong>{{usage.asn | round}}%</strong>
                                    ResearchGate etc
                                </div>
                            </div>

                        </v-col>


                        <v-col :cols="barCols" class="journals-area lift">
                            <div class="under-bar">
                                <div class="num font-weight-bold main-number ">
                                    {{subscribedJournals.length}}
                                </div>
                                <div class="label">
                                        subscribed journals
                                </div>
                            </div>
                            <div class="dots-bar-wrapper">
                                    <div v-for="journal in data.journals"
                                         :key="journal.issn_l"
                                         class="journal-dot"
                                         :class="{subscribed: journal.subscribed}"
                                    >
                                    </div>

                            </div>
                        </v-col>

                    </v-row>
                </div>

            </v-card-text>

</template>

<script>
    import _ from "lodash"
    import CostStick from "../../components/CostStick"

    export default {
        props: ["data", "editable"],
        components: {CostStick},
        name: "SliderTab",
        data() {
            return {
                sliderPercent: 0,
                totalUsage: 0,
                barHeight: 500,
                makeItSoLoading: false,
                // subrCost:0,
                // illCost: 0,
            }
        },
        computed: {
            // data() {
            //     return this.$store.state.wizardData
            //
            // },
            showMe() {
                return this.$store.state.wizardOpen
            },
            journalsCheaperToSubscribe(){
                return this.data.journals.filter(j=>{
                    return j.cost_subscription_minus_ill < 0
                })
            },
            cost() {
                // i think maybe this is garbage...
                return .01 * (this.subrCostPercent + this.illCostPercent) * this.data._summary.cost_bigdeal_projected
            },
            costFromSlider() {
                const sliderCost = .01 * this.sliderPercent * this.data._summary.cost_bigdeal_projected
                return sliderCost
                return Math.max(sliderCost, this.illCost)
            },
            subrCostPercent() {
                return 100 * this.subrCost / this.data._summary.cost_bigdeal_projected
            },
            illCostPercent() {
                return 100 * this.illCost / this.data._summary.cost_bigdeal_projected
            },
            costFromSubrs() {
                const costs = this.data.journals.map(j => {
                    if (j.subscribed) {
                        return j.cost_subscription
                    } else {
                        return j.cost_ill
                    }
                })
                return costs.reduce((a, b) => a + b)
            },
            subrCost() {
                return this.data.journals
                    .filter(j => !!j.subscribed)
                    .map(j => j.cost_subscription)
                    .reduce((a, b) => a + b, 0)
            },
            illCost() {
                return this.data.journals
                    .filter(j => !j.subscribed)
                    .map(j => j.cost_ill)
                    .reduce((a, b) => a + b, 0)
            },
            loading() {
                return this.$store.state.tabDataLoading
            },
            barCols(){
                return 2
                return (this.editable) ? 2 : 3
            },


            subscribedJournals() {
                return this.data.journals.filter(j => !!j.subscribed)
            },
            usage() {
                const ret = {
                    oa: 0,
                    backfile: 0,
                    asn: 0,
                    ill: 0,
                    otherDelayed: 0,
                    subr: 0,
                }
                this.data.journals.forEach(j => {
                    ret.oa += j.use_groups_free_instant.oa
                    ret.backfile += j.use_groups_free_instant.backfile
                    ret.asn += j.use_groups_free_instant.social_networks

                    if (j.subscribed) {
                        ret.subr += j.use_groups_if_subscribed.subscription
                    } else {
                        ret.ill += j.use_groups_if_not_subscribed.ill
                        ret.otherDelayed += j.use_groups_if_not_subscribed.other_delayed
                    }
                })
                const total = Object.values(ret).reduce((a, b) => a + b) || 1
                Object.keys(ret).forEach(k => {
                    ret[k] = 100 * ret[k] / total
                })


                return ret
            },
            instantUsage() {
                const usage = this.usage
                return usage.oa + usage.backfile + usage.asn + usage.subr
            },
            tabDataDigest() {
                return this.$store.getters.tabDataDigest
            },
            instantUsagePercent() {
            },
            numSubr() {
                return 0
            },
            numJournals() {
                return this.data && this.data.journals.length
            },
            journalSubjects() {
                const dict = _.groupBy(this.data.journals, j => {
                    return j.subject
                })
                const subjects = Object.keys(dict).map(k => {
                    return {
                        name: k,
                        journals: dict[k]
                    }
                })
                return subjects
            },
            topCost() {
                return 31091 // tetrahedron letters
                return Math.max(...this.data.journals.map(j => j.cost_subscription))
            },
            topPaidUsage() {
                return 4283.8 // cell

                return Math.max(...this.data.journals.map(j => {
                    return j.use_groups_if_subscribed.subscription
                }))
            },
            journalsXy() {

                return this.data.journals.map(j => {
                    return {
                        ...j,
                        x: (100 * Math.log(j.cost_subscription) / Math.log(this.topCost)) + '%',
                        y: (100 * Math.log(j.use_groups_if_subscribed.subscription) / Math.log(this.topPaidUsage)) + '%'
                    }
                })
            },
            editMode(){
                return this.$store.state.editMode
            },


        },
        methods: {
            sliderEnd() {
                if (this.sliderPercent < this.illCostPercent) {
                    this.sliderPercent = this.illCostPercent
                }
            },
            makeItSo(){
                this.$store.commit("clearEditMode")
                this.makeItSoLoading = true
                const subrIssnls = this.data.journals
                    .filter(j => j.subscribed)
                    .map(j => j.issn_l)
                this.$store.dispatch("setSubrs", subrIssnls)
                    .then(r => {
                        this.makeItSoLoading = false
                        this.$store.commit('clearWizard')
                        this.$store.commit('snackbar', "Subscriptions updated!", "info")
                    })
            },
            updateJournals() {
                if (!this.data) return

                const myMax = this.costFromSlider

                // unsubscribe all
                this.data.journals.forEach(j => j.subscribed = false)

                // ILL cost must be paid regardless
                let mySpendSoFar = this.data.journals.map(j => j.cost_ill).reduce((a, b) => a + b, 0)

                // subscribe to journals where subr is cheaper than ILL
                this.data.journals.forEach(j => {
                    if (j.cost_subscription_minus_ill < 0) {
                        j.subscribed = true
                        mySpendSoFar += j.cost_subscription_minus_ill
                    }
                })

                if (mySpendSoFar >= myMax) return


                // subscribe to as many other journals as we can afford
                this.data.journals.forEach(j => {
                    mySpendSoFar += j.cost_subscription_minus_ill
                    if (mySpendSoFar <= myMax) {
                        j.subscribed = true
                    } else {
                        j.subscribed = false
                    }
                })

                // set the subrCost
                // this.subrCost = this.data.journals
                //     .filter(j=>j.subscribed)
                //     .map(j => j.cost_subscription_minus_ill)
                //     .reduce((a, b) => a + b, 0)

            },
        },
        mounted() {

        },
        watch: {
            sliderPercent: function (to, from) {
                console.log("cost percent changed")
                this.updateJournals()
            },

            editMode: function (to, from) {
                console.log("edit mode changed")
                if (this.editMode){
                    this.sliderEnd()
                }
            },





            // 'tabDataDigest': {
            //     deep: false,
            //     handler: function (to, from) {
            //         console.log("tabData changed", to)
            //         if (!this.data || !this.data._summary) return
            //         this.sliderEnd()
            //
            //     }
            // },


        }
    }
</script>

<style lang="scss">
    $bar-height: 400px;

    .text-summary {
        font-size: 20px;
        line-height: 1.5;
        &.editable {
            font-size: 20px;
        }
    }

    .under-bar {
        padding: 10px 0;
        line-height: 1;

        .main-number {
            font-size: 50px;
            color: #333;
            text-align: center;
            width: 100%;
        }


        .label {
            font-size: 14px;
            text-align: center;

        }
    }

    .journal-dot {
        background: #ccc;
        height: 4px;
        width: 4px;
        border-radius: 5px;
        margin: 1px 1px 0 0;
        line-height: .1;
        display:block;

        &.subscribed {
            background: dodgerblue;
        }
    }


    .lift {
        margin-top: -80px;
    }
    .slider-col {
        padding-top: 100px;
    }
    .v-slider--vertical {
        height: $bar-height !important;
        margin: 0 !important;
    }

    .dots-bar-wrapper {
        height: $bar-height;
        display: block;
        display:flex;
        flex-wrap: wrap-reverse;
    }

    .bar-wrapper {
        height: $bar-height;
        display: flex;
        flex-direction: column;

        .bar-fill {
            background: #ccc;
            flex-grow: 1000;
        }

        .bar {
            padding-left: 3px;
            border-top: 1px solid #eee;
            font-size: 12px;
            color: #fff;

            &.cost {
                background: #555;
            }

            &.free {
                background: green;
            }

            &.paid.instant {
                background: dodgerblue;
            }
        }
    }

</style>
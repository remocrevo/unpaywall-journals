<template>
    <v-container class="account">
        <breadcrumbs></breadcrumbs>
        <h1 class="display-3">Your Account</h1>
        <v-card outlined>
            <v-card-title>Your Packages</v-card-title>
            <v-card-text>
                <v-simple-table>
                    <thead>
                    <tr>
                        <th class="text-left">Name</th>
                        <th class="text-left">Journals</th>
                        <th class="text-left">COUNTER uploaded</th>
                    </tr>
                    </thead>
                    <tbody>
                    <tr v-for="pkg in $store.state.pkgs"
                        :key="pkg.id"
                        @click="$router.push(`/a/${account.id}/${pkg.id}`)"
                        style="cursor:pointer;">
                        <td>{{ pkg.name }}</td>
                        <td>{{ pkg.numJournals }}</td>
                        <td>{{ pkg.hasCounterData }}</td>
                    </tr>
                    </tbody>
                </v-simple-table>
            </v-card-text>
            <v-card-actions>
                <div class="wrap" @click="$store.commit('openNotSupportedMsg')">
                    <v-btn disabled depressed color="primary">Add new package</v-btn>
                </div>
            </v-card-actions>
        </v-card>

    </v-container>
</template>

<script>
    import Breadcrumbs from "../components/Breadcrumbs"
    export default {
        name: "Account",
        components: {Breadcrumbs},
        data() {
            return {
            }
        },
        methods: {
            increment() {
                this.$store.commit("increment")
            }
        },
        computed: {
            count() {
                return this.$store.getters.count
            },
            account() {
                return this.$store.state.user
            },

        },
        mounted() {
            console.log("mount up")
            this.$store.commit("clearPkg")
            this.$store.commit("clearScenario")

        },
    }
</script>

<style scoped>

</style>
<template>
    <v-container>
        <v-card class="pa-3" >
            <h2 class="display-1">Help</h2>
                            <v-alert text type="info">
                                Our help articles are currently hosted on the Knowledge Base, but will be moved here soon.
                            </v-alert>
                            <div>
                                <v-btn depressed
                                       href="https://support.unpaywall.org/support/solutions/articles/44001822205-getting-started"
                                       target="_blank"
                                       color="primary">
                                    visit knowledge base
                                    <v-icon right>mdi-open-in-new</v-icon>
                                </v-btn>
                            </div>

        </v-card>


    </v-container>
</template>

<script>
    export default {
        props: [],

        name: "Support",
        computed: {
        }
    }
</script>

<style scoped>

</style>
<template>
    <div id="way-block" class="way-block">
    <v-container grid-list-md text-md-center class="inside-container" >
        <v-layout row align-center fill-height class="test">
            <v-flex xs12 sm12 md12 lg8 offset-lg2  >
                <v-card color="green darken-2" class="inside-card">
                    <v-container grid-list-md text-md-center>
                        <v-layout row wrap>
                            <v-flex xs12 sm12 lg4 md4 class="way-input">
                                <v-autocomplete
                                        label="З відки?"
                                        :items="stations"
                                        placeholder="Місто відправки"
                                        append-icon="place"
                                        v-model="from"
                                ></v-autocomplete>
                            </v-flex>

                            <v-flex xs12 sm0 lg1 md1>
                                <i class="material-icons center-icon">
                                    swap_calls
                                </i>
                            </v-flex>

                            <v-flex xs12 sm12 lg4 md4 class="way-input" >
                                <v-autocomplete
                                        label="Куди?"
                                        :items="stations"
                                        placeholder="Місто прибуття"
                                        append-icon="place"
                                        v-model="to"
                                ></v-autocomplete>
                            </v-flex>
                            <v-flex xs12 sm12 lg1 md1>
                                <i class="material-icons right-icon">event</i>
                            </v-flex>
                            <v-flex xs12 sm12 lg2 md2 class="way-input">
                                <v-menu :close-on-content-click="false" :nudge-right="40"
                                        lazy
                                        transition="scale-transition"
                                        offset-y
                                        full-width
                                        min-width="290px"
                                >
                                    <v-text-field
                                            slot="activator"
                                            v-model="date"
                                            label="Коли?"
                                            placeholder="День відправки"
                                            readonly
                                    ></v-text-field>
                                    <v-date-picker v-model="date"></v-date-picker>
                                </v-menu>
                            </v-flex>
                        </v-layout>
                    </v-container>
                </v-card>
                <v-flex xs12>
                    <v-btn color="orange" dark
                           @click="LOAD_TICKETS"
                           :disabled="dialog"
                           :loading="dialog"
                    >
                        <span class="find">Знайти квиток</span>
                        <v-icon>arrow_forward</v-icon>
                    </v-btn>
                    <v-dialog
                            v-model="dialog"
                            hide-overlay
                            persistent
                            width="300"
                    >
                        <v-card
                                color="primary"
                                dark
                        >
                            <v-card-text>
                                Відбувається загрузка білетів
                                <v-progress-linear
                                        indeterminate
                                        color="white"
                                        class="mb-0"
                                ></v-progress-linear>
                            </v-card-text>
                        </v-card>
                    </v-dialog>
                </v-flex>
            </v-flex>
        </v-layout>
    </v-container>
    </div>
</template>

<script>

    import { mapGetters } from 'vuex';

    export default {
        name: "WayChooser",
        data: () => ({
            dialog: false,
            stations:  []
        }),
        watch: {
            dialog (val) {
                if (!val) return;

                setTimeout(() => (this.dialog = false), 4000)
            }
        },
        methods: {
            LOAD_TICKETS: function () {
                if (this.from.length > 0 && this.to.length > 0 && new Date(this.def_date) >= new Date(new Date().toJSON().slice(0,10).replace(/-/g,'-'))) {
                    this.$store.dispatch("SET_TICKETS", []);
                    this.$store.dispatch('SET_ACTIVE_PAGE');
                    this.$store.dispatch("BLOCK_LOADER_ACTIVATE", "contentBlock");

                    let from = this.def_from;
                    let to = this.def_to;

                    let filter = {
                        FROM_PLACE: this.def_stations_list.filter(function (val) {
                            return val.NAME === from
                        })[0].ID,
                        TO_PLACE: this.def_stations_list.filter(function (val) {
                            return val.NAME === to
                        })[0].ID,
                        FROM_DATE: this.def_date,
                        TYPE: this.def_type
                    };

                    this.GET_FILTERED_TICKETS(this, window.api.storage.getToken(), filter).then(function (res) {
                        if (res.data !== undefined) {
                            res.this.$store.dispatch("SET_TICKETS", res.data);
                        }
                        res.this.$store.dispatch("BLOCK_LOADER_DEACTIVATE", "contentBlock");
                    });


                }
            },

            GET_FILTERED_TICKETS: async (component, token, filter) => {
                {
                    let data = await window.api.ticket.getFilteredTickets(token, filter);

                    if (data.status === 200) {
                        return  {this: component, data: data.data};
                    } else {
                        return {this: component, data: undefined};
                    }
                }
            },

            GET_STATIONS: async (component, token) => {
                {
                    let data = await window.api.stations.getAll(token);


                    if (data.status === 200) {
                        component.stations  = Object.values(data.data);

                        component.$store.commit('SET_STATIONS', component.stations);
                        console.log(component.stations)
                    }
                }
            },

            /**
             * @return {string}
             */
            GET_KEY_BY_VALUE: function (object, value) {
                return Object.keys(object).find(key => object[key] === value);
            },
            LOAD_STATIONS_WITH_KEY: async (component, token) => {
                {
                    let data = await window.api.stations.getWithKeys(token);


                    if (data.status === 200) {
                        component.$store.commit('SET_STATIONS_LIST', data.data);
                    }
                }
            }
        },
        mutations: {
            dialogC: function () {
                this.dialog = true;
            }
        },
        created() {
            let token = window.api.storage.getToken();

            this.GET_STATIONS(this, token);
            this.LOAD_STATIONS_WITH_KEY(this, token);
        },
        computed: {
            from: {
                get () {
                    return this.def_from;
                },
                set (value) {
                    this.$store.commit('SET_FROM', value)
                }
            },
            to: {
                get () {
                    return this.def_to;
                },
                set (value) {
                    this.$store.commit('SET_TO', value)
                }
            },
            date: {
                get () {
                    return this.def_date;
                },
                set (value) {
                    this.$store.commit('SET_DATE', value)
                }
            },
            ...mapGetters({
                def_from: 'GET_FROM',
                def_to: "GET_TO",
                def_date: "GET_DATE",
                def_type: "GET_TYPE",
                def_stations_list: "GET_STATIONS_LIST"
            })
        }
    }

</script>

<style lang="sass" scoped>

    .way-input
        color: white !important
        background-color: white
        padding: 20px !important
        padding-bottom: 0 !important

    #way-block
        padding-top: calc(50vh - 100px)
        @media all and (max-width: 960px)
            padding-top: calc(50vh - 238px)

        .inside-container
            z-index: 999

            .inside-card
                color: white

                .center-icon
                    margin-top: 30px

                .right-icon
                    margin-top: 30px

        .find
            font-size: 12px

</style>
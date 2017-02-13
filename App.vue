<template>
    <div class="wrap">
        <toolbar
                :has-selection="hasSelection"
                :has-photos="hasPhotos"
                :has-all-selected="hasAllSelected"
        >

        </toolbar>
        <dropzone
                :has-photos="hasPhotos"
                :has-selection="hasSelection"
        >
        </dropzone>
        <checkout></checkout>
        <ticket></ticket>
    </div>
</template>

<script type="text/babel">
    import Checkout from './components/checkout/Index.vue'
    import Toolbar from './components/Toolbar.vue'
    import Dropzone from './components/Dropzone.vue'
    import Ticket from './components/Ticket.vue'
    import {mapGetters, mapActions} from 'vuex'

    export default {
        name: 'app',
        components: {
            toolbar: Toolbar,
            checkout: Checkout,
            dropzone: Dropzone,
            ticket: Ticket
        },
        data () {
            return {
                files: [],
            }
        },
        created () {
            this.$store.dispatch('getOrder')
            this.$store.dispatch('getProperties')
            this.$store.dispatch('getCustomer')
        },
        mounted() {
            this.listen()
        },
        computed: {
            ...mapGetters({
                order: 'order',
                photos: 'photos',
                selection: 'selection'
            }),
            // Do we have images in our array?
            hasPhotos () {
                return this.photos.length > 0
            },

            // Do we have images in our selection?
            hasSelection () {
                return this.selection.length > 0
            },

            // Are all the images selected?
            hasAllSelected () {
                return this.selection.length === this.photos.length;
            }

        },
        methods: {
            listen() {
                Echo.private('orders.' + this.order.id)
                        .listen('PriceCalculated', event => {
                            this.$store.dispatch('setOrderPrice', event)
                        })
            }
        }
    }
</script>

<template>
    <div>
        <h2 class="title is-2 is-centered step-title">Afrekenen</h2>
        <form id="checkout-form">
            <div class="control cardinfo-card-number is-separate">
                <label class="label cardinfo-label" for="card-number">Kaart nummer</label>
                <div class='input-wrapper' id="card-number"></div>
                <div id="card-image" class="form-input-test"></div>
            </div>

            <div class="control cardinfo-wrapper">
                <div class="cardinfo-exp-date">
                    <label class="label cardinfo-label" for="expiration-date">Geldig tot</label>
                    <div class='input-wrapper form-input-test' id="expiration-date"></div>
                </div>

                <div class="control cardinfo-cvv">
                    <label class="label cardinfo-label" for="cvv">CVV</label>
                    <div class='input-wrapper' id="cvv"></div>
                </div>
            </div>
            <div class="control" v-if="hasCards">
                <span class="is-link is-underlined" @click="$store.dispatch('showChooseCreditcardStep')">bestaande kaart gebruiken?</span>
            </div>
            <div class="control">
                <button type="submit" id="button-pay" class="button is-primary" :class="{ 'is-loading': isLoading }">
                    <span>{{ order.total_price_formatted }} betalen</span>
                </button>
            </div>
        </form>
    </div>
</template>
<script type="text/babel">
    import {mapGetters, mapActions} from 'vuex'

    export default {
        computed: {
            ...mapGetters({
                isLoading: 'isLoading',
                hasCards: 'hasCards',
                order: 'order'
            })
        }
    }
</script>

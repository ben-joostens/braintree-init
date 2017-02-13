<template>
    <div class="checkout">
        <modal :is-active="isCheckingOut" :handle-close="closeCheckoutModal" modal-class="modal-checkout">
            <header slot="header">
                <img src="/img/checkout-header.png" width="360" height="259">
            </header>
            <div class="box" slot="content">
                <choose-payment-option v-if="isActiveStep(steps.CHOOSE_PAYMENT)"></choose-payment-option>
                <delivery v-if="isActiveStep(steps.CHOOSE_DELIVERY)"></delivery>
                <add-address v-if="isActiveStep(steps.ADD_ADDRESS)"></add-address>
                <add-card v-show="isActiveStep(steps.ADD_CARD)"></add-card>
                <select-card v-if="isActiveStep(steps.CHOOSE_CREDITCARD)"></select-card>
            </div>
        </modal>
    </div>
</template>

<script type="text/babel">
    import {mapGetters, mapActions} from 'vuex'
    import Modal from '../Modal.vue'
    import ChoosePaymentOption from './ChoosePaymentOption.vue'
    import AddAddress from '../address/Add.vue'
    import AddCard from './AddCard.vue'
    import SelectCard from './SelectCard.vue'
    import Delivery from './Delivery.vue'

    export default {
        components: {
            Modal,
            ChoosePaymentOption,
            Delivery,
            AddAddress,
            AddCard,
            SelectCard,
        },

        computed: {
            authToken() {
                if (_.isEmpty(this.customer.clientToken)) {
                    return Laravel.btToken
                }
                return this.customer.clientToken
            },
            ...mapGetters([
                'activeStep',
                'isCheckingOut',
                'isHomeDelivery',
                'customer',
                'order',
                'steps'
            ])
        },
        mounted () {
            this.createBraintree()
        },
        methods: {
            isActiveStep(step) {
                return (this.activeStep === step)
            },
            closeCheckoutModal () {
                this.$store.dispatch('haltCheckout')
            },
            createBraintree() {
                console.info('creating braintree...')
                let submit = document.getElementById('button-pay');
                let self = this;

                client.create({
                    authorization: self.authToken
                }, function (err, clientInstance) {
                    hostedFields.create({
                        client: clientInstance,
                        styles: {
                            'input': {
                                'font-size': '18px',
                                'color': '#363636'
                            },
                            '.number': {
                                'font-family': 'Ubuntu Mono'
                            },
                            '.valid': {
                                'color': '#2DAE70'
                            }
                        },
                        fields: {
                            number: { selector: '#card-number' },
                            cvv: { selector: '#cvv' },
                            expirationDate: { selector: '#expiration-date' }
                        }
                    }, function (err, hostedFieldsInstance) {
                        if (err) {
                            console.error(err);
                            return;
                        }
                        hostedFieldsInstance.on('cardTypeChange', function (event) {
                            if (event.cards.length === 1) {
                                let form = document.getElementById('card-image')
                                form.className = ""
                                form.classList.add(event.cards[0].type);
                            }
                        });

                        hostedFieldsInstance.on('validityChange', function (event) {
                            // Check if all fields are valid, then show submit button
                            var formValid = Object.keys(event.fields).every(function (key) {
                                return event.fields[key].isValid;
                            });

                            let button = document.getElementById('button-pay');

                            if (formValid) {
                                button.classList.remove('is-disabled');
                            } else {
                                button.classList.add('is-disabled');
                            }
                        });

                        submit.addEventListener('click', function (event) {
                            self.$store.dispatch('startLoading')

                            event.preventDefault();

                            hostedFieldsInstance.tokenize(function (err, payload) {
                                if (err) {
                                    console.error(err);
                                    return;
                                }

                                let data = {
                                    customer: self.customer,
                                    order: self.order,
                                    nonce: payload.nonce,
                                    paymentMethodToken: null
                                }

                                self.$store.dispatch('pay', data).then(() => {
                                    location = "/success/" + self.order.id;
                                })
                            });
                        }, false);
                    })
                });
            }
        }
    }
</script>

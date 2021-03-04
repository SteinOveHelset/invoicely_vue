<template>
    <div class="page-add-invoice">
        <div class="columns is-multiline">
            <div class="column is-12">
                <h1 class="title">Add invoice</h1>
            </div>

            <div class="column is-12">
                <h2 class="is-size-5 mb-4">Client</h2>

                <div class="select">
                    <select name="client" v-model="invoice.client">
                        <option value="">- Select client -</option>
                        <option 
                            v-for="client in clients"
                            v-bind:key="client.id"
                            v-bind:value="client"
                        >
                            {{ client.name }}
                        </option>
                    </select>
                </div>

                <div class="box mt-4" v-if="invoice.client">
                    <p><strong>{{ invoice.client.name }}</strong></p>
                    <p><strong>Email:</strong> {{ invoice.client.email }}</p>
                    <p v-if="invoice.client.address1">{{ invoice.client.address1 }}</p>
                    <p v-if="invoice.client.address2">{{ invoice.client.address2 }}</p>
                    <p v-if="invoice.client.zipcode || invoice.client.place">{{ invoice.client.zipcode }} {{ invoice.client.place }}</p>
                    <p v-if="invoice.client.country">{{ invoice.client.country }}</p>
                </div>
            </div>

            <div class="column is-12">
                <h2 class="is-size-5 mb-4">Items</h2>

                <ItemForm
                    v-for="item in invoice.items"
                    v-bind:key="item.item_num"
                    v-bind:initialItem="item"
                    v-on:updatePrice="updateTotals"
                >
                </ItemForm>

                <button class="button is-light" @click="addItem">+</button>
            </div>

            <div class="column is-12">
                <h2 class="is-size-5 mb-4">Misc</h2>

                <div class="field">
                    <label>Due days</label>

                    <div class="control">
                        <input type="number" class="input" v-model="invoice.due_days">
                    </div>
                </div>

                <div class="field">
                    <label>Sender reference</label>

                    <div class="control">
                        <input type="text" class="input" v-model="invoice.sender_reference">
                    </div>
                </div>
            </div>
        </div>

        <div class="column is-12">
            <h2 class="is-size-5 mb-4">Total</h2>

            <p><strong>Net amount</strong>: {{ invoice.net_amount }}</p>
            <p><strong>Vat amount</strong>: {{ invoice.vat_amount }}</p>
            <p><strong>Gross amount</strong>: {{ invoice.gross_amount }}</p>
        </div>

        <div class="column is-12">
            <button class="button is-success" @click="submitForm">Save</button>
        </div>
    </div>
</template>

<script>
import axios from 'axios'
import { toast } from 'bulma-toast'

import ItemForm from '@/components/ItemForm.vue'

export default {
    name: 'AddInvoice',
    components: {
        ItemForm
    },
    data() {
        return {
            invoice: {
                client: '',
                items: [
                    {
                        item_num: 0,
                        title: '',
                        unit_price: '',
                        quantity: 1,
                        vat_rate: 0,
                        net_amount: 0
                    }
                ],
                due_days: 14, 
                net_amount: 0,
                vat_amount: 0,
                gross_amount: 0
            },
            clients: []
        }
    },
    async mounted() {
        await this.getClients()
    },
    methods: {
        getClients() {
            axios
                .get('/api/v1/clients/')
                .then(response => {
                    this.clients = response.data
                })
                .catch(error => {
                    console.log(JSON.stringify(error))
                })
        },
        addItem() {
            this.invoice.items.push({
                item_num: this.invoice.items.length,
                title: '',
                unit_price: '',
                quantity: 1,
                vat_rate: 0,
                net_amount: 0
            })
        },
        updateTotals(changedItem) {
            let net_amount = 0
            let vat_amount = 0

            let item = this.invoice.items.filter(i => i.item_num === changedItem.item_num)

            item = changedItem

            for (let i = 0; i < this.invoice.items.length; i++) {
                const vat_rate = this.invoice.items[i].vat_rate

                vat_amount += this.invoice.items[i].net_amount * (vat_rate / 100)
                net_amount += this.invoice.items[i].net_amount
            }

            this.invoice.net_amount = net_amount
            this.invoice.vat_amount = vat_amount
            this.invoice.gross_amount = net_amount + vat_amount
            this.invoice.discount_amount = 0
        },
        submitForm() {
            this.invoice.client_name = this.invoice.client.name
            this.invoice.client_email = this.invoice.client.email
            this.invoice.client_org_number = this.invoice.client.org_number
            this.invoice.client_address1 = this.invoice.client.address1
            this.invoice.client_address2 = this.invoice.client.address2
            this.invoice.client_zipcode = this.invoice.client.zipcode
            this.invoice.client_place = this.invoice.client.place
            this.invoice.client_country = this.invoice.client.country
            this.invoice.client_contact_person = this.invoice.client.contact_person
            this.invoice.client_contact_reference = this.invoice.client.contact_reference
            this.invoice.client = this.invoice.client.id

            axios
                .post('/api/v1/invoices/', this.invoice)
                .then(response => {
                    toast({
                        message: 'The invoice was added',
                        type: 'is-success',
                        dismissible: true,
                        pauseOnHover: true,
                        duration: 2000,
                        position: 'bottom-right',
                    })

                    this.$router.push('/dashboard/invoices')
                })
                .catch(error => {
                    console.log(JSON.stringify(error))
                })
        }
    }
}
</script>

<style lang="scss">
    .select, .select select {
        width: 100%;
    }
</style>
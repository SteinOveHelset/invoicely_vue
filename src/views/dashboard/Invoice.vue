<template>
    <div class="page-invoice">
        <div class="columns is-multiline">
            <div class="column is-12">
                <h1 class="title">Invoice - {{ invoice.invoice_number }}</h1>

                <hr>

                <button @click="getPdf()" class="button is-dark">Download PDF</button>
            </div>

            <div class="column is-12">
                <h3 class="is-size-4">Client</h3>

                <p><strong>{{ invoice.client_name }}</strong></p>
                
                <p v-if="invoice.client_address1">{{ invoice.client_address1 }}</p>
                <p v-if="invoice.client_address2">{{ invoice.client_address2 }}</p>
                <p v-if="invoice.client_zipcode || invoice.client_place">{{ invoice.client_zipcode }} {{ invoice.client_place }}</p>
                <p v-if="invoice.client_country">{{ invoice.client_country }}</p>
            </div>

            <div class="column is-12">
                <h3 class="is-size-4">Items</h3>

                <table class="table is-fullwidth">
                    <thead>
                        <tr>
                            <td>Title</td>
                            <td>Quantity</td>
                            <td>Amount</td>
                        </tr>
                    </thead>

                    <tbody>
                        <tr 
                            v-for="item in invoice.items"
                            v-bind:key="item.id"
                        >
                            <td>{{ item.title }}</td>
                            <td>{{ item.quantity }}</td>
                            <td>{{ item.net_amount }}</td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>
    </div>
</template>

<script>
    import axios from 'axios'

    const fileDownload = require('js-file-download')

    export default {
        name: 'Invoice',
        data() {
            return {
                invoice: {},
                items: []
            }
        },
        mounted() {
            this.getInvoice()
        },
        methods: {
            getInvoice() {
                const invoiceID = this.$route.params.id

                axios
                    .get(`/api/v1/invoices/${invoiceID}`)
                    .then(response => {
                        this.invoice = response.data
                    })
                    .catch(error => {
                        console.log(JSON.stringify(error))
                    })
            },
            getPdf() {
                const invoiceID = this.$route.params.id

                axios
                    .get(`/api/v1/invoices/${invoiceID}/generate_pdf/`, {
                        responseType: 'blob',
                    }).then(res => {
                        fileDownload(res.data, `invoice_${invoiceID}.pdf`);
                    }).catch(err => {
                        console.log(err);
                    })
            }
        }
    }
</script>
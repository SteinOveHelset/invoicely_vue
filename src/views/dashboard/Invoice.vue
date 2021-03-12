<template>
    <div class="page-invoice">
        <nav class="breadcrumb" aria-label="breadcrumbs">
            <ul>
                <li><router-link to="/dashboard">Dashboard</router-link></li>
                <li><router-link to="/dashboard/invoices">Invoices</router-link></li>
                <li class="is-active"><router-link :to="{ name: 'Invoice', params: { id: invoice.id }}" aria-current="true">{{ invoice.invoice_number }}</router-link></li>
            </ul>
        </nav>

        <div class="columns is-multiline">
            <div class="column is-12">
                <h1 class="title">Invoice - {{ invoice.invoice_number }}</h1>

                <div class="buttons">
                    <button @click="getPdf()" class="button is-dark">Download PDF</button>
                    <button @click="setAsPaid()" class="button is-success" v-if="!invoice.is_paid">Set as paid</button>
                </div>
            </div>

            <div class="column is-12 mb-4">
                <div class="box">
                    <h3 class="is-size-4 mb-5">Client</h3>

                    <p><strong>{{ invoice.client_name }}</strong></p>
                    
                    <p v-if="invoice.client_address1">{{ invoice.client_address1 }}</p>
                    <p v-if="invoice.client_address2">{{ invoice.client_address2 }}</p>
                    <p v-if="invoice.client_zipcode || invoice.client_place">{{ invoice.client_zipcode }} {{ invoice.client_place }}</p>
                    <p v-if="invoice.client_country">{{ invoice.client_country }}</p>
                </div>
            </div>

            <div class="column is-12 mb-4">
                <div class="box">
                    <h3 class="is-size-4 mb-5">Items</h3>

                    <table class="table is-fullwidth">
                        <thead>
                            <tr>
                                <th>Title</th>
                                <th>Quantity</th>
                                <th>Vat rate</th>
                                <th>Total</th>
                            </tr>
                        </thead>

                        <tbody>
                            <tr 
                                v-for="item in invoice.items"
                                v-bind:key="item.id"
                            >
                                <td>{{ item.title }}</td>
                                <td>{{ item.quantity }}</td>
                                <td>{{ item.vat_rate }}%</td>
                                <td>{{ getItemTotal(item) }}</td>
                            </tr>
                        </tbody>
                    </table>
                </div>
            </div>

            <div class="column is-12">
                <div class="box">
                    <h3 class="is-size-4 mb-5">Summary</h3>

                    <div class="columns">
                        <div class="column is-6">
                            <p><strong>Net amount</strong>: {{ invoice.net_amount }}</p>
                            <p><strong>Vat amount</strong>: {{ invoice.vat_amount }}</p>
                            <p><strong>Gross amount</strong>: {{ invoice.gross_amount }}</p>
                            <p><strong>Bank account</strong>: {{ invoice.bankaccount }}</p>
                        </div>
                    
                        <div class="column is-6">
                            <p><strong>Our reference</strong>: {{ invoice.sender_reference }}</p>
                            <p><strong>Client reference</strong>: {{ invoice.client_contact_reference }}</p>
                            <p><strong>Due date</strong>: {{ invoice.get_due_date_formatted }}</p>
                            <p><strong>Status</strong>: {{ getStatusLabel() }}</p>
                            <p><strong>Invoice type</strong>: {{ getInvoiceType() }}</p>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    import axios from 'axios'
    import { toast } from 'bulma-toast'

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
            },
            getStatusLabel() {
                if (this.invoice.is_paid) {
                    return 'Is paid'
                } else {
                    return 'Is not paid'
                }
            },
            getInvoiceType() {
                if (this.invoice.invoice_type === 'creditnote') {
                    return 'Credit note'
                } else {
                    return 'Invoice'
                }
            },
            getItemTotal(item) {
                const unit_price = item.unit_price
                const quantity = item.quantity
                const total = item.net_amount + (item.net_amount * (item.vat_rate / 100))

                return parseFloat(total).toFixed(2)
            },
            async setAsPaid() {
                this.invoice.is_paid = true

                let items = this.invoice.items

                delete this.invoice['items']

                await axios
                    .patch(`/api/v1/invoices/${this.invoice.id}/`, this.invoice)
                    .then(response => {
                        toast({
                            message: 'The changes was saved',
                            type: 'is-success',
                            dismissible: true,
                            pauseOnHover: true,
                            duration: 2000,
                            position: 'bottom-right',
                        })
                    })
                    .catch(error => {
                        console.log(JSON.stringify(error))
                    })
                
                this.invoice.items = items
            }
        }
    }
</script>
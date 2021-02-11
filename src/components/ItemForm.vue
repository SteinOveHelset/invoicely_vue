<template>
    <div class="columns">
        <div class="column is-3">
            <div class="field">
                <label>Title</label>
                <div class="control">
                    <input type="text" class="input" v-model="item.title">
                </div>
            </div>
        </div>

        <div class="column is-3">
            <div class="field">
                <label>Price</label>
                <div class="control">
                    <input type="text" class="input" v-model="item.unit_price">
                </div>
            </div>
        </div>

        <div class="column is-2">
            <div class="field">
                <label>Quantity</label>
                <div class="control">
                    <input type="number" class="input" v-model="item.quantity">
                </div>
            </div>
        </div>

        <div class="column is-2">
            <div class="field">
                <label>Vat rate</label>
                <div class="control">
                    <div class="select">
                        <select v-model="item.vat_rate">
                            <option value="0">0%</option>
                            <option value="14">14%</option>
                            <option value="25">25%</option>
                        </select>
                    </div>
                </div>
            </div>
        </div>

        <div class="column is-2">
            <div class="field">
                <label>Gross amount</label>
                <div class="control">
                    <input type="text" class="input" v-bind:value="gross_amount" disabled>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    name: 'ItemForm',
    props: {
        initialItem: Object
    },
    data() {
        return {
            item: this.initialItem
        }
    },
    computed: {
        gross_amount() {
            const unit_price = this.item.unit_price
            const quantity = this.item.quantity
            const vat_rate = this.item.vat_rate

            this.item.net_amount = unit_price * quantity

            this.$emit('updatePrice', this.item)

            return this.item.net_amount + (this.item.net_amount * (vat_rate / 100))
        }
    }
}
</script>
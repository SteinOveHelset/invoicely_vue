<template>
    <div class="page-client">
        <nav class="breadcrumb" aria-label="breadcrumbs">
            <ul>
                <li><router-link to="/dashboard">Dashboard</router-link></li>
                <li><router-link to="/dashboard/clients">Clients</router-link></li>
                <li class="is-active"><router-link :to="{ name: 'Client', params: { id: client.id }}" aria-current="true">{{ client.name }}</router-link></li>
            </ul>
        </nav>
        
        <div class="columns is-multiline">
            <div class="column is-12">
                <h1 class="title">{{ client.name }}</h1>

                <router-link :to="{ name: 'EditClient', params: { id: client.id }}" class="button is-light mt-4">Edit</router-link>
            </div>

            <div class="column is-12">
                <h2 class="subtitle">Contact details</h2>

                <p><strong>{{ client.name }}</strong></p>
                
                <p v-if="client.address1">{{ client.address1 }}</p>
                <p v-if="client.address2">{{ client.address2 }}</p>
                <p v-if="client.zipcode || client.place">{{ client.zipcode }} {{ client.place }}</p>
                <p v-if="client.country">{{ client.country }}</p>
            </div>
        </div>
    </div>
</template>

<script>
import axios from 'axios'

export default {
    name: 'Client',
    data () {
        return {
            client: {}
        }
    },
    mounted() {
        this.getClient()
    },
    methods: {
        getClient() {
            const clientID = this.$route.params.id

            axios
                .get(`/api/v1/clients/${clientID}`)
                .then(response => {
                    this.client = response.data
                })
                .catch(error => {
                    console.log(JSON.stringify(error))
                })
        }
    }
}
</script>
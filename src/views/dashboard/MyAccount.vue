<template>
    <div class="page-my-account">
        <h1 class="title">My account</h1>

        <div class="buttons">
            <router-link to="/dashboard/my-account/edit-team" class="button is-light">Edit team</router-link>
            <button @click="logout()" class="button is-danger">Log out</button>
        </div>
    </div>
</template>

<script>
import axios from 'axios'

export default {
    name: 'MyAccount',
    methods: {
        logout() {
            axios
                .post("/api/v1/token/logout/")
                .then(response => {
                    axios.defaults.headers.common["Authorization"] = ""

                    localStorage.removeItem("token")

                    this.$store.commit('removeToken')

                    this.$router.push('/')
                })
                .catch(error => {
                    if (error.response) {
                        console.log(JSON.stringify(error.response.data))
                    } else if (error.message) {
                        console.log(JSON.stringify(error.message))
                    } else {
                        console.log(JSON.stringify(error))
                    }
                })
        }
    }
}
</script>
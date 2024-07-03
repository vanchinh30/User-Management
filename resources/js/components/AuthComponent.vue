<template>
    <div class="container">
        <div class="row justify-content-center mt-5">
            <div class="col-md-6">
                <div class="card">
                    <div class="card-header">Login</div>

                    <div class="card-body">
                        <form @submit.prevent="login">
                            <div class="form-group">
                                <label for="username">Username</label>
                                <input type="text" class="form-control" id="username" v-model="username" required>
                            </div>

                            <div class="form-group">
                                <label for="password">Password</label>
                                <input type="password" class="form-control" id="password" v-model="password" required>
                            </div>

                            <button type="submit" class="btn btn-primary">Login</button>
                        </form>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import axios from 'axios';

export default {
    data() {
        return {
            username: '',
            password: ''
        };
    },
    methods: {
        login() {
            axios.post('/api/login', {
                username: this.username, 
                password: this.password
            })
            .then(response => {
                localStorage.setItem('token', response.data.token);
                window.location.href = '/user-management';
            })
            .catch(error => {
                console.error('Login error:', error);
            });
        }
    }
};
</script>

<style scoped>
/* Custom styles for the component */
</style>

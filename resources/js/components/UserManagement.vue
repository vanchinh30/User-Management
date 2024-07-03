<template>
    <div class="container-fluid">
        <div class="row mb-3 mt-3 heading">
        <div class="col-md-3">
            <h2>User List</h2>
        </div>
        <button class="btn btn-danger" @click="logout">Logout</button>
        </div>

        <div class="row mb-3 mt-3 filter">
            <div class="col-md-3">
                <div class="input-group">
                    <input type="text" class="form-control" v-model="searchQuery" placeholder="Search by name or email">
                    <div class="input-group-append">
                        <button class="btn btn-primary" type="button" @click="searchUser">Search</button>
                    </div>
                </div>
            </div>
            <div class="col-md-3 text-right">
                <button class="btn btn-success" @click="showCreateModal = true">Create User</button>
            </div>
        </div>

        <!-- Modal for creating/updating users -->
        <div class="modal" tabindex="-1" role="dialog" style="display: block;" v-if="showCreateModal">
            <div class="modal-dialog modal-lg" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title">{{ isEdit ? 'Edit User' : 'Add User' }}</h5>
                        <button type="button" class="close" @click="closeModal">
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                    <form @submit.prevent="isEdit ? updateUser() : createUser()">
                        <div class="modal-body">
                            <div class="form-group">
                                <label for="email">Email:</label>
                                <input type="email" class="form-control" id="email" v-model="form.email" required>
                            </div>
                            <div class="form-group">
                                <label for="name">Name:</label>
                                <input type="text" class="form-control" id="name" v-model="form.name" required>
                            </div>
                            <div class="form-group">
                                <label for="image">Image:</label>
                                <input type="file" class="form-control-file" id="image" @change="onImageChange">
                            </div>
                        </div>
                        <div class="modal-footer">
                            <button type="button" class="btn btn-secondary" @click="closeModal">Close</button>
                            <button type="submit" class="btn btn-primary">{{ isEdit ? 'Update' : 'Create' }}</button>
                        </div>
                    </form>
                </div>
            </div>
        </div>

        <!-- User List -->
        <table class="table">
            <thead>
                <tr>
                    <th>ID</th>
                    <th>Email</th>
                    <th>Name</th>
                    <th>Image</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="user in users" :key="user.id">
                    <td>{{ user.id }}</td>
                    <td>{{ user.email }}</td>
                    <td>{{ user.name }}</td>
                    <td>
                        <img
                            :src="`/storage/images/${user.image}`"
                            alt="User Image"
                            width="50"
                            v-if="user.image"
                        >
                    </td>
                    <td>
                        <button class="btn btn-sm btn-primary" @click="editUser(user)">Edit</button>
                        <button class="btn btn-sm btn-danger" @click="deleteUser(user.id)">Delete</button>
                    </td>
                </tr>
            </tbody>
        </table>
    </div>
</template>

<script>
export default {
    data() {
        return {
            users: [], 
            searchQuery: '',
            form: {
                email: '',
                name: '',
                image: null,
            },
            isEdit: false,
            editUserId: null,
            showCreateModal: false 
        };
    },
    created() {
        this.fetchUsers(); 
    },
    methods: {
        fetchUsers() {
            axios.get('/api/users')
                .then(response => {
                    this.users = response.data;
                })
                .catch(error => {
                    console.error('Error fetching users:', error);
                });
        },

        logout() {
            axios.post('/api/logout')
                .then(response => {
                    console.log(response.data); 
                    window.location.href = '/'; 
                })
                .catch(error => {
                    console.error('Error logging out:', error);
                    alert("Failed to logout.");
                });
        },

        async getImageUrl(imageName) {
        try {
            const response = await axios.get(`/api/user/image/${imageName}`);
            return response.data; 
            } catch (error) {
                console.error('Error fetching image:', error);
                return null; 
            }
        },


        searchUser() {
            axios.get('/api/users/search', {
                params: {
                    query: this.searchQuery
                }
            })
            .then(response => {
                this.users = response.data;
            })
            .catch(error => {
                console.error('Error searching users:', error);
            });
        },

        createUser() {
            let formData = new FormData();
            formData.append('email', this.form.email);
            formData.append('name', this.form.name);
            formData.append('image', this.form.image);
            axios.post('/api/users', formData)
                .then(response => {
                    this.fetchUsers();
                    this.resetForm();
                    this.closeModal();
                    alert("create user successfully!");
                })
                .catch(error => {
                    console.error('Error creating user:', error);
                });
        },

        editUser(user) {
            this.isEdit = true;
            this.editUserId = user.id;
            this.form.email = user.email;
            this.form.name = user.name;
            this.form.image = user.image;
            this.showCreateModal = true; 
        },

        updateUser() {
            let formData = new FormData();
            formData.append('email', this.form.email);
            formData.append('name', this.form.name);
            formData.append('image', this.form.image);
            axios.post(`/api/users/${this.editUserId}`, formData)
                .then(response => {
                    this.fetchUsers();
                    this.resetForm();
                    this.closeModal();
                    alert("Update user successfully!");
                })
                .catch(error => {
                    console.error('Error updating user:', error);
                });
        },

        closeModal() {
            this.showCreateModal = false;
            this.isEdit = false;
            this.editUserId = null;
            this.resetForm();
        },

        deleteUser(userId) {
            if (confirm("Are you sure you want to delete this user?")) {
                axios.delete(`/api/users/${userId}`)
                    .then(response => {
                        this.fetchUsers();
                        alert("User deleted successfully!");
                    })
                    .catch(error => {
                        console.error('Error deleting user:', error);
                        alert("Failed to delete user.");
                    });
                }
        },

        onImageChange(event) {
            this.form.image = event.target.files[0];
        },

        resetForm() {
            this.form.email = '';
            this.form.name = '';
            this.form.image = null;
        }
    }
};
</script>

<style scoped>
/* CSS scoped cho component */
.modal {
    display: block;
    position: fixed;
    z-index: 1;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    overflow: auto;
    background-color: rgba(0,0,0,0.4);
}

.modal-content {
    background-color: #fefefe;
    margin: 15% auto;
    padding: 20px;
    border: 1px solid #888;
    width: 80%;
}

.close {
    color: #aaaaaa;
    float: right;
    font-size: 28px;
    font-weight: bold;
}

.close:hover,
.close:focus {
    color: #000;
    text-decoration: none;
    cursor: pointer;
}

.filter {
    justify-content: end;
}
.heading {
    justify-content: space-between;
}
</style>

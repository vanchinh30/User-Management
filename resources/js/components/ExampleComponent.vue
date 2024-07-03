<template>
    <div>
        <h1>Hello, Vue Component!</h1>
        <p>{{ message }}</p>
    </div>
</template>

<script>
export default {
    data() {
        return {
            message: 'This is a message from Vue component!'
        };
    }
}
</script>

<style scoped>
/* CSS styles */
</style>

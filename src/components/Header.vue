<template>
     <header class='header'>
        <h1>UW Course Planner</h1>
        <div id="nav">
            <router-link to="/">Home</router-link> |
            <router-link to="/about">About</router-link>
        </div>
    </header>
</template>

<script>
export default {
    name: "Header"
    
}
</script>

<style scoped>
.header {
    background-color: #095fe8;
    color: #fff;
}

#nav {
  padding: 10px;
}

#nav a {
  font-weight: bold;
}

#nav a.router-link-exact-active {
  color: #42b983;
}
</style>
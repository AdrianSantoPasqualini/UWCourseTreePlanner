<template>
    <div id="sidebar">
        <DropDown class="dropdown"/>
    </div>
</template>

<script>
import DropDown from "./DropDown";
export default {
    name: "SideBar",
    components: {
        DropDown: DropDown,
    }
}
</script>

<style scoped>
.dropdown {
    margin-top: 5px;
}

#sidebar {
    background-color: #fff;
    width: 30%;
    border-radius: 0.5rem;
    box-shadow: 0 0 2px 2px #ededed;
    padding: 10px;
    margin: 5px;

}
</style>
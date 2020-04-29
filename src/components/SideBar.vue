<template>
    <div id="sidebar">
        <DropDown v-on:selection-change="changeSelection" class="dropdown"/>
    </div>
</template>

<script>
import DropDown from "./DropDown";
export default {
    name: "SideBar",
    components: {
        DropDown: DropDown,
    },
    methods: {
        changeSelection(courses) {
            this.$emit('selection-change', courses);
        }
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
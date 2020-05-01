<template>
    <div id="sidebar">
        <DropDown v-on:selection-change="changeSelection" v-bind:courseList="courseList" class="dropdown"/>
        <CourseChoiceBox class="choice-box"
            v-for="choice in courseOptions"
            v-bind:key="choice.amount"
            v-bind:choice="choice" />
    </div>
</template>

<script>
import DropDown from "./DropDown";
import CourseChoiceBox from '../components/CourseChoiceBox';

export default {
    name: "SideBar",
    components: {
        DropDown: DropDown,
        CourseChoiceBox: CourseChoiceBox,
    },
    props: ["courseList", "courseOptions"],
    methods: {
        changeSelection(courses) {
            this.$emit('selection-change', courses);
        }
    }
}
</script>

<style scoped>
.dropdown {
    width: 100%;
    margin-top: 5px;
    margin-bottom: 5px;
}
.choice-box {
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
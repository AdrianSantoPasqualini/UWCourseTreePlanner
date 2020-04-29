<template>
    <div class="homePanel">
        <SideBar v-on:selection-change="changeSelection"/>
        <Chart />
    </div>
</template>

<script>
import Chart from '../components/Chart';
import SideBar from '../components/SideBar';

export default {
    name: 'Home',
    components: {
        Chart: Chart,
        SideBar: SideBar,
    },
    data() {
        return {
            selectedCourses: []
        }
    },
    methods: {
        changeSelection(courses) {
            this.selectedCourses = courses;
        }
    }
}
</script>

<style scoped>
.homePanel {
    height: 100%;
    display: flex;
}
</style>
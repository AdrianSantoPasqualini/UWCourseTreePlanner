<template>
    <div class="homePanel">
        <SideBar v-on:generate-tree="buildChartData" v-bind:courseList="courseList"/>
        <Chart v-bind:chartData="{}"/>
    </div>
</template>

<script>
import Chart from '../components/Chart';
import SideBar from '../components/SideBar';
import TrieSearch from "trie-search";
import axios from "axios"
const UW_API_KEY = "ad7c8daa89f123ff2f7ee0c0a4678694";

export default {
    name: 'Home',
    components: {
        Chart: Chart,
        SideBar: SideBar,
    },

    data() {
        return {
            courseList: new TrieSearch(),
        }
    },

    methods: {
        buildChartData(courseData, chosenCourses) {
            console.log(courseData);
            console.log(chosenCourses)
        }
    },

    created() {
        var courseTrie = new TrieSearch(['name']);
        axios.get("https://api.uwaterloo.ca/v2/courses.json?key=" + UW_API_KEY)
        .then(response => {
            var id = 0;
            const courses = response.data.data.map(course => {
                id++;
                return {
                    subject: course.subject,
                    catalog_number: course.catalog_number,
                    name: course.subject + course.catalog_number,
                    course_id: course.course_id,
                    id: id,
                    title: course.title,
                }
            })
            courseTrie.addAll(courses);
            this.courseList = courseTrie;
        })
        .catch(err => console.log(err));
    }
}
</script>

<style scoped>
.homePanel {
    height: 100%;
    display: flex;
}
</style>
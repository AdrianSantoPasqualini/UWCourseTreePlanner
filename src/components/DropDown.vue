<template>
    <div class="dropdown-container">
        <Multiselect v-model="value" :options="filteredCourseList" v-on:search-change="filterCourses"
                        label="name" track-by="name">
        </Multiselect>
    </div>
        
</template>

<script>
import TrieSearch from "trie-search";
import Multiselect from 'vue-multiselect'

const UW_API_KEY = "ad7c8daa89f123ff2f7ee0c0a4678694";


export default {
    name: "DropDown",
    components: {
        Multiselect
    },

    data() {
        return {
            value: null,
            courseList: new TrieSearch(),
            filteredCourseList: [],
            searchText: "",
        }
    },

    methods: {
        filterCourses(event) {
            console.log(event);
            this.searchText = event.replace(/\s/g, "");
            this.filteredCourseList = this.courseList.get(this.searchText);
        },
        
        addCourse(course) {
            console.log(course);
        }
    },

    created() {
        var courseTrie = new TrieSearch(['name']);
        fetch("https://api.uwaterloo.ca/v2/courses.json?key=" + UW_API_KEY)
        .then(response => response.json())
        .then(response => {
            var id = 0;
            const courses = response.data.map(course => {
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

<style src="vue-multiselect/dist/vue-multiselect.min.css"> </style>
<style scoped>

</style>
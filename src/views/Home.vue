<template>
    <div class="homePanel">
        <SideBar v-on:selection-change="changeSelection" v-bind:courseOptions="courseOptions" v-bind:courseList="courseList"/>
        <Chart v-bind:selectedCourses="selectedCourses" v-bind:courseList="courseList"/>
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
            selectedCourses: [],
            courseList: new TrieSearch(),
            courseOptions: [],
            seenCourses: new TrieSearch(['name']),
            courseData: [],
        }
    },
    methods: {
        async changeSelection(newCourses) {
            this.courseData = [];
            for (const course of newCourses) {
                await this.generateCourseTree(course);
            }
            console.log(this.courseOptions)
        },

        async generateCourseTree(course) {
            var newCourse = {
                    name: course.subject + course.catalog_number,
                    rawPrerequisites: "No Prerequisites",
                    parsedPrerequisites: [],
                    value: 1,
                    courseInfo: course,
            }
            this.seenCourses.addAll([newCourse]);
            await axios.get("https://api.uwaterloo.ca/v2/courses/"+course.subject+"/"+course.catalog_number+"/prerequisites.json?key="+UW_API_KEY)
            .then(async response => {
                if (response.data.data.prerequisites_parsed) {
                    newCourse.rawPrerequisites = response.data.data.prerequisites;
                    newCourse.parsedPrerequisites = response.data.data.prerequisites_parsed;
                    await this.generatePrereqChildrenFromList(response.data.data.prerequisites_parsed, course.subject + course.catalog_number);
                }
                this.courseData.push(newCourse);
            });
            return newCourse;
        },

        async generatePrereqChildrenFromList(prerequisites_parsed, parent) {
            if (typeof prerequisites_parsed[0] === "number") {
                const newChoice = {
                    name: parent,
                    amount: prerequisites_parsed[0],
                    options: prerequisites_parsed.slice(1).map(course => {
                        return {selected: false, name: course}})
                }
                this.courseOptions.push(newChoice);
                await this.generatePrereqChildrenSingleTerm(prerequisites_parsed, parent);
            } else {
                for (const prereq of prerequisites_parsed) {
                    await this.generatePrereqChildrenSingleTerm(prereq, parent);
                }
            }
        },

        async generatePrereqChildrenSingleTerm(prereqTerm, parent) {
            if (typeof prereqTerm === "string") {
                if (!this.seenCourses.get(prereqTerm).length) {
                    await this.generateCourseTree(this.courseList.get(prereqTerm)[0])
                    .catch(err => console.log("Couldnt find course: ", err));
                }
            } else {
                const newChoice = {
                    name: parent,
                    amount: prereqTerm[0],
                    options: prereqTerm.slice(1).map(course => {
                        return {selected: false, name: course}})
                }
                this.courseOptions.push(newChoice);
                for (let i = 1; i < prereqTerm.length; i++) {
                    await this.generatePrereqChildrenSingleTerm(prereqTerm[i], parent)
                }
            }
        },
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
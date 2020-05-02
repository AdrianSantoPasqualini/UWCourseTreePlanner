<template>
    <div class="homePanel">
        <SideBar v-on:generate-tree="buildChartData" v-bind:courseList="courseList"/>
        <Chart v-bind:chartData="chartData"/>
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
            courseData: [],
            chosenCourses: new TrieSearch(['courseInfo', 'name']),
            chartData: [],
            displayedCourses: new TrieSearch(['name']),
        }
    },

    methods: {
        buildChartData(courseData, chosenCourses) {
            this.chartData = [];
            this.displayedCourses = new TrieSearch(['name']);
            this.courseData = courseData;
            this.chosenCourses = chosenCourses;
            for (var course of this.courseData) {
                const chosen = this.chosenCourses.get(course.name);
                if (chosen.length > 0 && chosen[0].choosers.includes("User")) {
                    this.chartData.push(this.generateCourseTree(course, course.name));
                }
            }
        },

        generateCourseTree(course, parent) {
            this.displayedCourses.addAll([course]);
            var newCourse = {
                    name: course.name,
                    rawPrerequisites: course.rawPrerequisites,
                    value: 1,
                    courseData: course,
                    children: [],
                    links: [],
            }
            if (course.parsedPrerequisites && course.parsedPrerequisites.length > 0) {
                var prereqs = this.generatePrereqChildrenFromList(course.parsedPrerequisites, parent);
                newCourse.children = newCourse.children.concat(prereqs.prereqChildren);
                newCourse.links = newCourse.links.concat(prereqs.prereqLinks);
            }
            return newCourse;
        },

        generatePrereqChildrenFromList(prerequisites_parsed, parent) {
            var linksAndChildren = {
                prereqChildren: [],
                prereqLinks: [],
            }
            if (typeof prerequisites_parsed[0] === "number") {
                return this.generatePrereqChildrenSingleTerm(prerequisites_parsed, parent);
            } else {
                for (const prereq of prerequisites_parsed) {
                    const prereqs = this.generatePrereqChildrenSingleTerm(prereq, parent)
                    linksAndChildren.prereqChildren = linksAndChildren.prereqChildren.concat(prereqs.prereqChildren);
                    linksAndChildren.prereqLinks = linksAndChildren.prereqLinks.concat(prereqs.prereqLinks);
                }
            }
            return linksAndChildren;
        },
        generatePrereqChildrenSingleTerm(prereqTerm, parent) {
            var linksAndChildren = {
                prereqChildren: [],
                prereqLinks: [],
            }
            if (typeof prereqTerm === "string") {
                if (!this.displayedCourses.get(prereqTerm).length) {
                    var newChild = this.generateCourseTree(this.chosenCourses.get(prereqTerm)[0].courseInfo, prereqTerm);
                    linksAndChildren.prereqChildren.push(newChild);
                } else {
                    linksAndChildren.prereqLinks.push(prereqTerm);
                }
            } else {
                for (let i = 1; i < prereqTerm.length; i++) {
                    const chosen = this.chosenCourses.get(prereqTerm[i]);
                    if (chosen.length > 0 && chosen[0].choosers.includes(parent)) {
                        const prereqs = this.generatePrereqChildrenSingleTerm(prereqTerm[i], parent);
                        linksAndChildren.prereqChildren = linksAndChildren.prereqChildren.concat(prereqs.prereqChildren);
                        linksAndChildren.prereqLinks = linksAndChildren.prereqLinks.concat(prereqs.prereqLinks);
                    }
                }
            }
            return linksAndChildren;
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
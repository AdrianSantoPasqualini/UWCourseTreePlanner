<template>
    <div class="homePanel">
        <SideBar id="sidebar" v-on:generate-tree="buildChartData" v-bind:courseList="courseList"/>
        <Chart v-bind:chartData="chartData"/>
    </div>
</template>

<script>
import Chart from '../components/Chart';
import SideBar from '../components/SideBar';
import TrieSearch from "trie-search";
import * as am4core from "@amcharts/amcharts4/core";
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
            levelData: [],
        }
    },

    methods: {
        levelToColor(level) {
            level = level % 4
            switch (level) {
                case 1:
                    return am4core.color("#F42B03");
                case 2:
                    return am4core.color("#D84A05");
                case 3:
                    return am4core.color("#EC7505");
                case 0:
                    return am4core.color("#E89005");
            }
        },

        buildChartData(courseData, chosenCourses) {
            this.chartData = [];
            this.displayedCourses = new TrieSearch(['name']);
            this.courseData = courseData;
            this.chosenCourses = chosenCourses;
            for (var course of this.courseData) {
                const chosen = this.chosenCourses.get(course.name);
                if (chosen.length > 0 && chosen[0].choosers.includes("User")) {
                    this.chartData.push(this.generateCourseTree(course, course.name, 1));
                }
            }
        },

        generateCourseTree(course, parent, level) {
            this.displayedCourses.addAll([course]);
            var newCourse = {
                    name: course.name,
                    rawPrerequisites: course.rawPrerequisites,
                    courseData: course,
                    children: [],
                    links: [],
                    level: level,
                    color: this.levelToColor(level)
            }
            if (this.levelData[level]) {
                this.levelData[level]++;
            } else {
                this.levelData[level] = 1;
            }
            if (course.parsedPrerequisites && course.parsedPrerequisites.length > 0) {
                var prereqs = this.generatePrereqChildrenFromList(course.parsedPrerequisites, parent, level+1);
                newCourse.children = newCourse.children.concat(prereqs.prereqChildren);
                newCourse.links = newCourse.links.concat(prereqs.prereqLinks);
            } else {
                newCourse.value = 5;
            }
            return newCourse;
        },

        generatePrereqChildrenFromList(prerequisites_parsed, parent, level) {
            var linksAndChildren = {
                prereqChildren: [],
                prereqLinks: [],
            }
            if (typeof prerequisites_parsed[0] === "number") {
                return this.generatePrereqChildrenSingleTerm(prerequisites_parsed, parent, level);
            } else {
                for (const prereq of prerequisites_parsed) {
                    const prereqs = this.generatePrereqChildrenSingleTerm(prereq, parent, level)
                    linksAndChildren.prereqChildren = linksAndChildren.prereqChildren.concat(prereqs.prereqChildren);
                    linksAndChildren.prereqLinks = linksAndChildren.prereqLinks.concat(prereqs.prereqLinks);
                }
            }
            return linksAndChildren;
        },
        generatePrereqChildrenSingleTerm(prereqTerm, parent, level) {
            var linksAndChildren = {
                prereqChildren: [],
                prereqLinks: [],
            }
            if (typeof prereqTerm === "string") {
                //if (!this.displayedCourses.get(prereqTerm).length) {
                    var newChild = this.generateCourseTree(this.chosenCourses.get(prereqTerm)[0].courseInfo, prereqTerm, level);
                    linksAndChildren.prereqChildren.push(newChild);
                //} else {
                  //  linksAndChildren.prereqChildren.push(prereqTerm);
                //}
            } else {
                for (let i = 1; i < prereqTerm.length; i++) {
                    const chosen = this.chosenCourses.get(prereqTerm[i]);
                    if (chosen.length > 0 && chosen[0].choosers.includes(parent)) {
                        const prereqs = this.generatePrereqChildrenSingleTerm(prereqTerm[i], parent, level);
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
#sidebar {
    background-color: #fff;
    width: 100%;
    height: 99%;
    border-radius: 0.5rem;
    box-shadow: 0 0 2px 2px #ededed;
    overflow-y: scroll;
    margin: 0.5%;
}

</style>
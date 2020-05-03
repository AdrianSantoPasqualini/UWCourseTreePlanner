<template>
    <div>
        <div class="sidebar-container">
            <DropDown
                v-on:add-selection="addSelection"
                v-on:remove-selection="removeSelection"
                v-bind:courseList="courseList" class="dropdown"/>
            <!--
            <div v-for="course in courseData" v-bind:key="course.id">
                <div v-if="chosenCourses.get(course.name)[0].chosenBy > 0">
                    <CourseChoiceBox class="choice-box"
                            v-if="typeof course.parsedPrerequisites[0] === 'number'"
                            v-on:choice-made="updateChosenCourses"
                            v-bind:choice="{parent: course.name, options: course.parsedPrerequisites}"/>
                    <div v-else v-for="prereq in course.parsedPrerequisites" v-bind:key="course.id + prereq">
                        <CourseChoiceBox class="choice-box"
                            v-if="Array.isArray(prereq)"
                            v-on:choice-made="updateChosenCourses"
                            v-bind:choice="{parent: course.name, options: prereq}"/>
                    </div>
                </div>
            </div>
            -->
            <div class="course-choices">
                    <CourseBox class="course-box"
                        v-for="course in this.courseBoxList" v-bind:key="course.id"
                        v-on:choice-made="updateChosenCourses" v-bind:course="course" />
            </div>
            <button class="submitChoices" v-on:click="$emit('generate-tree', courseData, chosenCourses)">
                Generate Prequisite Tree
            </button>
        </div>
    </div>
</template>

<script>
import DropDown from "./DropDown";
import CourseBox from '../components/CourseBox';
import axios from "axios"
import TrieSearch from "trie-search";
const UW_API_KEY = "ad7c8daa89f123ff2f7ee0c0a4678694";

export default {
    name: "SideBar",
    components: {
        DropDown: DropDown,
        CourseBox: CourseBox,
    },
    props: ["courseList"],
    data() {
        return {
            courseData: [],
            courseTrie: new TrieSearch(['name']),
            chosenCourses: new TrieSearch([['courseInfo', 'name']]),
        }
    },
    computed: {
        courseBoxList: function(){
            return this.courseData.filter((course) => {
                return this.chosenCourses.get(course.name)[0].chosenBy > 0 && course.parsedPrerequisites.length > 0;
            });
        }
    },
    methods: {
        async addSelection(course) {
            this.updateChosenCourses({parent: "User", selected: course.name, unselected: 0})
        },
        async removeSelection(course) {
            this.updateChosenCourses({parent: "User", selected: 0, unselected: course.name})
        },

        async updateChosenCourses(change) {
            if (change.selected && !this.courseTrie.get(change.selected).length) {
                await this.generateCourseData(this.courseList.get(change.selected)[0], change.parent);
            } else if (change.selected && this.chosenCourses.get(change.selected).length > 0) {
                this.chosenCourses.get(change.selected)[0].chosenBy++;
                this.chosenCourses.get(change.selected)[0].choosers.push(change.parent);
                if (this.chosenCourses.get(change.selected)[0].chosenBy === 1) {
                    this.reselectChildren(change.selected, this.chosenCourses.get(change.selected)[0].courseInfo.parsedPrerequisites);
                }
            }
            if (change.unselected && this.chosenCourses.get(change.unselected).length > 0) {
                this.chosenCourses.get(change.unselected)[0].chosenBy--;
                this.chosenCourses.get(change.unselected)[0].choosers = this.chosenCourses.get(change.unselected)[0].choosers.filter(chooser => {
                            return chooser !== change.parent;
                });
                if (this.chosenCourses.get(change.unselected)[0].chosenBy <= 0) {
                    this.deselectChildren(change.unselected, this.chosenCourses.get(change.unselected)[0].courseInfo.parsedPrerequisites);
                }
            }
        },

        reselectChildren(parent, prereqs) {
            if (typeof prereqs[0] === "number") {
                return;
            }
            for (const term of prereqs) {
                if (typeof term === "string") {
                    this.chosenCourses.get(term)[0].chosenBy++;
                    this.chosenCourses.get(term)[0].choosers.push(parent);
                    if (this.chosenCourses.get(term)[0].chosenBy <= 0) {
                        this.reselectChildren(term, this.chosenCourses.get(term)[0].courseInfo.parsedPrerequisites);
                    }
                }
            }
        },

        deselectChildren(parent, prereqs) {
            for (const term of prereqs) {
                if (typeof term === "string") {
                    if (this.chosenCourses.get(term).length > 0 && this.chosenCourses.get(term)[0].choosers.includes(parent)) {
                        this.chosenCourses.get(term)[0].chosenBy--;
                        this.chosenCourses.get(term)[0].choosers = this.chosenCourses.get(term)[0].choosers.filter(chooser => {
                            return chooser !== parent;
                        });
                        if (this.chosenCourses.get(term)[0].chosenBy <= 0) {
                            this.deselectChildren(term, this.chosenCourses.get(term)[0].courseInfo.parsedPrerequisites);
                        }
                    }
                } else if (Array.isArray(term)) {
                    this.deselectChildren(parent, term);
                }
            }
        },

        async generateCourseData(course, parent) {
            var newCourse = {
                    name: course.subject + course.catalog_number,
                    rawPrerequisites: "No Prerequisites",
                    parsedPrerequisites: [],
                    value: 1,
                    courseInfo: course,
            }
            this.chosenCourses.add({courseInfo: newCourse, chosenBy: 1, choosers: [parent]});
            this.courseTrie.add(newCourse);
            await axios.get("https://api.uwaterloo.ca/v2/courses/"+course.subject+"/"+course.catalog_number+"/prerequisites.json?key="+UW_API_KEY)
            .then(async response => {
                if (response.data.data.prerequisites_parsed) {
                    newCourse.rawPrerequisites = response.data.data.prerequisites;
                    newCourse.parsedPrerequisites = response.data.data.prerequisites_parsed;
                }
                this.courseData.push(newCourse);
            });
            if (Array.isArray(newCourse.parsedPrerequisites[0]) && newCourse.parsedPrerequisites.length === 1) {
                newCourse.parsedPrerequisites = newCourse.parsedPrerequisites[0];
            }
            if (newCourse.parsedPrerequisites[0] === 1 && newCourse.parsedPrerequisites.length === 2) {
                if (Array.isArray(newCourse.parsedPrerequisites[1])) {
                    newCourse.parsedPrerequisites = [1].concat(newCourse.parsedPrerequisites[1]);
                }
            } else if (newCourse.parsedPrerequisites[0] === 1) {
                var newPrereqs = [1];
                for (const prereqCourse of newCourse.parsedPrerequisites.slice(1)) {
                    if (Array.isArray(prereqCourse) && prereqCourse[0] === 1) {
                        newPrereqs = newPrereqs.concat(prereqCourse.slice(1));
                    } else {
                        newPrereqs.push(prereqCourse);
                    }
                }
                newCourse.parsedPrerequisites = newPrereqs;
            } else {
                for (const prereqCourse of newCourse.parsedPrerequisites) {
                    if (typeof prereqCourse === "string") {
                        if (!this.courseTrie.get(prereqCourse).length) {
                            await this.generateCourseData(this.courseList.get(prereqCourse)[0], course.name);
                        } else {
                            this.chosenCourses.get(prereqCourse)[0].choosers.push(course.name);
                            this.chosenCourses.get(prereqCourse)[0].chosenBy++;
                        }
                    }
                }
            }
            return newCourse;
        },

    },
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
.sidebar-container {
    padding: 10px;
}
.submitChoices {
    border-radius: 0.5rem;
    border: 1px solid white;
    padding: 5px;
    margin: 5px;
    flex-grow: 1;
    font-size: 14px;
    background-color: #26C281;
    color: white;
    
}
.submitChoices:focus {
   outline: none;
   box-shadow: none;
}

.course-choices {
    display: flex;
    flex-wrap: wrap;
}
.course-box {
    flex-grow: 1;
}
</style>
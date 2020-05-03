<template>
    <div class="course-box">
        <strong class="course-box-title">{{course.name + " Prerequisites:"}} </strong>
        <div class="prereq-container">
            <div class="prereq" v-for="prereq in prereqs" v-bind:key="prereq[1]" >
                <CourseChoiceBox class="choice-box"
                    v-if="typeof prereq[0] === 'number'"
                    v-on:choice-made="updateChosenCourses"
                    v-bind:choice="{parent: course.name, amount: prereq[0], options: prereq.slice(1)}"/>
                <div v-else class="non-optional-prereq">
                    {{prereq}}
                </div>
            </div>
        </div>
    </div>
    
</template>

<script>
import CourseChoiceBox from '../components/CourseChoiceBox';

export default {
    name: "CourseBox",
    components: {
        CourseChoiceBox: CourseChoiceBox,
    },
    props: ["course"],
    methods: {
        updateChosenCourses(change) {
            this.$emit("choice-made", change)
        }
    },
    computed: {
        prereqs: function() {
            if (typeof this.course.parsedPrerequisites[0] === "number") {
                return [this.course.parsedPrerequisites]
            } else {
                return this.course.parsedPrerequisites;
            }
        }
    }
}
</script>

<style scoped>
.course-box {
    max-width: 33%;
    margin: 0.5%;
    border: 1px solid #e6e6e6;
    border-radius: 0.5rem;
    display: flex;
    flex-direction: column;
    justify-content: center;
}
.choice-box {
    width: 100%;
}
.course-box-title {
    margin-top: 2.5%;
}
.non-optional-prereq {
    border-radius: 0.5rem;
    border: 1px solid white;
    padding: 5px;
    margin: 5px;
    flex-grow: 1;
    font-size: 14px;
    background-color: #26C281;
    color: white;
}

.prereq-container {
    display: flex;
    flex-wrap: wrap;
    padding: 5px;
    height: 99%
}
.prereq {
    flex-grow: 1;
    margin: 5px;
    line-height: 100%;
    display: flex;
    justify-content: center;
    align-items: center; 
}

</style>
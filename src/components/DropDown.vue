<template>
    <div class="dropdown-container">
        <Multiselect
            v-model="value"
            :options="filteredCourseList"
            v-on:search-change="filterCourses"
            v-on:select="addSelection"
            v-on:remove="removeSelection"
            label="name" track-by="name"
            :close-on-select="false"
            :clear-on-select="false"
            :multiple="true"
            :internal-search="false"
            :options-limit="100"
            :hide-selected="true"
            selectLabel=""
            placeholder="Search for Courses">
            <template slot="noOptions">Enter a course code (e.g. CS135, AFM101, etc.)</template>
            <template slot="noResult">No unselected courses match your search.</template>
            <template slot="option" slot-scope="props">
                <div class="option__title"><strong>{{ props.option.name + ":"}}</strong></div>
                <div class="option__small">{{ props.option.title }}</div>
            </template>
        </Multiselect>
    </div>
        
</template>

<script>
import Multiselect from 'vue-multiselect'

export default {
    name: "DropDown",
    components: {
        Multiselect
    },
    props: ["courseList"],
    data() {
        return {
            value: null,
            filteredCourseList: [],
            searchText: "",
        }
    },

    methods: {
        filterCourses(event) {
            this.searchText = event.replace(/\s/g, "");
            this.filteredCourseList = this.courseList.get(this.searchText);
        },
        addSelection(course) {
            this.$emit('add-selection', course);
        },
        removeSelection(course) {
            this.$emit('remove-selection', course);
        }
    },
}
</script>

<style src="vue-multiselect/dist/vue-multiselect.min.css"> </style>
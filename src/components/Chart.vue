<template>
  <div class="chart" id="chartdiv">
  </div>
</template>


<script>
import * as am4core from "@amcharts/amcharts4/core";
import * as am4plugins_forceDirected from "@amcharts/amcharts4/plugins/forceDirected"; 
import am4themes_animated from "@amcharts/amcharts4/themes/animated";
import axios from "axios"
am4core.useTheme(am4themes_animated);

const UW_API_KEY = "ad7c8daa89f123ff2f7ee0c0a4678694";

export default {
  name: 'Chart',
  data() {
    return {
      chartData: []
    }
  },
  props: ["selectedCourses", "courseList"],
  
  methods: {
    
    updateChart() {
      if (this.chart) {
        this.chart.dispose();
      }
      var chart = am4core.create("chartdiv", am4plugins_forceDirected.ForceDirectedTree);
      var series = chart.series.push(new am4plugins_forceDirected.ForceDirectedSeries());
      series.data = this.chartData;
      // Set up data fields
      series.dataFields.value = "value";
      series.dataFields.name = "name";
      series.dataFields.children = "children";
      series.dataFields.id = "name";
      series.dataFields.linkWith = "link";
      // Add labels
      series.nodes.template.label.text = "[bold]{name}[/]";
      series.nodes.template.tooltipText = "{rawPrerequisites}";
      series.fontSize = 10;
      series.minRadius = 15;
      series.maxRadius = 40;
      series.centerStrength = 0.5;
    },

    numberToString(num) {
      switch (num) {
        case 1:
          return "One Of:";
        case 2:
          return "Two Of:";
        case 3:
          return "Three Of:";
        case 4:
          return "Four Of:";
        case 5:
          return "Five Of:";
        case 6:
          return "Six Of:";
        case 7:
          return "Seven Of:";
        case 8:
          return "Eight Of:";
        default:
          return "Num Too High";
      }
    },

    async generateCourseTree(course) {
      var newCourse = {
            name: course.subject + course.catalog_number,
            rawPrerequisites: "No Prerequisites",
            value: 1,
            courseData: course,
            children: [],
      }
      await axios.get("https://api.uwaterloo.ca/v2/courses/"+course.subject+"/"+course.catalog_number+"/prerequisites.json?key="+UW_API_KEY)
      .then(async response => {
          if (response.data.data.prerequisites_parsed) {
            newCourse.rawPrerequisites = response.data.data.prerequisites;
            newCourse.children = newCourse.children.concat(await this.generatePrereqChildrenFromList(response.data.data.prerequisites_parsed));
          }          
      });
      return newCourse;
    },

    async generatePrereqChildrenFromList(prerequisites_parsed) {
      var prereqChildren = [];

      if (typeof prerequisites_parsed[0] === "number") {
        return this.generatePrereqChildrenSingleTerm(prerequisites_parsed);
      } else {
        for (const prereq of prerequisites_parsed) {
          prereqChildren = [...prereqChildren, await this.generatePrereqChildrenSingleTerm(prereq)];
        }
      }
      return prereqChildren;
    },

    async generatePrereqChildrenSingleTerm(prereqTerm) {
      if (typeof prereqTerm === "string") {
        return await this.generateCourseTree(this.courseList.get(prereqTerm)[0])
        .catch(err => console.log("Couldnt find course: ", err));
      } else {
        let courseOptions = {
          name: this.numberToString(prereqTerm[0]),
          rawPrerequisites: "Choose one of these courses:",
          value: 1,
          children: []
        }
        for (let i = 1; i < prereqTerm.length; i++) {
          const newChild = await this.generatePrereqChildrenSingleTerm(prereqTerm[i])
          courseOptions.children = [...courseOptions.children, newChild];
        }
        return courseOptions;
      }
    },
  },

  watch: {
    selectedCourses: async function(newCourses) {
      let newChartData = [];
      for (const course of newCourses) {
          var newCourse = await this.generateCourseTree(course);
          newChartData = [...newChartData, newCourse]
      }
      this.chartData = newChartData;
      await this.updateChart();
      
      return newCourses;
    }
  },
  
  mounted() {
    // Create chart
    var chart = am4core.create("chartdiv", am4plugins_forceDirected.ForceDirectedTree);
    // Create series
    chart.series.push(new am4plugins_forceDirected.ForceDirectedSeries())
  },

  beforeDestroy() {
    if (this.chart) {
      this.chart.dispose();
    }
  }
}
</script>

<style scoped>
.chart {
  margin-left: 10px;
  width: 70%;
}
</style>

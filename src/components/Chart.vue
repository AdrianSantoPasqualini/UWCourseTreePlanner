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
      series.nodes.template.label.text = "{name}";
      series.nodes.template.tooltipText = "{courseData.title}";
      series.fontSize = 10;
      series.minRadius = 15;
      series.maxRadius = 40;
      series.centerStrength = 0.5;
    },

    async generateCourseTree(course) {
      var newCourse = {
            name: course.subject + course.catalog_number,
            value: 1,
            courseData: course,
            children: [],
      }
      await axios.get("https://api.uwaterloo.ca/v2/courses/"+course.subject+"/"+course.catalog_number+"/prerequisites.json?key="+UW_API_KEY)
      .then(async response => {
          if (response.data.data.prerequisites_parsed) {
            newCourse.children = newCourse.children.concat(await this.generatePrereqChildren(response.data.data.prerequisites_parsed));
            console.log(newCourse.children);
          }          
      });
      return newCourse;
    },

    async generatePrereqChildren(prerequisites_parsed) {
      var prereqChildren = [];
      for (const prereq of prerequisites_parsed) {
        if (typeof prereq === "string") {
          prereqChildren = [...prereqChildren, await this.generateCourseTree(this.courseList.get(prereq)[0])];
        }
      }
      console.log(prereqChildren)
      return prereqChildren;
    }
  },



  watch: {
    selectedCourses: async function(newCourses) {
      console.log("Start");
      let newChartData = [];
      for (const course of newCourses) {
          var newCourse = await this.generateCourseTree(course);
          newChartData = [...newChartData, newCourse]
      }
      
      console.log("End");
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

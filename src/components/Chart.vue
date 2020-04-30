<template>
  <div class="chart" id="chartdiv">
  </div>
</template>


<script>
import * as am4core from "@amcharts/amcharts4/core";
import * as am4plugins_forceDirected from "@amcharts/amcharts4/plugins/forceDirected"; 
import am4themes_animated from "@amcharts/amcharts4/themes/animated";
import TrieSearch from "trie-search";
import axios from "axios"
am4core.useTheme(am4themes_animated);

const UW_API_KEY = "ad7c8daa89f123ff2f7ee0c0a4678694";

export default {
  name: 'Chart',
  data() {
    return {
      chartData: [],
      displayedCourses: new TrieSearch(['name']),
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
      series.dataFields.linkWith = "links";
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
      this.displayedCourses.addAll([course]);
      var newCourse = {
            name: course.subject + course.catalog_number,
            rawPrerequisites: "No Prerequisites",
            value: 1,
            courseData: course,
            children: [],
            links: [],
      }
      await axios.get("https://api.uwaterloo.ca/v2/courses/"+course.subject+"/"+course.catalog_number+"/prerequisites.json?key="+UW_API_KEY)
      .then(async response => {
          if (response.data.data.prerequisites_parsed) {
            newCourse.rawPrerequisites = response.data.data.prerequisites;
            var prereqs = await this.generatePrereqChildrenFromList(response.data.data.prerequisites_parsed);
            newCourse.children = newCourse.children.concat(prereqs.prereqChildren);
            newCourse.links = newCourse.links.concat(prereqs.prereqLinks);
          }          
      });
      return newCourse;
    },

    async generatePrereqChildrenFromList(prerequisites_parsed) {
      var linksAndChildren = {
        prereqChildren: [],
        prereqLinks: [],
      }
      if (typeof prerequisites_parsed[0] === "number") {
        return await this.generatePrereqChildrenSingleTerm(prerequisites_parsed);
      } else {
        for (const prereq of prerequisites_parsed) {
          const prereqs = await this.generatePrereqChildrenSingleTerm(prereq)
          linksAndChildren.prereqChildren = linksAndChildren.prereqChildren.concat(prereqs.prereqChildren);
          linksAndChildren.prereqLinks = linksAndChildren.prereqLinks.concat(prereqs.prereqLinks);
        }
      }
      return linksAndChildren;
    },

    async generatePrereqChildrenSingleTerm(prereqTerm) {
      var linksAndChildren = {
        prereqChildren: [],
        prereqLinks: [],
      }
      if (typeof prereqTerm === "string") {
        console.log(this.displayedCourses.get(prereqTerm));
        if (!this.displayedCourses.get(prereqTerm).length) {
          var newChild = await this.generateCourseTree(this.courseList.get(prereqTerm)[0])
          .catch(err => console.log("Couldnt find course: ", err));
          linksAndChildren.prereqChildren.push(newChild);
        } else {
          linksAndChildren.prereqLinks.push(prereqTerm);
        }
      } else {
        let courseOptions = {
          name: this.numberToString(prereqTerm[0]),
          rawPrerequisites: "Choose one of these courses:",
          value: 1,
          children: [],
          links: []
        }
        for (let i = 1; i < prereqTerm.length; i++) {
          const prereqs = await this.generatePrereqChildrenSingleTerm(prereqTerm[i])
          courseOptions.children = courseOptions.children.concat(prereqs.prereqChildren);
          courseOptions.links = courseOptions.links.concat(prereqs.prereqLinks);
        }
        linksAndChildren.prereqChildren.push(courseOptions);
      }
      return linksAndChildren;
    },
  },

  watch: {
    selectedCourses: async function(newCourses) {
      this.displayedCourses = new TrieSearch(['name']);
      let newChartData = [];
      for (const course of newCourses) {
          var newCourse = await this.generateCourseTree(course);
          newChartData.push(newCourse);
      }
      console.log("END", this.displayedCourses.get("p"));
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

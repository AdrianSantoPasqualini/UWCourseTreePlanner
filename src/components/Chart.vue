<template>
  <div class="chart" id="chartdiv">
  </div>
</template>


<script>
import * as am4core from "@amcharts/amcharts4/core";
import * as am4plugins_forceDirected from "@amcharts/amcharts4/plugins/forceDirected"; 
import am4themes_animated from "@amcharts/amcharts4/themes/animated";
import TrieSearch from "trie-search";
am4core.useTheme(am4themes_animated);

export default {
  name: 'Chart',
  data() {
    return {
      displayedCourses: new TrieSearch(['name']),
    }
  },
  props: ["courseData"],
  
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
  },

  watch: {
    /*
    chartData: async function(newCourses) {
      console.log(newCourses)
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
    */
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

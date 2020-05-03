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
  props: ["chartData", "chosenCourses", "levelData"],
  
  methods: {
    
    updateChart() {
      var chart = am4core.create("chartdiv", am4plugins_forceDirected.ForceDirectedTree);
      var series = chart.series.push(new am4plugins_forceDirected.ForceDirectedSeries());
      series.data = this.chartData;
      // Set up data fields
      series.dataFields.value = "value";
      series.dataFields.name = "name";
      series.dataFields.children = "children";
      series.dataFields.id = "name";
      series.dataFields.color = "color";
      series.colors.list = [
        am4core.color("#F42B03"),
        am4core.color("#EC7505"),
        am4core.color("#E89005")
      ];
      series.manyBodyStrength = -25;
      //series.centerStrength = 0;
      //series.links.template.strength = 10
      //series.dataFields.linkWith = "links";
      //series.dataFields.fixed = "fixed";
      //series.nodes.template.propertyFields.x = "x";
      //series.nodes.template.propertyFields.y = "y";
      // Add labels
      series.nodes.template.label.text = "[bold]{name}[/]";
      series.nodes.template.tooltipText = "{rawPrerequisites}";
      series.nodes.template.adapter.add("fontSize", function(curr, target) {
        if (target.dataItem.value < 10) {
          return 10;
        }
        return target.dataItem.value;
      });
      series.minRadius = 30;
      series.maxRadius = 60;
      series.maxLevels = 3;
    },
  },

  watch: {
    chartData: async function(newData) {
      this.chartData = newData;
      this.updateChart();
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
  width: 0px;
}
</style>

<template>
  <div class="chart" id="chartdiv">
  </div>
</template>


<script>
import * as am4core from "@amcharts/amcharts4/core";
import * as am4plugins_forceDirected from "@amcharts/amcharts4/plugins/forceDirected"; 
import am4themes_animated from "@amcharts/amcharts4/themes/animated";
am4core.useTheme(am4themes_animated);


export default {
  name: 'Chart',
  data() {
    return {
      chartData: []
    }
  },

  mounted() {
    // Create chart
    var chart = am4core.create("chartdiv", am4plugins_forceDirected.ForceDirectedTree);

    // Create series
    var series = chart.series.push(new am4plugins_forceDirected.ForceDirectedSeries())

    // Set data
    series.data = this.chartData;

    // Set up data fields
    series.dataFields.value = "value";
    series.dataFields.name = "name";
    series.dataFields.children = "children";

    // Add labels
    series.nodes.template.label.text = "{name}";
    series.nodes.template.tooltipText = "{name}: [bold]{value}[/]";
    series.fontSize = 10;
    series.minRadius = 15;
    series.maxRadius = 40;

    series.centerStrength = 0.5;
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

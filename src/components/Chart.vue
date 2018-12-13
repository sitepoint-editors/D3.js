<template>
  <div>
    <svg></svg>
  </div>
</template>

<script>
import * as d3 from "d3";
import _ from "lodash";

export default {
  props: ["issues"],
  data() {
    return {
      chart: null
    };
  },
  watch: {
    issues(val) {
      if (this.chart != null) this.chart.remove();
      this.renderChart(val);
    }
  },
  methods: {
    renderChart(issues_val) {
      const margin = 60;
      const svg_width = 1000;
      const svg_height = 600;
      const chart_width = 1000 - 2 * margin;
      const chart_height = 600 - 2 * margin;

      const svg = d3
        .select("svg")
        .attr("width", svg_width)
        .attr("height", svg_height);

      this.chart = svg
        .append("g")
        .attr("transform", `translate(${margin}, ${margin})`);

      const yScale = d3
        .scaleLinear()
        .range([chart_height, 0])
        .domain([0, _.maxBy(issues_val, "issues").issues]);

      this.chart
        .append("g")
        .call(d3.axisLeft(yScale).ticks(_.maxBy(issues_val, "issues").issues));

      const xScale = d3
        .scaleBand()
        .range([0, chart_width])
        .domain(issues_val.map(s => s.day))
        .padding(0.2);

      this.chart
        .append("g")
        .attr("transform", `translate(0, ${chart_height})`)
        .call(d3.axisBottom(xScale));

      const barGroups = this.chart
        .selectAll("rect")
        .data(issues_val)
        .enter();

      barGroups
        .append("rect")
        .attr("class", "bar")
        .attr("x", g => xScale(g.day))
        .attr("y", g => yScale(g.issues))
        .attr("height", g => chart_height - yScale(g.issues))
        .attr("width", xScale.bandwidth())
        .on("mouseenter", function(actual, i) {
          d3.select(this)
            .transition()
            .duration(300)
            .attr("opacity", 0.6)
            .attr("x", a => xScale(a.day) - 5)
            .attr("width", xScale.bandwidth() + 10);
          barGroups
            .append("text")
            .attr("class", "value")
            .attr("x", a => xScale(a.day) + xScale.bandwidth() / 2)
            .attr("y", a => yScale(a.issues) - 20)
            .attr("text-anchor", "middle")
            .text((a, idx) => {
              return idx !== i ? "" : `${a.issues} issues`;
            });
        })
        .on("mouseleave", function() {
          d3.selectAll(".issues").attr("opacity", 1);

          d3.select(this)
            .transition()
            .duration(300)
            .attr("opacity", 1)
            .attr("x", a => xScale(a.day))
            .attr("width", xScale.bandwidth());

          svg.selectAll(".value").remove();
        });

      svg
        .append("text")
        .attr("class", "label")
        .attr("x", -(chart_height / 2) - margin)
        .attr("y", margin / 2.4)
        .attr("transform", "rotate(-90)")
        .attr("text-anchor", "middle")
        .text("Issues opened");

      svg
        .append("text")
        .attr("class", "label")
        .attr("x", chart_width / 2 + margin)
        .attr("y", chart_height + margin * 1.7)
        .attr("text-anchor", "middle")
        .text("Days");

      svg
        .append("text")
        .attr("class", "title")
        .attr("x", chart_width / 2 + margin)
        .attr("y", 40)
        .attr("text-anchor", "middle")
        .text("Issues in the past 1 week");
    }
  }
};
</script>

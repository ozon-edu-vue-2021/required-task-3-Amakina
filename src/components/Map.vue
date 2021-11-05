<template>
  <div class="map">
    <h3>Карта офиса</h3>

    <div v-show="!isLoading" class="map-root">
      <MapSVG ref="svg" />
      <TableSVG v-show="false" ref="table" />
    </div>
    <div v-show="isLoading">Loading...</div>
  </div>
</template>

<script>
import MapSVG from "@/assets/images/map.svg";
import TableSVG from "@/assets/images/workPlace.svg";
import * as d3 from "d3";
import tables from "@/assets/data/tables.json";
import legend from "@/assets/data/legend.json";

export default {
  components: {
    MapSVG,
    TableSVG,
  },
  props: {
    isUserOpenned: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      isLoading: true,
      svg: null,
      g: null,
      tables: [],
      tableSVG: null,
    };
  },
  watch: {
    isUserOpenned(v) {
      if (!v) {
        this.unselectTable();
      }
    },
  },
  destroyed() {
    this.$el.removeEventListener("click", this.onClickOutside);
  },
  mounted() {
    this.$el.addEventListener("click", this.onClickOutside);

    this.tables = tables;

    this.svg = d3.select(this.$refs.svg);
    this.g = this.svg.select("g");
    this.tableSVG = d3.select(this.$refs.table);

    if (this.g) {
      this.drawTables();
    } else {
      console.log("error");
    }
  },
  methods: {
    onClickOutside() {
      this.unselectTable();
      this.$emit("update:personSelect");
    },
    drawTables() {
      const svgTablesGroup = this.g.append("g").classed("groupPlaces", true);

      this.tables.map((table) => {
        const svgTable = svgTablesGroup
          .append("g")
          .attr("transform", `translate(${table.x}, ${table.y}) scale(0.5)`)
          .attr("id", table._id)
          .classed("employer-place", true);

        svgTable
          .append("g")
          .attr("transform", `rotate(${table.rotate || 0})`)
          .attr("group_id", table.group_id)
          .classed("table", true)
          .html(this.tableSVG.html())
          .attr(
            "fill",
            legend.find((it) => it.group_id === table.group_id)?.color ??
              "transparent"
          )
          .on("click", (e) => {
            e.stopPropagation();

            this.selectTable(svgTable);

            this.$emit("update:personSelect", table._id);
          })
          .append("circle")
          .classed("selected-element-marker", true)
          .attr("cx", 1)
          .attr("cy", 1)
          .attr("r", "2px")
          .style("fill", "transparent");
      });

      this.isLoading = false;
    },
    selectTable(svgTable) {
      this.unselectTable();

      svgTable.classed("selected-table", true);
      svgTable.select(".selected-element-marker").style("fill", "red");
    },
    unselectTable() {
      const selectedTable = d3.select(".selected-table");
      selectedTable
        .select(".selected-element-marker")
        .style("fill", "transparent");
      selectedTable.classed("selected-table", false);
    },
  },
};
</script>

<style scoped>
.map {
  height: 100%;
  width: 100%;
  padding: 24px;
  overflow: hidden;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
}

.map-root {
  height: 100%;
  width: 100%;
  overflow: hidden;
  box-sizing: border-box;
}

h3 {
  margin-top: 0px;
}

::v-deep svg {
  height: 100%;
  width: 100%;
}

::v-deep .table {
  cursor: pointer;
}
</style>

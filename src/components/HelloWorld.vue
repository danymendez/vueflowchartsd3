<template>
  <div class="container">
    <div class="row">
      <div class="col-sm-3">
        <div class="card">
          <div class="card-body" draggable="true">
            This is some text within a card body.
          </div>
        </div>
        Data Array
        <textarea
          name=""
          id=""
          cols="30"
          rows="5"
          v-model="arrayData"
          readonly
        ></textarea>
        <br />
        item Array
        <textarea
          name=""
          id=""
          cols="30"
          rows="5"
          v-model="itemData"
          readonly
        ></textarea>
      </div>

      <div
        id="zoom"
        class="col-sm-6 drawing-area"
        @drop="drop($event)"
        @dragover.prevent
        @dragenter.prevent
      ></div>
      <div class="col-sm-2">
        <div class="form-group">
          <label>x</label>
          <input v-model="x" type="text" class="form-control" />
        </div>
        <div class="form-group">
          <label>y</label>
          <input v-model="y" type="text" class="form-control" />
        </div>
        <div class="form-group">
          <label>w</label>
          <input v-model="w" type="text" class="form-control" />
        </div>
        <div class="form-group">
          <label>h</label>
          <input v-model="h" type="text" class="form-control" />
        </div>
        <div class="form-group">
          <label>id</label>
          <input v-model="id" type="text" class="form-control" />
        </div>
        <div class="form-group">
          <label>parent</label>
          <input v-model="parent" type="text" class="form-control" />
        </div>
        <div class="form-group">
          <label>lvl</label>
          <input v-model="lvl" type="text" class="form-control" />
        </div>
      </div>
      <div class="col-sm-1">
        <button type="button" v-on:click="addLine">Add Line</button>
        <button type="button" v-on:click="addRect">Draw me/Reset</button>
        <button type="button" v-on:click="addToData">Add data</button>
      </div>
    </div>
  </div>
</template>

<script>
import * as d3 from "d3";

export default {
  name: "HelloWorld",
  data() {
    return {
      svg: Object,
      g: Object,
      data: Object,
      dataline: Object,
      itemData: "",
      x: 40,
      y: 40,
      w: 100,
      h: 30,
      id: 1,
      parent: 0,
      lvl: 0,
      arrayData: "",
    };
  },
  props: {
    msg: String,
  },
  methods: {
    init() {
      this.data = [];
      this.dataline = [];
    },
    getMBottonC(x, y, w, h) {
      return {
        x: x + w / 2,
        y: y - h,
      };
    },
    getMTopC(x, y, w) {
      return {
        x: x + w / 2,
        y: y,
      };
    },
    addToData() {
      this.data.push({
        x: Number(this.x),
        y: Number(this.y),
        w: Number(this.w),
        h: Number(this.h),
        id: Number(this.id),
        parent: Number(this.parent),
        lvl: Number(this.lvl),
      });
      this.arrayData = JSON.stringify(this.data);
    },
    addData(dataObject) {
      this.data.push({
        x: dataObject.x,
        y: dataObject.y,
        w: dataObject.w,
        h: dataObject.h,
        id: dataObject.id,
        parent: dataObject.parent,
        lvl: dataObject.lvl,
      });
    },
    drawLine() {
      for (var i = 0; i < this.dataline.length; i++) {
        this.g
          .append("line")
          .style("stroke", "lightgreen")
          .style("stroke-width", 10)
          .attr("x1", this.dataline[i].x1)
          .attr("y1", this.dataline[i].y1)
          .attr("x2", this.dataline[i].x2)
          .attr("y2", this.dataline[i].y2);
      }
    },
    addDataLine(x1, y1, x2, y2) {
      this.dataline.push({ x1: x1, y1: y1, x2: x2, y2: y2 });
    },
    drop() {
      this.addRect();
    },
    resetSVGArea() {
      d3.selectAll("rect").remove();
      d3.selectAll("line").remove();
      this.resetDataFromParent();
    },
    addRect() {
      this.resetSVGArea();
      let _this = this;
      _this.g = _this.svg.append("g").attr("cursor", "grab");
      _this.g
        .selectAll("rect")
        .data(_this.data)
        .join("rect")
        .attr("x", ({ x }) => x)
        .attr("y", ({ y }) => y)
        .attr("width", ({ w }) => w)
        .attr("height", ({ h }) => h)
        .attr("stroke", "black")
        .attr("fill", "#69a3b2")
        .call(
          d3
            .drag()
            .on("start", function (_, d) {
              console.log(d);
              _this.itemData = JSON.stringify(d);
              d3.select(this).raise();
              d3.select(this).attr("stroke", "red");
              _this.g.attr("cursor", "grabbing");
            })
            .on("drag", function (event, d) {
              if (d.parent == 0) {
                d3.select(this)
                  .attr("x", (d.x = event.x))
                  .attr("y", (d.y = event.y));

                _this.addRect();
              }
            })
            .on("end", function () {
              _this.g.attr("cursor", "grab");
            })
        );
      this.drawLine();
    },
    resetDataFromParent() {
      //let _this = this;
      this.dataline = [];
      let parents = this.data.filter((item) => item.parent === 0);
      let xBetweenDistance = 40;
      let yBetweenDistance = 60;
      for (var i = 0; i < parents.length; i++) {
        if (this.hasChildren(parents[i])) {

          let children = this.getChildren(parents[i]);
          let xFirstChildren =
            parents[i].x +
            parents[i].w / 2 -
            this.getTotalWidthChildren(children, xBetweenDistance) / 2;
          let yFirstChildren = parents[i].y + parents[i].h + yBetweenDistance;
            
            //Horizontal between parents and children
          this.addDataLine(
            xFirstChildren + parents[i].w / 2,
            yFirstChildren - yBetweenDistance / 2,
            xFirstChildren -
              parents[i].w / 2 +
              this.getTotalWidthChildren(children, xBetweenDistance),
            yFirstChildren - yBetweenDistance / 2
          );

        // var gMParent = this.getMBottonC 
         this.addDataLine(
            xFirstChildren + parents[i].w / 2,
            yFirstChildren - yBetweenDistance / 2,
            xFirstChildren -
              parents[i].w / 2 +
              this.getTotalWidthChildren(children, xBetweenDistance),
            yFirstChildren - yBetweenDistance / 2
          );

          for (var j = 0; j < children.length; j++) {
            if (j == 0) {
              this.setChildrenCordenate(
                children[j],
                xFirstChildren,
                yFirstChildren
              );
            } else {
              this.setChildrenCordenate(
                children[j],
                children[j - 1].x + children[j - 1].w + xBetweenDistance,
                children[j - 1].y
              );
            }
          }
        }
      }
      this.arrayData = JSON.stringify(this.data);
    },
    setChildrenCordenate(child, x, y) {
      let index = this.data.findIndex((item) => item.id == child.id);
      console.log(index);
      this.data[index].x = x;
      this.data[index].y = y;
    },
    getTotalWidthChildren(array, xBetweenDistance) {
      let wTotal = d3.sum(array, function (d) {
        return d.w;
      });
      return wTotal + xBetweenDistance * (array.length - 1);
    },
    getChildren(parent) {
      let children = this.data.filter((item) => item.parent == parent.id);
      return children;
    },
    hasChildren(parent) {
      return this.getChildren(parent).length > 0;
    },
    createSVGArea() {
      let _this = this;
      let divID = "zoom";
      _this.svg = d3
        .select(`#${divID}`)
        .append("svg")
        .attr("viewBox", [
          0,
          0,
          document.getElementById(divID).offsetWidth,
          document.getElementById(divID).offsetHeight,
        ]);
    },
    addZoom() {
      let _this = this;
      _this.svg.call(
        d3
          .zoom()
          .extent([
            [0, 0],
            [
              document.getElementById("zoom").offsetWidth,
              document.getElementById("zoom").offsetHeight,
            ],
          ])
          .scaleExtent([1, 8])
          .on("zoom", function ({ transform }) {
            _this.g.attr("transform", transform);
          })
      );
    },
  },
  mounted() {
    this.init();
    this.createSVGArea();
    this.addZoom();
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
</style>

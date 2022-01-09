<template>
  <div class="container">
    <div class="row">
      <div class="col-sm-3">
        <div class="card">
          <div class="card-body">
            <div class="list-group list-group-library-task p-2">
              <div
                href="#!"
                s=""
                class="
                  list-group-item
                  list-group-item-action
                  list-group-item-border-top
                  p-0
                  mb-3
                "
              >
                <div
                  draggable="true"
                  @dragstart="dragstart"
                  class="
                    d-flex
                    justify-content-between
                    align-items-center
                    draggable
                    rounded
                  "
                >
                  <small class="d-flex align-items-start align-items-center"
                    ><i
                      aria-hidden="true"
                      class="fas fa-chart-line text-success fa-2x mr-2"
                    ></i>
                    <b> Demand Forecast</b></small
                  >
                  <small
                    ><i aria-hidden="true" class="task-icon-dragdrop"></i
                  ></small>
                </div>
                <p class="bg-light-library-task pb-2 mb-0">
                  <small class="text-muted"
                    >Some placeholder content in a paragraph.</small
                  >
                </p>
              </div>
              <div
                href="#"
                s=""
                class="
                  list-group-item
                  list-group-item-action
                  list-group-item-border-top
                  p-0
                  mb-3
                "
              >
                <div
                  draggable="true"
                  @dragstart="dragstart"
                  class="
                    d-flex
                    justify-content-between
                    align-items-center
                    draggable
                    rounded
                  "
                >
                  <small class="d-flex align-items-start align-items-center"
                    ><i
                      aria-hidden="true"
                      class="fas fa-history text-primary fa-2x mr-2"
                    ></i>
                    <b> Demand Forecast Conversion</b></small
                  >
                  <small
                    ><i aria-hidden="true" class="task-icon-dragdrop"></i
                  ></small>
                </div>
                <p class="bg-light-library-task pb-2 mb-0">
                  <small class="text-muted"
                    >Some placeholder content in a paragraph.</small
                  >
                </p>
              </div>
            </div>
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
        class="col-sm-9 drawing-area"
        @drop="drop($event)"
        @dragover.prevent
        @dragenter.prevent
      ></div>
      <!-- <div class="col-sm-2" >
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
        <div class="form-group">
          <label>html</label>
          <input v-model="html" type="text" class="form-control" />
        </div>
      </div>
      <div class="col-sm-1">
        <button type="button" v-on:click="addLine">Add Line</button>
        <button type="button" v-on:click="addRect">Draw me/Reset</button>
        <button type="button" v-on:click="addToData">Add data</button>
      </div> -->
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
      w: 150,
      h: 50,
      id: 1,
      parent: 0,
      lvl: 0,
      arrayData: "",
      html: `<div draggable="true" class="d-flex justify-content-between align-items-center draggable rounded">
      <small class="d-flex align-items-start align-items-center">
      <i aria-hidden="true" class="fas fa-history text-primary fa-2x mr-2"></i>
      <b> Demand Forecast Conversion</b></small><small><i aria-hidden="true" class="task-icon-dragdrop"></i>
      </small>
      </div>`,
      areaDrag: Object,
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
    getIdAreaDragging(x, y) {
      let parents = this.data.filter((item) => item.parent === 0);
      let id = 0;
      for (var i = 0; i < parents.length; i++) {
        let index = this.data.findIndex((item) => item.id == parents[i].id);
        let object = this.data[index];
        if (
          x > object.areaDrag.x &&
          x < object.x + object.w &&
          y > object.areaDrag.y &&
          y < object.areaDrag.y + object.areaDrag.h
        ) {
          id = object.id;
          break;
        }
      }
      return id;
    },
    setDragginArea() {
      let parents = this.data.filter((item) => item.parent === 0);
      for (var i = 0; i < parents.length; i++) {
        let index = this.data.findIndex((item) => item.id == parents[i].id);
        this.data[index].areaDrag = this.getAreaDragging(
          this.data[index].x,
          this.data[index].y,
          this.data[index].w,
          50
        );
      }
    },
    getAreaDragging(x, y, w, h) {
      return {
        x: x,
        y: y + h,
        w: w,
        h: h,
      };
    },
    polarToCartesian(r, theta) {
      return {
        x: Math.round(r * Math.cos((theta * Math.PI) / 180)),
        y: Math.round(r * Math.sin((theta * Math.PI) / 180)) * -1,
      };
    },
    cartesianToPolar(x, y) {
      return {
        r: Math.sqrt(Math.pow(x, 2) + Math.pow(y, 2)),
        theta: ((Math.atan(y / x) * 180) / Math.PI) * -1,
      };
    },
    getRectAxis(x, y, w, h) {
      let _this = this;
      return {
        a: {
          x: x,
          y: y,
        },
        b: {
          x2: x + w,
          y: y,
        },
        c: {
          x: x,
          y2: y + h,
        },
        d: {
          x2: x + w,
          y2: y + h,
        },
        top:{
          x:x+w/2,
          y:y
        },
        botton:{
          x:x+w/2,
          y:y+h
        },
        midsegment: {
          x: x + w / 2,
          y: y + h / 2
        },
        polar:_this.cartesianToPolar(x,y)
      };
    },
    dragstart(event) {
     let axis = this.setRectAxisFromTop(100,100,150,50);
      this.addData({
        x: axis.a.x,
        y: axis.a.y,
        w: 150,
        h: 50,
        id: this.data.length + 1,
        parent: 0,
        lvl: 0,
        html: event.target.outerHTML,
      });
    },
    getMBottonC(x, y, w, h) {
      return {
        x: x + w / 2,
        y: y + h,
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
        html: this.html,
      });
      this.arrayData = JSON.stringify(this.data);
    },
    getDistanceSourceTarget(source,target){
      return Math.sqrt(Math.pow(target.x2+source.x1,2)+Math.pow(target.y2+source.y1,2));
    },
    setRectAxisFromTop(x,y,w,h){
        let _this=this;
        return {
        a: {
          x: x-(w/2),
          y: y,
        },
        b: {
          x2: x + (w/2),
          y: y,
        },
        c: {
          x: x-(w/2),
          y2: y + h,
        },
        d: {
          x2: x + (w/2),
          y2: y + h,
        },
        top:{
          x:x,
          y:y
        },
        botton:{
          x:x-w/2,
          y:y+h
        },
        midsegment: {
          x: x,
          y: y + h / 2
        },
        polar:_this.cartesianToPolar(x,y)
      };
    },
    addData(dataObject) {
      let _this = this;
      this.data.push({
        x: dataObject.x,
        y: dataObject.y,
        w: dataObject.w,
        h: dataObject.h,
        id: dataObject.id,
        parent: dataObject.parent,
        lvl: dataObject.lvl,
        html: dataObject.html,
        rectAxisData: _this.getRectAxis(dataObject.x,dataObject.y,dataObject.w,dataObject.h),
      });
    },
    drawLine() {
      for (var i = 0; i < this.dataline.length; i++) {
        this.g
          .append("line")
          .style("stroke", "black")
          .style("stroke-width", 3)
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
      this.g.selectAll("foreignObject").remove();
      this.g.selectAll("line").remove();
      this.g.selectAll("rect").remove();

      this.resetDataFromParent();
      this.setDragginArea();
    },
    addRect() {
      this.resetSVGArea();
      let _this = this;

      _this.g
        .selectAll("foreignObject")
        .data(_this.data)
        .join("foreignObject")
        .attr("x", ({ x }) => x)
        .attr("y", ({ y }) => y)
        .attr("width", ({ w }) => w)
        .attr("height", ({ h }) => h)
        .call(
          d3
            .drag()
            .on("start", function (_, d) {
              _this.gDragStart(this, d);
            })
            .on("drag", function (event, d) {
              _this.gDragging(this, event, d);
            })
            .on("end", function (_, d) {
              _this.gDragEnd(this, d);
            })
        )
        .append("xhtml:a")
        .attr(
          "class",
          "list-group-item list-group-item-action list-group-item-border-top"
        )
        .html(({ html }) => html);
      this.drawLine();
    },
    addCircle(x, y) {
      this.g
        .append("circle")
        .attr("cx", x)
        .attr("cy", y)
        .attr("r", 5)
        .attr("stroke", "black")
        .attr("fill", "#69a3b2");
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

          //Vertical line
          var gMBottonCParent = this.getMBottonC(
            parents[i].x,
            parents[i].y,
            parents[i].w,
            parents[i].h
          );
          this.addDataLine(
            gMBottonCParent.x,
            gMBottonCParent.y,
            gMBottonCParent.x,
            gMBottonCParent.y + yBetweenDistance / 2
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
            var gMTopCChild = this.getMTopC(
              children[j].x,
              children[j].y,
              children[j].w,
              children[j].h
            );
            this.addDataLine(
              gMTopCChild.x,
              gMTopCChild.y,
              gMTopCChild.x,
              gMTopCChild.y - yBetweenDistance / 2
            );
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

      _this.g = _this.svg.append("g").attr("cursor", "grab");
    },
    gDragStart(_thisDrag, d) {
      let _this = this;
      _this.itemData = JSON.stringify(d);
      let id = _this.getIdAreaDragging(d.x, d.y);
      if (!_this.hasChildren(d)) {
        d.parent = id;
      }
      d3.select(_thisDrag).raise();
      d3.select(_thisDrag).attr("stroke", "red");
      _this.g.attr("cursor", "grabbing");
    },
    gDragging(_thisDrag, event, d) {
      let _this = this;

      d3.select(_thisDrag)
        .attr("x", (d.x = event.x))
        .attr("y", (d.y = event.y));
      let id = _this.getIdAreaDragging(d.x, d.y);
      d.rectAxisData = _this.getRectAxis(d.x,d.y,d.w,d.h);
      if (id > 0 && !_this.hasChildren(d)) {
        let parent = _this.data.filter((item) => item.id == id);
        let bottoMiddle = _this.getMBottonC(
          parent[0].x,
          parent[0].y,
          parent[0].w,
          parent[0].h
        );
        _this.addRect();
        _this.addCircle(bottoMiddle.x, bottoMiddle.y);
      } else {
        _this.g.selectAll("circle").remove();
        _this.addRect();
      }
    },
    gDragEnd(_, d) {
      let _this = this;
      if (!_this.hasChildren(d)) {
        d.parent = _this.getIdAreaDragging(d.x, d.y);
      }
      _this.g.attr("cursor", "grab");
      _this.g.selectAll("circle").remove();
      _this.addRect();
      //--- llamado al API
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
          .scaleExtent([0.7, 8])
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
<style>
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

.list-group-item-border-top {
  width: inherit;
  height: inherit;
  padding: 5px;
}

.list-group-item-border-top:nth-child() {
  width: inherit;
  height: inherit;
  padding: 0 !important;
}

a {
  color: #42b983;
}
</style>

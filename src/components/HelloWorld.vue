<template>
    <div class="container">
        <div class="row">
            <div class="col-sm-3">
                <div class="card">
                    <div class="card-body" draggable="true">
                        This is some text within a card body.
                    </div>
                </div>
            </div>
            <div id="zoom" class="col-sm-8 drawing-area" @drop="drop($event)" @dragover.prevent @dragenter.prevent>

            </div>
            <div class="col-sm-1">
                 <button type="button" v-on:click="addLine">Add Line</button>
                  <button type="button" v-on:click="drawDiagram">Draw me</button>
            </div>
        </div>
    </div>
</template>

<script>
    import * as d3 from 'd3'

    export default {
        name: 'HelloWorld',
        data() {
            return {
                svg: Object,
                g: Object,
                data: Object
            };
        },
        props: {
            msg: String
        },
        methods: {
            init() {

            },
             drawDiagram(){
                 var x1 = 40; var w = 100; var h=40; var y1 = 40;
                 var y3 = y1-h;
                  this.data.push({ x: 40, y:40 , w: w, h:h , id:1, parent:0, lvl:0});
                      this.data.push({ x: x1, y:y3 , w: w, h:h, id:2, parent:1,lvl:0 });
                         this.data.push({ x: 70*5, y:70 , w: w, h:h, id:3, parent:1,lvl:0 });
                  this.addRect();
            },
            addLine(){
                let _this = this



const found = _this.data.find(element => element.parent == 0);

console.log(found);
// const link = Array.from({length: _this.data.length}, (_, i) => {
//     console.log(_this.data.find(element => element.parent === _this.data[i].parent).x);
//   return {
//    source: [_this.data.find(element => element.parent === _this.data[1].parent).x,_this.data.find(element => element.parent === _this.data[1].parent).y],
//     target: [_this.data[1].x,_this.data[1].y]
//   };});
for(var i =0;i<_this.data.length;i++){
// const link = d3.linkHorizontal()({
//                                   source: [_this.data.find(element => element.parent === _this.data[i].parent).x,_this.data.find(element => element.parent === _this.data[i].parent).y],
//                                    target: [_this.data[i].x,_this.data[i].y]
//                                 });
if(_this.data[i].parent!=0){
                _this.g
                        .append('path')
                        .attr('d', d3.linkHorizontal()({
                                  source: [_this.data.find(element => element.id === _this.data[i].parent).x,_this.data.find(element => element.id === _this.data[i].parent).y],
                                   target: [_this.data[i].x,_this.data[i].y]
                                }))
                        .attr('stroke', 'black')
                        .attr('fill', 'none');



                        }

}
// console.log(link);


            },
            drop() {

                // var svgdoc = d3.select('svg');

                // var svgBoundClientRect = svgdoc.node().getBoundingClientRect();
                // console.log(svgBoundClientRect);
                // console.log('x: ' + event.pageX);
                // console.log('y: ' + event.pageY);

                // this.data.push({ x: (event.pageX - svgBoundClientRect.x), y: (event.pageY - svgBoundClientRect.y), w: 100, h:40 });
                // if(this.data.length===0){
                //     this.data.push({ x: 40, y: 40, w: 100, h:40 });
                // }else{
                //      this.data.push({ x: this.data.length%4===0?40:(this.data[this.data.length-1].x+(40*5)), y: this.data.length%4===0?(this.data[this.data.length-1].y+(40*3)):this.data[this.data.length-1].y, w: 100, h:40 });
                // }
                // console.log(this.data);
                this.addRect();
            },
            addRect() {

                let _this = this;
                d3.selectAll("g").remove();
                _this.g = _this.svg.append("g").attr("cursor", "grab");
                _this.g.selectAll("rect")
                    .data(_this.data)
                    .join("rect")
                    .attr("x", ({ x }) => x)
                    .attr("y", ({ y }) => y)
                    .attr('width', ({ w }) => w)
                    .attr('height', ({ h }) => h)
                    .attr('stroke', 'black')
                    .attr('fill', '#69a3b2')
                    .call(d3.drag()
                        .on("start", function () {
                            d3.select(this).raise();
                            _this.g.attr("cursor", "grabbing");
                        }).on("drag", function (event, d) {

                            d3.select(this).attr("x", d.x = event.x).attr("y", d.y = event.y);
                        })
                        .on("end", function () {
                            _this.g.attr("cursor", "grab");
                        }));


                _this.svg.call(d3.zoom()
                    .extent([[0, 0], [document.getElementById("zoom").offsetWidth, document.getElementById("zoom").offsetHeight]])
                    .scaleExtent([1, 8])
                    .on("zoom", function ({ transform }) {
                        _this.g.attr("transform", transform);
                    }));
            }
        },
        mounted() {
            let _this = this;
            _this.data = [];
            _this.svg = d3
                .select("#zoom")
                .append("svg")
                .attr("viewBox", [0, 0, document.getElementById("zoom").offsetWidth, document.getElementById("zoom").offsetHeight]);

        }
    }

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

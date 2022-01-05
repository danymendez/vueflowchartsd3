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
                <!-- <button type="button" v-on:click="addRect">Add me</button> -->
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
            drop(event) {

                var svgdoc = d3.select('svg');

                var svgBoundClientRect = svgdoc.node().getBoundingClientRect();
                console.log(svgBoundClientRect);
                console.log('x: ' + event.pageX);
                console.log('y: ' + event.pageY);

                this.data.push({ x: (event.pageX - svgBoundClientRect.x), y: (event.pageY - svgBoundClientRect.y) });
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
                    .attr('width', 40)
                    .attr('height', 40)
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

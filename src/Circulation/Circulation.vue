<template>
    <div>
        <iv-visualisation :title="pageName" :vue_config="vue_config" :page_number="4">
            <template #hotspots>
                <iv-pane position="left" format="full">
                    <iv-sidebar-content :showPagination="true">
                        <iv-sidebar-section title="Circulation">
                            Consider an arbitrary curve, [C],
                            <ul>
                                <li>Simple, piecewise-smooth and Closed,</li>
                                <li>Positive (Anti-Clockwise) Orientation,</li>
                                <li>[D] is an area enclosed by the curve, [C].</li>
                            </ul>

                            Then, for a vector field, [A], the circulation, [Gamma] is defined to be the line integral,

                            <div class="center">
                                <iv-equation-box equation="\Gamma \oint_C \mathbf{A} \cdot \, d\mathbf{r}." />
                            </div>

                            Therefore, for [EQUATION], the circulation, [Gamma], is 

                            <div class="center">
                                <iv-equation-box equation="\Gamma = \oint_C P(x,\;y(x))\;dx + Q(x(y),\;y)\;dy," />
                            </div>

                            Where [EQUATION]. <br>

                            <strong>N.B.</strong> When integrating [P] and [Q], remember to convert [P] as a function of [x] <strong>only</strong> and [Q] as a function of [y] <strong>only</strong>. <br><br>

                            Circulation is a measure of how much the vector field is "spinning" overall, it is called the <strong>macroscopic circulation</strong>.
                            
                        </iv-sidebar-section>
                    </iv-sidebar-content>
                </iv-pane>
            </template>

            <div class="center">
                <div id="graph" style="width:450px; height:450px;" />
            </div>
        </iv-visualisation>
    </div>
</template>
<script>
import vue_config from '../../vue.config.js';
import * as math from 'mathjs';
import Plotly from 'plotly.js-dist';

export default {
    name:"circulation",
    data() {
        return {
            pageName:"Circulation",
            vue_config
        }
    },
    mounted() {   
        // Colour definitions:
        let 
            // magenta = "#FF00FF",
            // blue = "#0000FF",
            // green = "#008000",
            impBlue = "#003E74",
            black = "rgb(0,0,0)",
            // white = "rgb(255,255,255)",
            cyan = "rgb(0,146,146)"; //1
            // lilac = "rgb(182,109,255)", //2
            // orange = "rgb(219,209,0)", //3
            // orange2 = "rgb(240,150,0)"; //3

        
        // Plot Utilities:
        /**
        * sets camera options in layout.
        * @param {array} point - point of viewpoint.
        */
        // function createView(point) {
        // let norm = Math.sqrt(point[0]*point[0] + (5*point[1])*(5*point[1]) + point[2]*point[2]);
        // let a = 0.5 + point[0]/norm, b = 1 +  5*point[1]/norm, c = 0.5 + point[2]/norm;
        // let camera = {
        //     up: {x: 0, y: 0, z: 1},
        //     eye: {x: -a, y: -b, z: c + 0.5},
        //     center: {x: 0, y: 0, z: -0.2}
        // }
        // return camera
        // }

        /**
        * Creates Axes Object.
        * @param {float} length - length of the half axes.
        */
        // function createAxes(length) {
        //     let axes = [];
        //     let vector = [0, length];
        //     let initialVec;
        //     let vecString = ["x", "y", "z"];
        //     let originString = ["", "", "(0,0,0)"];
        //     for (let i = 0; i < 3; ++i) {
        //         initialVec = [[0, 0], [0, 0], [0, 0]];
        //         initialVec[i] = vector;
        //         axes.push(
        //             {
        //                 type: "scatter3d",
        //                 mode: "lines+text",
        //                 x: initialVec[0],
        //                 y: initialVec[1],
        //                 z: initialVec[2],
        //                 line: {color: "rgb(0,0,0)", width: 4},
        //                 text: [originString[i], vecString[i]],
        //                 textfont: {size: 15},
        //                 textposition: "top"
        //             }
        //         );
        //     }
        //     return axes;
        // }

        // Change of Basis:
        /**
        * changes spherical to cartesian coordinates
        * @param {float} r - r.
        * @param {float} theta - theta.
        * @param {float} phi - phi.
        */
        // function sp2c(r,theta,phi) {
        //     return [
        //         r * Math.sin(theta) * Math.cos(phi),
        //         r * Math.sin(theta) * Math.sin(phi),
        //         r * Math.cos(theta)
        //     ];
        // }
        /**
        * changes cartesian to spherical coordinates
        * @param {float} x - x.
        * @param {float} y - y.
        * @param {float} z - z.
        */
        // function c2sp(x,y,z) {
        //     let r = 0, theta = 0, phi = 0;
        //     if (x*x + y*y + z*z !== 0) {
        //         r = Math.sqrt(x*x + y*y + z*z);
        //         theta = Math.acos(z/r);
        //         phi = Math.atan2(y,x);
        //     }
        //     return [r, theta, phi];
        // }
        /**
        * changes polar to cartesian coordinates
        * @param {float} rho - rho.
        * @param {float} phi - phi.
        */
        function p2c(rho, phi) {
            return [
                rho * Math.cos(phi),
                rho * Math.sin(phi)
            ];
        }
        /**
        * changes cartesian to polar coordinates
        * @param {float} x - x.
        * @param {float} y - y.
        */
        function c2p(x,y) {
            let rho = 0, phi = 0;
            if (x*x + y*y !== 0) {
                rho = Math.sqrt(x*x + y*y);
                phi = Math.atan2(y,x);
            }
            return [rho, phi];
        }


        // function addText(data, textPosition=[0,0,0], text="", color="rgb(0,0,0)", size=16){
        //     if (textPosition.length === 3) {
        //         data.push({
        //             type: "scatter3d",
        //             mode: "lines+text",
        //             x: [textPosition[0]],
        //             y: [textPosition[1]],
        //             z: [textPosition[2]],
        //             line: {width: 0},
        //             text: [text],
        //             textfont: {color:color, size:size},
        //             opacity: 1
        //         });
        //     } else if (textPosition.length === 2) {
        //         data.push({
        //             type: "scatter",
        //             mode: "lines+text",
        //             x: [textPosition[0]],
        //             y: [textPosition[1]],
        //             line: {width: 0},
        //             text: [text],
        //             textfont: {color:color, size:size},
        //             opacity: 1
        //         });
        //     } else {
        //         return 1;
        //     }
        //     return 0;
        // }

        ///Shape Objects:
        //3D Objects:
        /**
        * Represents a line.
        * @constructor
        * @param {list} points - list of the points in the line.
        */
        // function Line(points) {
        //     this.x = [];
        //     this.y = [];
        //     this.z = [];
        //     for (let i = 0; i  < points.length; ++i) {
        //         this.x.push(points[i][0]);
        //         this.y.push(points[i][1]);
        //         this.z.push(points[i][2]);
        //     }

        //     this.gObject = function(color, width=7, dash="solid") {
        //         let lineObject = {
        //             type: "scatter3d",
        //             mode: "lines",
        //             x: this.x,
        //             y: this.y,
        //             z: this.z,
        //             line: {color: color, width: width, dash:dash}
        //         }
        //         return lineObject;
        //     }

        //     this.arrowHead = function(color, width=7, wingLen=0.1, dash="solid") {
        //         let lastElm = this.x.length - 1;
        //         let [r, theta, phi] = c2sp(this.x[lastElm]-this.x[0], this.y[lastElm]-this.y[0], this.z[lastElm]-this.z[0]);
        //         let offset = [this.x[0], this.y[0], this.z[0]];
        //         let frac = wingLen*r;
        //         let sin45 = Math.sin(Math.PI/4);
        //         let d = r - frac * sin45;
        //         let wingLength = Math.sqrt(Math.pow(frac*sin45,2) + d*d);
        //         let wingAngle = Math.acos(d/wingLength);


        //         let wings_xyz = [
        //             math.add(sp2c(wingLength, theta + wingAngle, phi), offset),
        //             math.add(sp2c(wingLength, theta - wingAngle, phi), offset)
        //         ];

        //         let wings = {
        //             type: "scatter3d",
        //             mode: "lines",
        //             x: [wings_xyz[0][0], this.x[lastElm], wings_xyz[1][0]],
        //             y: [wings_xyz[0][1], this.y[lastElm], wings_xyz[1][1]],
        //             z: [wings_xyz[0][2], this.z[lastElm], wings_xyz[1][2]],
        //             line: {color: color, width: width}
        //         }

        //         return wings;
        //     }
        // }
        
        /**
        * Represents a point.
        * @constructor
        * @param {array} position - position of the point.
        */
        // function Point(position) {
        //     this.position = position;
        //     this.gObject = function(color, size = 7, symbol="circle") {
        //         let pointObject = {
        //             type: "scatter3d",
        //             mode: "markers",
        //             x: [this.position[0]],
        //             y: [this.position[1]],
        //             z: [this.position[2]],
        //             marker: {"color": color, "size": size, "symbol": symbol}
        //         }
        //         return pointObject;
        //     }
        // }
        /**
        * Represents a sphere.
        * @constructor
        * @param {float} radius - radius of the sphere.
        */
        // function Sphere(radius) {
        //     this.radius = radius;
        //     let meshSize = 20;
        //     let theta = numeric.linspace(0, Math.PI, meshSize);
        //     let phi = numeric.linspace(0, 2*Math.PI, meshSize);
        //     this.x = [], this.y = [], this.z = [];
        //     for(let i = 0; i < meshSize; ++i) {
        //         this.x[i] = [], this.y[i] = [], this.z[i] = [];
        //         for(let j = 0; j < meshSize; ++j) {
        //             this.x[i].push(radius*Math.cos(phi[i])*Math.sin(theta[j]));
        //             this.y[i].push(radius*Math.sin(phi[i])*Math.sin(theta[j]));
        //             this.z[i].push(radius*Math.cos(theta[j]));
        //         }
        //     }
        //     this.gObject = function(color1, color2) {
        //         let sphere = {
        //             type: "surface",
        //             x: this.x,
        //             y: this.y,
        //             z: this.z,
        //             showscale: false,
        //             opacity: 0.6,
        //             colorscale: [[0.0, color1], [1.0, color2]]
        //         }
        //         return sphere;
        //     }
        // }
        /**
        * Represents a cylinder.
        * @constructor
        * @param {float} radius - radius of the circular cross-section.
        * @param {float} height - height of the cylinder.
        */
        // function Cylinder(radius, height){
        //     this.radius = radius;
        //     this.height = height;
        //     let meshSize = radius * 10;
        //     if (meshSize === 0) {meshSize = 2;}
        //     let phi = numeric.linspace(0, 2*Math.PI, meshSize);
        //     this.x = [], this.y = [], this.z = [];
        //     let hValue = numeric.linspace(-height, height, meshSize);

        //     let xTemp = [], yTemp = [], zTemp = [];
        //     for(let i = 0; i < meshSize; ++i) {
        //         xTemp.push(radius*Math.cos(phi[i]));
        //         yTemp.push(radius*Math.sin(phi[i]));
        //     }
        //     for(let i = 0; i < meshSize; ++i) {
        //         this.x.push(xTemp);
        //         this.y.push(yTemp);
        //         this.z.push(numeric.rep([meshSize], hValue[i]));
        //     }

        //     this.gObjectCurve = function(color1, color2) {
        //         let curve = {
        //             type: "surface",
        //             x: this.x,
        //             y: this.y,
        //             z: this.z,
        //             showscale: false,
        //             opacity: 0.7,
        //             colorscale: [[0.0, color1], [1.0, color2]]
        //         }
        //         return curve;
        //     }
        //     this.gObjectTop = function(color) {
        //         let top = {
        //             type: "scatter3d",
        //             mode: "lines",
        //             x: this.x[0],
        //             y: this.y[0],
        //             z: this.z[meshSize - 1],
        //             line: {color: color.slice(0, -1) + ",0.2)", width: 1},
        //             surfaceaxis: 2,
        //             opacity: 0.5
        //         }
        //         return top;
        //     }
        //     this.gObjectBottom = function(color) {
        //         let bottom = {
        //             type: "scatter3d",
        //             mode: "lines",
        //             x: this.x[0],
        //             y: this.y[0],
        //             z: this.z[0],
        //             line: {color: color.slice(0, -1) + ",0.7)", width: 1},
        //             surfaceaxis: 2,
        //             opacity: 0.7
        //         }
        //         return bottom;
        //     }
        // }
        /**
        * Represents a cuboid.
        * @constructor
        * @param {float} x - half length of cuboid in x-direction.
        * @param {float} y - half length of cuboid in y-direction.
        * @param {float} z - half length of cuboid in z-direction.
        */
        // function Cuboid(x, y, z){
        //     this.width = x,
        //     this.length = y,
        //     this.height = z,
        //     this.gObject = function(color) {
        //         let cuboid = {
        //             type: "mesh3d",
        //             x: [-x, -x, x, x, -x, -x, x, x],
        //             y: [-y, y, y, -y, -y, y, y, -y],
        //             z: [-z, -z, -z, -z, z, z, z, z],
        //             i : [0, 0, 3, 4, 4, 4, 4, 4, 5, 6, 6, 7],
        //             j : [2, 3, 4, 3, 6, 7, 1, 5, 2, 2, 7, 3],
        //             k : [1, 2, 0, 7, 5, 6, 0, 1, 1, 5, 2, 2],
        //             opacity: 0.5,
        //             colorscale: [['0', color], ['1', "rgb(255,255,255)"]],
        //             intensity: [0, 0.1, 0.3, 0.5, 0.7, 0.8, 0.9, 1],
        //             showscale: false
        //         }
        //         return cuboid
        //     }
        // }
        //2D Objects:
        /**
        * Represents a circle.
        * @constructor
        * @param {float} radius - radius of the circle.
        */
        // function Circle(radius) {
        //     this.radius = radius;

        //     this.gObject = function(color=cyan, centre=[0,0]) {
        //         let phi = numeric.linspace(0, 2*Math.PI, 40);
        //         let x = [], y = [];
        //         for (let i=0, n=phi.length; i<n; ++i) {
        //             x.push(this.radius*Math.cos(phi[i]) + centre[0]);
        //             y.push(this.radius*Math.sin(phi[i]) + centre[1]);
        //         }
        //         let circle = {
        //             type: "scatter",
        //             mode: "lines",
        //             x: x,
        //             y: y,
        //             line: {simplify: false, color: color},
        //             fill:'toself',
        //             opacity: 0.5
        //         }
        //         return circle;
        //     }
        // }
        /**
        * Represents a rectangle.
        * @constructor
        * @param {float} x0 - width (x direction).
        * @param {float} y0 - length (y direction).
        */
        function Rectangle(x0, y0) {
            this.x0 = x0;
            this.y0 = y0;

            this.gObject = function(color=cyan) {
                let rectangle = {
                    type: "scatter",
                    mode: "lines",
                    x: [0, this.x0, this.x0, 0, 0],
                    y: [0, 0, this.y0, this.y0, 0],
                    line: {simplify: false, color: color},
                    fill:'toself',
                    opacity: 0.5
                }
                return rectangle;
            }
        }
        /**
        * Represents a line.
        * @constructor
        * @param {list} points - list of the points in the line.
        */
        function Line2d(points) {
            this.x = [];
            this.y = [];

            for (let i = 0; i  < points.length; ++i) {
                this.x.push(points[i][0]);
                this.y.push(points[i][1]);
            }

            this.gObject = function(color, width=7, dash="solid") {
                let lineObject = {
                    type: "scatter",
                    mode: "lines",
                    x: this.x,
                    y: this.y,
                    line: {color: color, width: width, dash:dash}
                }
                return lineObject;
            }

            this.arrowHead = function(color, width=7, dash="solid") {
                let lastElm = this.x.length - 1;
                let [rho, phi] = c2p(this.x[lastElm] - this.x[0], this.y[lastElm] - this.y[0]);
                let offset = [this.x[0], this.y[0]];
                let frac = 0.2*rho;
                let sin45 = Math.sin(Math.PI/4);
                let d = rho - frac * sin45;
                let wingLength = Math.sqrt(Math.pow(frac*sin45,2) + d*d);
                let wingAngle = Math.acos(d/wingLength);

                let wings_xyz = [
                    math.add(p2c(wingLength, phi + wingAngle), offset),
                    math.add(p2c(wingLength, phi - wingAngle), offset)
                ];

                let wings = {
                    type: "scatter",
                    mode: "lines",
                    x: [wings_xyz[0][0], this.x[lastElm], wings_xyz[1][0]],
                    y: [wings_xyz[0][1], this.y[lastElm], wings_xyz[1][1]],
                    line: {color: color, width: width, dash: dash}
                }

                return wings;
            }
        }

        let layout = {
            width: 450, "height": 500,
            margin: {l:30, r:30, t:30, b:30},
            hovermode: "closest",
            showlegend: false,
            xaxis: {range: [-5, 5], zeroline: true, title: "x"},
            yaxis: {range: [-5, 5], zeroline: true, title: "y"},
            aspectratio: {x:1, y:1}
        };


        // let isBlackText = false;
        // let blackTextType = "lines";

        //plot
        function plotData(){
            Plotly.newPlot("graph", circulation(), layout)
            return;
        }

        //create a square with arrows going anticlockwise on each side
        function circulation(){

            let data = []
            let square = new Rectangle(3,2.8)
            data.push(square.gObject(impBlue))
            let arr1 = new Line2d([[-1,0],[1.5,0]])
            let arr2 = new Line2d([[3,-1],[3,1.4]])
            let arr3 = new Line2d([[4,2.8],[1.5,2.8]])
            let arr4 = new Line2d([[0,3.8],[0,1.4]])
            data.push(arr1.arrowHead(black,3))
            data.push(arr2.arrowHead(black,3))
            data.push(arr3.arrowHead(black,3))
            data.push(arr4.arrowHead(black,3))
            data.push({
                x: [1.5],
                y: [1.4],
                mode: 'text',
                text: ['D'],
                type: 'scatter',
                showlegend: true,
                font: {size: 30}
                })

            return data;
        }

        //load main
        function main() {

            plotData();
            // initGuidance(["graph","text"]);
        }

        main();
    },
}
</script>
<style>
.center {
    display: flex;
    flex-direction: column;
    place-items: center;
    /* padding-top: 100px; */
}
</style>
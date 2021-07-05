<template>
    <div>
        <iv-visualisation :title="pageName" :vue_config="vue_config" :page_number="2">
            <template #hotspots>
                <iv-pane position="left" format="full">
                    <iv-sidebar-content :showPagination="true">
                        <iv-sidebar-section title="Divergence">
                            The divergence of a vector field [A] is: <br>
                            <div class="center">
                                <iv-equation-box equation="\text{divergence} \, \mathbf{A} = \nabla \cdot \mathbf{A}" />
                            </div>

                            It describes the strength of the source or sink for a given point. <br><br>

                            The Divergence Theorem states that the divergence of a vector field inside a region is equal to the flux of the vector field flowing through the region. Suppose [V] is the volume bounded by region [S]. If [A] is a continuously differentiable vector field defined on a neighborhood of [v], we have: <br>
                            <div class="center">
                                <iv-equation-box equation="\iiint_V (\nabla \cdot \mathbf{A}) \,dV = \oiint_S (\mathbf{A} \cdot \mathbf{\hat{n}}) \,dS" />
                            </div>
                            Where [n] is the out pointing normal of the surface element [dS]. <br><br>

                            A simple demonstration is shown in the animation on the right hand side. Each unit volume element [dV] can be considered as a small cube containing a source, with flux coming out on each side of the cube. When placing two neighbouring volume elements together, the flux flowing through the adjacent sides will cancel each other. Hence the overall outcome of piling up all the volume elements will be the flux coming out of the entire surface. Since the volume elements are small enough to be considered as a point source, this total flux is therefore equal to the vector sum of the total strength of each source, which is the total divergence.
                            
                        </iv-sidebar-section>
                    </iv-sidebar-content>
                </iv-pane>

                <iv-fixed-hotspot position="bottom" transparent>
                    <iv-slider id="frameSlider" name="Frame #" :min="0" :max="176" :step="1" :tick_step="20" :init_val="0" :playButton="true" @sliderChangedbyPlay="changeSlider" @click="changeSlider" />
                </iv-fixed-hotspot>
            </template>

            <!-- Graph -->
            <div class="center" style="padding-top: 50px;">
                <div id="graph" style="width:450px; height:450px;"></div>
            </div>
        </iv-visualisation>
    </div>
</template>
<script>
import vue_config from '../../vue.config.js'
import Plotly from 'plotly.js-dist'
import $ from 'jquery';
import * as math from 'mathjs';
import numeric from 'numeric';
// import MathJax from 'mathjax'

export default {
    name: "divergence",
    data() {
        return {
            pageName:"Divergence",
            vue_config,
            isPaused: false,
            frameNo: 0,
            redrawPlot: false,
        };
    },
    methods: {
        changeSlider(e) {
            this.frameNo = e;
        }
    },
    watch: {
        frameNo: function() {
            this.redrawPlot = true;
        }
    },
    mounted() {
        let v = this;
        // MathJax.typeset();

        let animationFrames;
        let animationIndex, animationLimit;
        let duration = 50;
        let isPaused = v.isPaused;
        let stops;
        let playID, sliderID;

        /**
         * initialises the animation frames.
         * @param {string} playButtonID - Play button ID.
         * @param {object} allFrames - The frames needed to animate.
         * @param {list} extra - list of objects with inanimated plots.
         * @param {object} layout - layout for the animation.
         * @param {float} setDuration - frame transition duration (ms).
         * @param {list} stopValues - stopping points (limit: upto 2 stops can be introduced)
         */
        function initAnimation(
            playButtonID,
            allFrames,
            extra = [],
            layoutIn = {},
            setDuration = 50,
            stopValues = [0, 0]
        ) {
            Plotly.purge("graph");
            playID = playButtonID;
            sliderID = "#" + playID.slice() + "Slider";
            duration = setDuration;
            animationFrames = allFrames;
            animationLimit = animationFrames.length;
            stops = stopValues;
            isPaused = true;
            animationIndex = 0;
            let dataIn = [];
            for (let i = 0, n = animationFrames[0].data.length; i < n; ++i) {
                dataIn.push({
                    type: animationFrames[0].data.type,
                });
            }
            for (let i = 0, n = extra.length; i < n; ++i) {
                dataIn.push(extra[i]);
            }
            Plotly.newPlot("graph", dataIn, (layout = layoutIn));
            reset();
        }

        /** resets animation. */
        function reset() {
            isPaused = true;
            animationIndex = 0;
            historyPlot(animationIndex);
            // document.getElementById(playID).value = isPaused ? "Play" : "Pause";
            // v.buttonState = 0;
            updateSlider();
            return;
        }

        /**
         * plots index-th frame
         * @param {int} index - index of the frame
         */
        function historyPlot(index) {
            // console.log("historyPlot");
            animationIndex = index;
            let data = [];
            for (let i = 0, n = animationFrames[index].data.length; i < n; ++i) {
                data.push(animationFrames[index].data[i]);
            }
            Plotly.animate(
                "graph",
                { data: data },
                {
                    fromcurrent: true,
                    transition: { duration: 0 },
                    frame: { duration: 0, redraw: true },
                }
            );
            isPaused = true;
            // document.getElementById(playID).value = isPaused ? "Play" : "Pause";
            // v.buttonState = 0;
            return;
        }

        /** Updates animation. */
        function update() {
            animationIndex++;
            if (animationIndex === animationLimit) {
                isPaused = true;
                // document.getElementById(playID).value = "Reset";
                // v.buttonState = 3;
                return;
            }
            if (!isPaused) {
                let data = [];
                for (let i = 0, n = animationFrames[1].data.length; i < n; ++i) {
                    data.push(animationFrames[animationIndex].data[i]);
                }
                Plotly.animate(
                    "graph",
                    { data: data },
                    {
                        fromcurrent: true,
                        transition: { duration: duration },
                        frame: { duration: duration, redraw: true },
                        mode: "next",
                    }
                );
                pauseComp(duration + 5);
                requestAnimationFrame(update);
                updateSlider();
                //Add stopping functionality here!!!
                if (
                    animationIndex === stops[0] ||
                    animationIndex === stops[1] ||
                    animationIndex === stops[2] ||
                    animationIndex === stops[3] ||
                    animationIndex === stops[4] ||
                    animationIndex === stops[5] ||
                    animationIndex === stops[6] ||
                    animationIndex === stops[7]
                ) {
                    isPaused = !isPaused;
                    // document.getElementById(playID).value = "Continue";
                    // v.buttonState = 2;
                    animationIndex--;
                }
            }
            return;
        }

        /**
         * pauses the computation.
         * @param {float} - time duration to pause the computation. (ms)
         */
        function pauseComp(ms) {
            ms += new Date().getTime();
            while (new Date() < ms);
            return;
        }

        /** updates linked frame slider value and position. */
        function updateSlider() {
            // console.log("updating slider");
            $(sliderID).val(animationIndex);
            v.frameNo = animationIndex;
            $(sliderID + "Display").text(animationIndex);
        }

        /** Starts the animation. */
        function startAnimation() {
            if (animationIndex < animationLimit) {
                isPaused = !isPaused;
                // if (isPaused) {
                //     v.buttonState = 1;
                // } else {
                //     v.buttonState = 0;
                // }
                requestAnimationFrame(update);
            } else {
                reset();
            }
            return;
        }

        let magenta = "#FF00FF",
            // blue = "#0000FF",
            green = "#008000",
            impBlue = "#003E74",
            black = "rgb(0,0,0)",
            white = "rgb(255,255,255)",
            // cyan = "rgb(0,146,146)", //1
            lilac = "rgb(182,109,255)", //2
            orange = "rgb(219,209,0)", //3
            red = "rgb(160, 0, 0)";
            // realOrange = "rgb(227,104, 0)";

        // Plot Utilities:
        /**
         * sets camera options in layout.
         * @param {array} point - point of viewpoint.
         */
        function createView(point) {
            let norm = Math.sqrt(point[0]*point[0] + (5*point[1])*(5*point[1]) + point[2]*point[2]);
            let a = 0.5 + point[0]/norm, b = 1 +  5*point[1]/norm, c = 0.5 + point[2]/norm;
            let camera = {
                up: {x: 0, y: 0, z: 1},
                eye: {x: -a, y: -b, z: c + 0.5},
                center: {x: 0, y: 0, z: -0.2}
            }
            return camera;
        }
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
        function sp2c(r,theta,phi) {
            return [
                r * Math.sin(theta) * Math.cos(phi),
                r * Math.sin(theta) * Math.sin(phi),
                r * Math.cos(theta)
            ];
        }
        /**
         * changes cartesian to spherical coordinates
         * @param {float} x - x.
         * @param {float} y - y.
         * @param {float} z - z.
         */
        function c2sp(x,y,z) {
            let r = 0, theta = 0, phi = 0;
            if (x*x + y*y + z*z !== 0) {
                r = Math.sqrt(x*x + y*y + z*z);
                theta = Math.acos(z/r);
                phi = Math.atan2(y,x);
            }
            return [r, theta, phi];
        }
        /**
         * changes polar to cartesian coordinates
         * @param {float} rho - rho.
         * @param {float} phi - phi.
         */
        // function p2c(rho, phi) {
        //     return [
        //         rho * Math.cos(phi),
        //         rho * Math.sin(phi)
        //     ];
        // }
        /**
         * changes cartesian to polar coordinates
         * @param {float} x - x.
         * @param {float} y - y.
         */
        // function c2p(x,y) {
        //     let rho = 0, phi = 0;
        //     if (x*x + y*y !== 0) {
        //         rho = Math.sqrt(x*x + y*y);
        //         phi = Math.atan2(y,x);
        //     }
        //     return [rho, phi];
        // }


        ///Shape Objects:
        //3D Objects:
        /**
         * Represents a line.
         * @constructor
         * @param {list} points - list of the points in the line.
         */
        function Line(points) {
            this.x = [];
            this.y = [];
            this.z = [];
            for (let i = 0; i  < points.length; ++i) {
                this.x.push(points[i][0]);
                this.y.push(points[i][1]);
                this.z.push(points[i][2]);
            }

            this.gObject = function(color, width=7, dash="solid") {
                let lineObject = {
                    type: "scatter3d",
                    mode: "lines",
                    x: this.x,
                    y: this.y,
                    z: this.z,
                    line: {color: color, width: width, dash:dash}
                }
                return lineObject;
            }

            this.arrowHead = function(color, width=7, wingLen=0.1, dash="solid") {
                let lastElm = this.x.length - 1;
                let [r, theta, phi] = c2sp(this.x[lastElm]-this.x[0], this.y[lastElm]-this.y[0], this.z[lastElm]-this.z[0]);
                let offset = [this.x[0], this.y[0], this.z[0]];
                let frac = wingLen*r;
                let sin45 = Math.sin(Math.PI/4);
                let d = r - frac * sin45;
                let wingLength = Math.sqrt(Math.pow(frac*sin45,2) + d*d);
                let wingAngle = Math.acos(d/wingLength);


                let wings_xyz = [
                    math.add(sp2c(wingLength, theta + wingAngle, phi), offset),
                    math.add(sp2c(wingLength, theta - wingAngle, phi), offset)
                ];

                let wings = {
                    type: "scatter3d",
                    mode: "lines",
                    x: [wings_xyz[0][0], this.x[lastElm], wings_xyz[1][0]],
                    y: [wings_xyz[0][1], this.y[lastElm], wings_xyz[1][1]],
                    z: [wings_xyz[0][2], this.z[lastElm], wings_xyz[1][2]],
                    line: {color: color, width: width, dash: dash}
                }

                return wings;
            }
        }
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
         * Represents a cube with arrows pointing out on each side.
         * @constructor
         * @param {float} x - x position of the centre of the cube.
         * @param {float} y - y position of the centre of the cube.
         * @param {float} z - z position of the centre of the cube.
         * @param {float} a - half length of the sides of the cube.
         */
        function outerCube(x,y,z,a) {
            let cube = []
            cube.push({
                    type: "mesh3d",
                    x: [x-a, x-a, x+a, x+a, x-a, x-a, x+a, x+a],
                    y: [y-a, y+a, y+a, y-a, y-a, y+a, y+a, y-a],
                    z: [z-a, z-a, z-a, z-a, z+a, z+a, z+a, z+a],
                    i : [0, 0, 3, 4, 4, 4, 4, 4, 5, 6, 6, 7],
                    j : [2, 3, 4, 3, 6, 7, 1, 5, 2, 2, 7, 3],
                    k : [1, 2, 0, 7, 5, 6, 0, 1, 1, 5, 2, 2],
                    opacity: 0.8,
                    colorscale: [['0', orange], ['1', white]],
                    intensity: [0, 0.1, 0.3, 0.5, 0.7, 0.8, 0.9, 1],
                    showscale: false
                }
            )
            let arr1 = new Line([[x+a,y,z],[x+a+1.5,y,z]])
            let arr11 = new Line([[x+a-1,y,z],[x+a+1.5,y,z]])
            let arr2 = new Line([[x-a,y,z],[x-a-1.5,y,z]])
            let arr22 = new Line([[x-a+1,y,z],[x-a-1.5,y,z]])
            let arr3 = new Line([[x,y+a,z],[x,y+a+1.5,z]])
            let arr33 = new Line([[x,y+a-1,z],[x,y+a+1.5,z]])
            let arr4 = new Line([[x,y-a,z],[x,y-a-1.5,z]])
            let arr44 = new Line([[x,y-a+1,z],[x,y-a-1.5,z]])
            let arr5 = new Line([[x,y,z+a],[x,y,z+a+1.5]])
            let arr55 = new Line([[x,y,z+a-1],[x,y,z+a+1.5]])
            let arr6 = new Line([[x,y,z-a],[x,y,z-a-1.5]])
            let arr66 = new Line([[x,y,z-a+1],[x,y,z-a-1.5]])
            cube.push(arr1.gObject(magenta,7))
            cube.push(arr11.arrowHead(magenta,7))
            cube.push(arr2.gObject(magenta,7))
            cube.push(arr22.arrowHead(magenta,7))
            cube.push(arr3.gObject(magenta,7))
            cube.push(arr33.arrowHead(magenta,7))
            cube.push(arr4.gObject(magenta,7))
            cube.push(arr44.arrowHead(magenta,7))
            cube.push(arr5.gObject(magenta,7))
            cube.push(arr55.arrowHead(magenta,7))
            cube.push(arr6.gObject(magenta,7))
            cube.push(arr66.arrowHead(magenta,7))
            return cube
        }
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
        //             z: [0, 0, 0, 0, z, z, z, z],
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

        /**
         * Represents a cuboid.
         * @constructor
         * @param {float} x - x position of the centre of the cube.
         * @param {float} y - y position of the centre of the cube.
         * @param {float} z - z position of the centre of the cube.
         * @param {float} a - half length of cuboid in x-direction.
         * @param {float} b - half length of cuboid in y-direction.
         * @param {float} c - half length of cuboid in z-direction.
         */
        function Cube(x, y, z, a, b, c){

            this.gObject = function(color1,color2, opacity) {
                let cuboid = {
                    type: "mesh3d",
                    x: [x-a, x-a, x+a, x+a, x-a, x-a, x+a, x+a],
                    y: [y-b, y+b, y+b, y-b, y-b, y+b, y+b, y-b],
                    z: [z-c, z-c, z-c, z-c, z+c, z+c, z+c, z+c],
                    i : [0, 0, 3, 4, 4, 4, 4, 4, 5, 6, 6, 7],
                    j : [2, 3, 4, 3, 6, 7, 1, 5, 2, 2, 7, 3],
                    k : [1, 2, 0, 7, 5, 6, 0, 1, 1, 5, 2, 2],
                    opacity: opacity, // was 0.5
                    colorscale: [['0', color1], ['1', color2]],
                    intensity: [0, 0.1, 0.3, 0.5, 0.7, 0.8, 0.9, 1],
                    showscale: false,
                    // opacity: opacity
                }
                return cuboid
            }
        }

        /**
        * Represents a 3d pringle shaped loop with an arrow pointing anticlockwise.
        * @constructor
        * @param {float} radius - radius of the loop.
        * @param {array} centre - central point of the loop.
        * @param {float} height - distance of the centre of the loop away from the z=0 plane
        * @param {float} wave - number of periods of the sin wave on the loop, has to be even to close the loop smoothly, >= 2
        */
        // function Pringles(radius, center) {
        //     this.radius = radius;
        //     this.center = center;


        //     this.gObject = function(color=black, width=7, dash="solid", height=1, wave=4,opacity=0.5) {
        //         let meshSize = 40; //multiple of 4!
        //         let phi = numeric.linspace(0, 2*Math.PI, meshSize);
        //         let theta = numeric.linspace(0,wave*Math.PI, meshSize);
        //         let x = [], y = [], z = [];

        //         for (let i=0, n=phi.length; i<n; ++i) {
        //             x.push(this.radius*Math.cos(phi[i]) + this.center[0]);
        //             y.push(this.radius*Math.sin(phi[i]) + this.center[1]);
        //             z.push(height*(height*Math.sin(theta[i]) + this.center[2])+height);
        //         }
        //         let line2 = new Line([[x[0],y[0],z[0]],[x[7],y[7],z[7]]])
        //         let lineObject = [{
        //             type: "scatter3d",
        //             mode: "lines",
        //             x: x,
        //             y: y,
        //             z: z,
        //             line: {color: color, width: width, dash:dash},
        //             surfaceaxis: 2,
        //             surfacecolor: green,
        //             opacity: opacity
        //             },
        //             line2.arrowHead(magenta,5)
        //         ]


        //         return lineObject;
        //     }

        // }

        /**
        * Represents a hemisphere bounded by the pringle loop and an arrow pointing upwards.
        * @constructor
        * @param {float} radius - radius of the loop.
        */

        // function Sphere(radius) {
        //     this.radius = radius;
        //     this.gObject = function(color1,color2, width=7, dash="solid",size, height=1, wave=4) {
        //         let meshSize = 40;
        //         let phi = numeric.linspace(0, 2*Math.PI, meshSize);
        //         let theta= numeric.linspace(0, 0.5*Math.PI, meshSize);
        //         let beta = numeric.linspace(0,wave*Math.PI, meshSize);

        //         this.x = [], this.y = [], this.z = [];
        //         for(let i = 0; i < meshSize; ++i) {
        //             this.x[i] = [], this.y[i] = [], this.z[i] = [];
        //             for(let j = 0; j < meshSize; ++j) {
        //                 this.x[i].push(this.radius*Math.cos(phi[i])*Math.sin(theta[j])+1);
        //                 this.y[i].push(this.radius*Math.sin(phi[i])*Math.sin(theta[j])+1);
        //                 this.z[i].push((size* this.radius*Math.cos(theta[j])+height*(Math.sin(theta[j])*Math.sin(theta[j])*Math.sin(theta[j]))*(height*Math.sin(beta[i]) + 1)*(Math.sin(theta[j]))+height));
        //             }
        //         }
        //         let x1 = 1;
        //         let y1 = 1;
        //         let z1 = height*(Math.sin(0.5*Math.PI)*Math.sin(0.5*Math.PI)*Math.sin(0.5*Math.PI))*(height*Math.sin(beta[0]) + 1)*(Math.sin(0.5*Math.PI));
        //         let x2 = this.radius*Math.cos(phi[0])*Math.sin(theta[0])+1;
        //         let y2 = this.radius*Math.sin(phi[0])*Math.sin(theta[0])+1;
        //         let z2 =this.radius*Math.cos(theta[0])+(Math.sin(theta[0])*Math.sin(theta[0])*Math.sin(theta[0]))*(Math.sin(beta[0]) + 1)*(Math.sin(theta[0]))+6;
        //         let line1 = new Line([[x1,y1,z1],[x2,y2,z2]])
        //         let sphere = [{
        //             type: "surface",
        //             x: this.x,
        //             y: this.y,
        //             z: this.z,
        //             showscale: false,
        //             opacity: 0.6,
        //             colorscale: [[0.0, color1], [1.0, color2]]
        //         },
        //         line1.gObject(black,3),
        //         line1.arrowHead(magenta,5)
        //         ]

        //         return sphere;
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
        * Represents a line.
        * @constructor
        * @param {list} points - list of the points in the line.
        */
        // function Line2d(points) {
        //     this.x = [];
        //     this.y = [];

        //     for (let i = 0; i  < points.length; ++i) {
        //         this.x.push(points[i][0]);
        //         this.y.push(points[i][1]);
        //     }

        //     this.gObject = function(color, width=7, dash="solid") {
        //         let lineObject = {
        //             type: "scatter",
        //             mode: "lines",
        //             x: this.x,
        //             y: this.y,
        //             line: {color: color, width: width, dash:dash}
        //         }
        //         return lineObject;
        //     }

        //     this.arrowHead = function(color, width=7, dash="solid") {
        //         let lastElm = this.x.length - 1;
        //         let [rho, phi] = c2p(this.x[lastElm] - this.x[0], this.y[lastElm] - this.y[0]);
        //         let offset = [this.x[0], this.y[0]];
        //         let frac = 0.2*rho;
        //         let sin45 = Math.sin(Math.PI/4);
        //         let d = rho - frac * sin45;
        //         let wingLength = Math.sqrt(Math.pow(frac*sin45,2) + d*d);
        //         let wingAngle = Math.acos(d/wingLength);

        //         let wings_xyz = [
        //             math.add(p2c(wingLength, phi + wingAngle), offset),
        //             math.add(p2c(wingLength, phi - wingAngle), offset)
        //         ];

        //         let wings = {
        //             type: "scatter",
        //             mode: "lines",
        //             x: [wings_xyz[0][0], this.x[lastElm], wings_xyz[1][0]],
        //             y: [wings_xyz[0][1], this.y[lastElm], wings_xyz[1][1]],
        //             line: {color: color, width: width}
        //         }

        //         return wings;
        //     }
        // }

        let layout = {
            width: 450, height: 500,
            margin: {l:0, r:0, t:0, b:0},
            hovermode: "closest",
            showlegend: false,
            scene: {
                camera: createView([1,1,1]),
                xaxis: {range: [-2.8, 3.2], zeroline: true, autorange: false,},
                yaxis: {range: [-2.8, 3.2], zeroline: true, autorange: false,},
                zaxis: {range: [-2.8, 3.2], zeroline: true, autorange: false,},
                aspectratio: {x:1, y:1, z:1},
            }
        };

        // let isBlackText = false;
        // let blackTextType = "lines";

        //create small cubes with arrow pointing out on each side.
        //by defining the position of the cube, the position of the arrows are set such that the arrows on two neighbour side of two cubes won't overlap each other
        //by set innerArr = fault, the arrow can be removed

        function cubeAndArrow(x,y,z,color,color1,innerArr1 = true,innerArr2 = true,innerArr3 = true,innerArr4 = true,innerArr5 = true,innerArr6 = true){
            let Cubes = []
            let cube = new Cube(x,y,z,0.5,0.5,0.5);
            Cubes.push(cube.gObject(color1, white , 0.8));

            if (innerArr1){
                if (x>0){
                    let arr1 = new Line([[x-0.5,y+0.2,z],[x-1,y+0.2,z]]);
                    let arr11 = new Line([[x+0.5,y+0.2,z],[x-1,y+0.2,z]]);
                    Cubes.push(arr1.gObject(color, 3));
                    Cubes.push(arr11.arrowHead(color,3));
                }else{
                    let arr1 = new Line([[x-0.5,y,z],[x-1,y,z]]);
                    let arr11 = new Line([[x+0.5,y,z],[x-1,y,z]]);
                    Cubes.push(arr1.gObject(color, 3));
                    Cubes.push(arr11.arrowHead(color,3));
                }
            }else{
                let arr1 = new Line([[0,0,0],[0,0,0]]);
                let arr11 = new Line([[0,0,0],[0,0,0]]);
                Cubes.push(arr1.gObject(color, 0));
                Cubes.push(arr11.arrowHead(color,0));
            }

            if (innerArr2){
                if (x<0){
                    let arr2 = new Line([[x+0.5,y+0.2,z],[x+1,y+0.2,z]]);
                    let arr22 = new Line([[x-0.5,y+0.2,z],[x+1,y+0.2,z]]);
                    Cubes.push(arr2.gObject(color, 3));
                    Cubes.push(arr22.arrowHead(color,3));
                }else{
                    let arr2 = new Line([[x+0.5,y,z],[x+1,y,z]]);
                    let arr22 = new Line([[x-0.5,y,z],[x+1,y,z]]);
                    Cubes.push(arr2.gObject(color, 3));
                    Cubes.push(arr22.arrowHead(color,3));
                }
            }else{
                let arr2 = new Line([[0,0,0],[0,0,0]]);
                let arr22 = new Line([[0,0,0],[0,0,0]]);
                Cubes.push(arr2.gObject(color, 0));
                Cubes.push(arr22.arrowHead(color,0));
            }

            if (innerArr3){
                if (y>0){
                    let arr3 = new Line([[x,y-0.5,z+0.2],[x,y-1,z+0.2]]);
                    let arr33 = new Line([[x,y+0.5,z+0.2],[x,y-1,z+0.2]]);
                    Cubes.push(arr3.gObject(color, 3));
                    Cubes.push(arr33.arrowHead(color,3));
                }else{
                    let arr3 = new Line([[x,y-0.5,z],[x,y-1,z]]);
                    let arr33 = new Line([[x,y+0.5,z],[x,y-1,z]]);
                    Cubes.push(arr3.gObject(color, 3));
                    Cubes.push(arr33.arrowHead(color,3));
                }
            }else{
                let arr3 = new Line([[0,0,0],[0,0,0]]);
                let arr33 = new Line([[0,0,0],[0,0,0]]);
                Cubes.push(arr3.gObject(color, 0));
                Cubes.push(arr33.arrowHead(color,0));
            }

            if (innerArr4){
                if (y<0){
                    let arr4 = new Line([[x,y+0.5,z+0.2],[x,y+1,z+0.2]]);
                    let arr44 = new Line([[x,y-0.5,z+0.2],[x,y+1,z+0.2]]);
                    Cubes.push(arr4.gObject(color, 3));
                    Cubes.push(arr44.arrowHead(color,3));
                }else{
                    let arr4 = new Line([[x,y+0.5,z],[x,y+1,z]]);
                    let arr44 = new Line([[x,y-0.5,z],[x,y+1,z]]);
                    Cubes.push(arr4.gObject(color, 3));
                    Cubes.push(arr44.arrowHead(color,3));
                }
            }else{
                let arr4 = new Line([[0,0,0],[0,0,0]]);
                let arr44 = new Line([[0,0,0],[0,0,0]]);
                Cubes.push(arr4.gObject(color, 0));
                Cubes.push(arr44.arrowHead(color,0));
            }

            if (innerArr5){
                if (z>0){
                    let arr5 = new Line([[x+0.2,y,z-0.5],[x+0.2,y,z-1]]);
                    let arr55 = new Line([[x+0.2,y,z+0.5],[x+0.2,y,z-1]]);
                    Cubes.push(arr5.gObject(color, 3));
                    Cubes.push(arr55.arrowHead(color,3));
                }else{
                    let arr5 = new Line([[x,y,z-0.5],[x,y,z-1]]);
                    let arr55 = new Line([[x,y,z+0.5],[x,y,z-1]]);
                    Cubes.push(arr5.gObject(color, 3));
                    Cubes.push(arr55.arrowHead(color,3));
                }
            }else{
                let arr5 = new Line([[0,0,0],[0,0,0]]);
                let arr55 = new Line([[0,0,0],[0,0,0]]);
                Cubes.push(arr5.gObject(color, 0));
                Cubes.push(arr55.arrowHead(color,0));
            }


            if (innerArr6){
                if (z<0){
                    let arr6 = new Line([[x+0.2,y,z+0.5],[x+0.2,y,z+1]]);
                    let arr66 = new Line([[x+0.2,y,z-0.5],[x+0.2,y,z+1]]);
                    Cubes.push(arr6.gObject(color, 3));
                    Cubes.push(arr66.arrowHead(color,3));
                }else{
                    let arr6 = new Line([[x,y,z+0.5],[x,y,z+1]]);
                    let arr66 = new Line([[x,y,z-0.5],[x,y,z+1]]);
                    Cubes.push(arr6.gObject(color, 3));
                    Cubes.push(arr66.arrowHead(color,3));
                }
            }else{
                let arr6 = new Line([[0,0,0],[0,0,0]]);
                let arr66 = new Line([[0,0,0],[0,0,0]]);
                Cubes.push(arr6.gObject(color, 0));
                Cubes.push(arr66.arrowHead(color,0));
            }

            return Cubes;
        }

        //create a blank object to added in frames so that some frames won't be shorter than the others
        function blank(data1, number){
            let data2 = []
            for (let i=0; i<number; ++i){
                data2.push({
                    type: "scatter3d",
                    mode: "lines",
                    x:[0,0],
                    y:[0,0],
                    z:[0,0],
                    lines: {width:0}
                })
            }
            data2 = data1.concat(data2)
            return data2;
        }

        //represents a 3d square with each side being highlighted
        function Surface3d(a1,b1,c1,a2,b2,c2,a3,b3,c3,a4,b4,c4){
            let edge1 = new Line([[a1,b1,c1],[a2,b2,c2]])
            let edge2 = new Line([[a2,b2,c2],[a3,b3,c3]])
            let edge3 = new Line([[a3,b3,c3],[a4,b4,c4]])
            let edge4 = new Line([[a4,b4,c4],[a1,b1,c1]])
            let square = [
            edge1.gObject(red,5),
            edge2.gObject(red,5),
            edge3.gObject(red,5),
            edge4.gObject(red,5),
            {
                    type: "mesh3d",
                    x: [a1,a2,a3,a4],
                    y: [b1,b2,b3,b4],
                    z: [c1,c2,c3,c4],
                    i : [0, 0],
                    j : [2, 3],
                    k : [1, 2],
                    opacity: 0.8,
                    colorscale: [['0', impBlue], ['1', impBlue]],
                    intensity: [0.8,0.8],
                    showscale: false
            }
            ]
            return square;
        }

        //creating frames for the animation
        function getCubes(frames){
            //create stops for the animation
            let stops = [0];
            let sep = numeric.linspace(0,0.5,20)

            //first frame -- a big cube with six arrows
            let bigCube = new outerCube(0.5,0.5,0.5,1)
            let data = new blank(bigCube, 200)
            frames.push({data: data});

            //second stage -- cutting the big cube to 8 same sized cubes
            for (let i = 0; i < 20; ++i){
                let Cube1 = new cubeAndArrow(0,0,0,black,green)
                let Cube2 = new cubeAndArrow(1+sep[i],0,0,black,green)
                Cube2 = Cube1.concat(Cube2)
                let Cube3 = new cubeAndArrow(0,1+sep[i],0,black, green)
                Cube3 = Cube2.concat(Cube3)
                let Cube4 = new cubeAndArrow(1+sep[i],1+sep[i],0,black,green)
                Cube4 = Cube3.concat(Cube4)
                let Cube5 = new cubeAndArrow(0,0,1+sep[i],black, green)
                Cube5 = Cube4.concat(Cube5)
                let Cube6 = new cubeAndArrow(1+sep[i],0,1+sep[i], black, green)
                Cube6 = Cube5.concat(Cube6)
                let Cube7 = new cubeAndArrow(0,1+sep[i],1+sep[i], black, green)
                Cube7 = Cube6.concat(Cube7)
                let Cube8 = new cubeAndArrow(1+sep[i],1+sep[i],1+sep[i],black, green)
                Cube8 = Cube7.concat(Cube8)
                let data = new blank(Cube8, 200)
                frames.push({data: data});
            }
            //make the animation stops itself after the second stage
            stops.push(frames.length - 1);

            //third stage -- starting from one small cube, adding one small cube each time, highlight the interface
            //make the arrows in between disappear and make the resulting cube change color

            //one cube
            let Cube1 = new cubeAndArrow(0,0,0,black,green)
            data = new blank(Cube1, 164)
            frames.push({data: data});


            //two cubes
            for (let i =0; i<22; ++i){
                if (i < 19){
                    let Cube2 = new cubeAndArrow(1.5-sep[i],0,0,lilac,green)
                    Cube2 = Cube1.concat(Cube2)
                    let data = new blank(Cube2, 151)
                    frames.push({data: data});
                }else{
                    let Cube1 = new cubeAndArrow(0,0,0,black,orange,true,false)
                    let Cube22 = new cubeAndArrow(1,0,0,black,orange,false)
                    let square = new Surface3d(0.51,-0.51,-0.51,0.51,0.51,-0.51,0.51,0.51,0.51,0.51,-0.51,0.51)
                    Cube22 = Cube1.concat(Cube22)
                    Cube22 = Cube22.concat(square)
                    let data = new blank(Cube22, 146)
                    frames.push({data: data});
                }
            }

            //three cubes
            for (let i =0; i<22; ++i){
                if (i < 19){
                    let Cube1 = new cubeAndArrow(0,0,0,black,orange,true,false)
                    let Cube22 = new cubeAndArrow(1,0,0,black,orange,false)
                    Cube22 = Cube1.concat(Cube22)
                    let Cube3 = new cubeAndArrow(0,1.5-sep[i],0,lilac, green)
                    Cube3 = Cube22.concat(Cube3)
                    let data = new blank(Cube3, 133)
                    frames.push({data: data});
                } else{
                    let Cube1 = new cubeAndArrow(0,0,0,black,orange,true,false,true,false)
                    let Cube2 = new cubeAndArrow(1,0,0,black,orange,false)
                    Cube2 = Cube1.concat(Cube2)
                    let Cube33= new cubeAndArrow(0,1,0,black,orange,true,true,false)
                    let square = new Surface3d(-0.51,0.51,0.51,0.51,0.51,0.51,0.51,0.51,-0.51,-0.51,0.51,-0.51)
                    Cube33 = Cube2.concat(Cube33)
                    Cube33 = Cube33.concat(square)
                    let data = new blank(Cube33, 128)
                    frames.push({data: data});
                }

            }

            //four cubes
            for (let i =0; i<22; ++i){
                if (i < 19){
                    let Cube1 = new cubeAndArrow(0,0,0,black,orange,true,false,true,false)
                    let Cube2 = new cubeAndArrow(1,0,0,black,orange,false)
                    Cube2 = Cube1.concat(Cube2)
                    let Cube33= new cubeAndArrow(0,1,0,black,orange,true,true,false)
                    Cube33 = Cube2.concat(Cube33)
                    let Cube4 = new cubeAndArrow(1.5-sep[i],1.5-sep[i],0,lilac,green)
                    Cube4 = Cube33.concat(Cube4)
                    let data = new blank(Cube4, 115)
                    frames.push({data: data});
                }else{
                    let Cube1 = new cubeAndArrow(0,0,0,black,orange,true,false,true,false)
                    let Cube2 = new cubeAndArrow(1,0,0,black,orange,false,true,true,false)
                    Cube2 = Cube1.concat(Cube2)
                    let Cube3 = new cubeAndArrow(0,1,0,black,orange,true,false,false)
                    Cube3 = Cube2.concat(Cube3)
                    let Cube44 = new cubeAndArrow(1,1,0,black,orange,false,true,false)
                    let square1 = new Surface3d(0.51,0.51,0.51,1.51,0.51,0.51,1.51,0.51,-0.51,0.51,0.51,-0.51)
                    let square2 = new Surface3d(0.51,0.51,0.51,0.51,1.51,0.51,0.51,1.51,-0.51,0.51,0.51,-0.51)
                    Cube44 = Cube3.concat(Cube44)
                    Cube44 = Cube44.concat(square1)
                    Cube44 = Cube44.concat(square2)
                    let data = new blank(Cube44, 105)
                    frames.push({data: data});
                }
            }


            //five cubes
            for (let i =0; i<22; ++i){
                if (i<19){
                    let Cube1 = new cubeAndArrow(0,0,0,black,orange,true,false,true,false)
                    let Cube2 = new cubeAndArrow(1,0,0,black,orange,false,true,true,false)
                    Cube2 = Cube1.concat(Cube2)
                    let Cube3 = new cubeAndArrow(0,1,0,black,orange,true,false,false)
                    Cube3 = Cube2.concat(Cube3)
                    let Cube44 = new cubeAndArrow(1,1,0,black,orange,false,true,false)
                    Cube44 = Cube3.concat(Cube44)
                    let Cube5 = new cubeAndArrow(0,0,1.5-sep[i],lilac, green)
                    Cube5 = Cube44.concat(Cube5)
                    let data = new blank(Cube5, 92)
                    frames.push({data: data});
                }else{
                    let Cube1 = new cubeAndArrow(0,0,0,black,orange,true,false,true,false,true,false)
                    let Cube2 = new cubeAndArrow(1,0,0,black,orange,false,true,true,false)
                    Cube2 = Cube1.concat(Cube2)
                    let Cube3 = new cubeAndArrow(0,1,0,black,orange,true,false,false)
                    Cube3 = Cube2.concat(Cube3)
                    let Cube4 = new cubeAndArrow(1,1,0,black,orange,false,true,false)
                    Cube4 = Cube3.concat(Cube4)
                    let Cube55 = new cubeAndArrow(0,0,1,black, orange,true,true,true,true,false)
                    let square = new Surface3d(0.51,-0.51,0.51,0.51,0.51,0.51,-0.51,0.51,0.51,-0.51,-0.51,0.51)
                    Cube55 = Cube4.concat(Cube55)
                    Cube55= Cube55.concat(square)
                    let data = new blank(Cube55, 87)
                    frames.push({data: data});
                }

            }


            //six cubes
            for (let i =0; i<22; ++i){
                if (i<19){
                    let Cube1 = new cubeAndArrow(0,0,0,black,orange,true,false,true,false,true,false)
                    let Cube2 = new cubeAndArrow(1,0,0,black,orange,false,true,true,false)
                    Cube2 = Cube1.concat(Cube2)
                    let Cube3 = new cubeAndArrow(0,1,0,black,orange,true,false,false)
                    Cube3 = Cube2.concat(Cube3)
                    let Cube4 = new cubeAndArrow(1,1,0,black,orange,false,true,false)
                    Cube4 = Cube3.concat(Cube4)
                    let Cube55 = new cubeAndArrow(0,0,1,black, orange,true,true,true,true,false)
                    Cube55 = Cube4.concat(Cube55)
                    let Cube6 = new cubeAndArrow(1.5-sep[i],0,1.5-sep[i], lilac, green)
                    Cube6 = Cube55.concat(Cube6)
                    let data = new blank(Cube6, 74)
                    frames.push({data: data});
                }else{
                    let Cube1 = new cubeAndArrow(0,0,0,black,orange,true,false,true,false,true,false)
                    let Cube2 = new cubeAndArrow(1,0,0,black,orange,false,true,true,false,true,false)
                    Cube2 = Cube1.concat(Cube2)
                    let Cube3 = new cubeAndArrow(0,1,0,black,orange,true,false,false)
                    Cube3 = Cube2.concat(Cube3)
                    let Cube4 = new cubeAndArrow(1,1,0,black,orange,false,true,false)
                    Cube4 = Cube3.concat(Cube4)
                    let Cube5 = new cubeAndArrow(0,0,1,black, orange,true,false,true,true,false)
                    Cube5 = Cube4.concat(Cube5)
                    let Cube66 = new cubeAndArrow(1,0,1,black,orange,false,true,true,true,false)
                    let square1 = new Surface3d(0.51,0.51,0.51,0.51,-0.51,0.51,0.51,-0.51,1.51,0.51,0.51,1.51)
                    let square2 = new Surface3d(0.51,0.51,0.51,0.51,-0.51,0.51,1.51,-0.51,0.51,1.51,0.51,0.51)
                    Cube66 = Cube5.concat(Cube66)
                    Cube66 = Cube66.concat(square1)
                    Cube66 = Cube66.concat(square2)
                    let data = new blank(Cube66, 64)
                    frames.push({data: data});
                }

            }


            //seven cubes
            for (let i =0; i<22; ++i){
                if(i<19){
                    let Cube1 = new cubeAndArrow(0,0,0,black,orange,true,false,true,false,true,false)
                    let Cube2 = new cubeAndArrow(1,0,0,black,orange,false,true,true,false,true,false)
                    Cube2 = Cube1.concat(Cube2)
                    let Cube3 = new cubeAndArrow(0,1,0,black,orange,true,false,false)
                    Cube3 = Cube2.concat(Cube3)
                    let Cube4 = new cubeAndArrow(1,1,0,black,orange,false,true,false)
                    Cube4 = Cube3.concat(Cube4)
                    let Cube5 = new cubeAndArrow(0,0,1,black, orange,true,false,true,true,false)
                    Cube5 = Cube4.concat(Cube5)
                    let Cube66 = new cubeAndArrow(1,0,1,black,orange,false,true,true,true,false)
                    Cube66 = Cube5.concat(Cube66)
                    let Cube7 = new cubeAndArrow(0,1.5-sep[i],1.5-sep[i],lilac, green)
                    Cube7 = Cube66.concat(Cube7)
                    let data = new blank(Cube7, 51)
                    frames.push({data: data});
                }else{
                    let Cube1 = new cubeAndArrow(0,0,0,black,orange,true,false,true,false,true,false)
                    let Cube2 = new cubeAndArrow(1,0,0,black,orange,false,true,true,false,true,false)
                    Cube2 = Cube1.concat(Cube2)
                    let Cube3 = new cubeAndArrow(0,1,0,black,orange,true,false,false,true,true,false)
                    Cube3 = Cube2.concat(Cube3)
                    let Cube4 = new cubeAndArrow(1,1,0,black,orange,false,true,false)
                    Cube4 = Cube3.concat(Cube4)
                    let Cube5 = new cubeAndArrow(0,0,1,black, orange,true,false,true,false,false)
                    Cube5 = Cube4.concat(Cube5)
                    let Cube6 = new cubeAndArrow(1,0,1,black,orange,false,true,true,true,false)
                    Cube6 = Cube5.concat(Cube6)
                    let Cube77 = new cubeAndArrow(0,1,1,black,orange,true,true,false,true,false)
                    let square1 = new Surface3d(0.51,0.51,0.51,-0.51,0.51,0.51,-0.51,0.51,1.51,0.51,0.51,1.51)
                    let square2 = new Surface3d(0.51,0.51,0.51,0.51,1.51,0.51,-0.51,1.51,0.51,-0.51,0.51,0.51)
                    Cube77 = Cube6.concat(Cube77)
                    Cube77 = Cube77.concat(square1)
                    Cube77 = Cube77.concat(square2)
                    let data = new blank(Cube77, 41)
                    frames.push({data: data});
                }
            }


            //eight cubes
            for (let i =0; i<22; ++i){
                if (i<19){
                    let Cube1 = new cubeAndArrow(0,0,0,black,orange,true,false,true,false,true,false)
                    let Cube2 = new cubeAndArrow(1,0,0,black,orange,false,true,true,false,true,false)
                    Cube2 = Cube1.concat(Cube2)
                    let Cube3 = new cubeAndArrow(0,1,0,black,orange,true,false,false,true,true,false)
                    Cube3 = Cube2.concat(Cube3)
                    let Cube4 = new cubeAndArrow(1,1,0,black,orange,false,true,false)
                    Cube4 = Cube3.concat(Cube4)
                    let Cube5 = new cubeAndArrow(0,0,1,black, orange,true,false,true,false,false)
                    Cube5 = Cube4.concat(Cube5)
                    let Cube6 = new cubeAndArrow(1,0,1,black,orange,false,true,true,true,false)
                    Cube6 = Cube5.concat(Cube6)
                    let Cube77 = new cubeAndArrow(0,1,1,black,orange,true,true,false,true,false)
                    Cube77 = Cube6.concat(Cube77)
                    let Cube8 = new cubeAndArrow(1.5-sep[i],1.5-sep[i],1.5-sep[i],lilac, green)
                    Cube8 = Cube77.concat(Cube8)
                    let data = new blank(Cube8, 28)
                    frames.push({data: data});
                }else{
                    let Cube1 = new cubeAndArrow(0,0,0,black,orange,true,false,true,false,true,false)
                    let Cube2 = new cubeAndArrow(1,0,0,black,orange,false,true,true,false,true,false)
                    Cube2 = Cube1.concat(Cube2)
                    let Cube3 = new cubeAndArrow(0,1,0,black,orange,true,false,false,true,true,false)
                    Cube3 = Cube2.concat(Cube3)
                    let Cube4 = new cubeAndArrow(1,1,0,black,orange,false,true,false,true,true,false)
                    Cube4 = Cube3.concat(Cube4)
                    let Cube5 = new cubeAndArrow(0,0,1,black, orange,true,false,true,false,false)
                    Cube5 = Cube4.concat(Cube5)
                    let Cube6 = new cubeAndArrow(1,0,1,black,orange,false,true,true,false,false)
                    Cube6 = Cube5.concat(Cube6)
                    let Cube7 = new cubeAndArrow(0,1,1,black,orange,true,false,false,true,false)
                    Cube7 = Cube6.concat(Cube7)
                    let Cube88 = new cubeAndArrow(1,1,1,black,orange,false,true,false,true,false)
                    let square1 = new Surface3d(0.51,0.51,0.51,0.51,0.51,1.51,0.51,1.51,1.51,0.51,1.51,0.51)
                    let square2 = new Surface3d(0.51,0.51,0.51,1.51,0.51,0.51,1.51,1.51,0.51,0.51,1.51,0.51)
                    let square3 = new Surface3d(0.51,0.51,0.51,1.51,0.51,0.51,1.51,0.51,1.51,0.51,0.51,1.51)
                    Cube88 = Cube7.concat(Cube88)
                    Cube88 = Cube88.concat(square1)
                    Cube88 = Cube88.concat(square2)
                    Cube88 = Cube88.concat(square3)
                    let data = new blank(Cube88, 30)
                    frames.push({data: data});
                }
            }


            //fourth stage -- after adding all eight cubes back, add the big cube on the outside
            Cube1 = new cubeAndArrow(0,0,0,black,orange,true,false,true,false,true,false)
            let Cube2 = new cubeAndArrow(1,0,0,black,orange,false,true,true,false,true,false)
            Cube2 = Cube1.concat(Cube2)
            let Cube3 = new cubeAndArrow(0,1,0,black,orange,true,false,false,true,true,false)
            Cube3 = Cube2.concat(Cube3)
            let Cube4 = new cubeAndArrow(1,1,0,black,orange,false,true,false,true,true,false)
            Cube4 = Cube3.concat(Cube4)
            let Cube5 = new cubeAndArrow(0,0,1,black, orange,true,false,true,false,false)
            Cube5 = Cube4.concat(Cube5)
            let Cube6 = new cubeAndArrow(1,0,1,black,orange,false,true,true,false,false)
            Cube6 = Cube5.concat(Cube6)
            let Cube7 = new cubeAndArrow(0,1,1,black,orange,true,false,false,true,false)
            Cube7 = Cube6.concat(Cube7)
            let Cube8 = new cubeAndArrow(1,1,1,black,orange,false,true,false,true,false)
            Cube8 = Cube7.concat(Cube8)
            let Cube9 = new outerCube(0.5,0.5,0.5,1)
            Cube9 = Cube9.concat(Cube8)
            data = new blank(Cube9, 100)
            frames.push({data: data});

            //return the array that represents stops
            return stops
        }

        //plots
        function initPlot(){
            // console.log("initPlot called");
            Plotly.purge("graph");
            let frames = [],

            //load frames and stops
            stops = getCubes(frames);

            //the slider for each frame
            $("#animateSlider").attr("max", frames.length - 1);
            //load animation
            initAnimation("animate", frames, [], layout, 10, stops, true);
            return;
        }

        //load main
        function main() {
            //slider
            $("input[type=range]").each(function () {
                $(this).on('input', function(){
                    $("#"+$(this).attr("id") + "Display").text( $(this).val() + $("#"+$(this).attr("id") + "Display").attr("data-unit") );
                    historyPlot(parseFloat($(this).val()));
                });
            });

            //animation
            // $("input[type=submit]").click(function () {
            //     // let idName = $(this).attr("id");
            //     startAnimation();
            // });
            if (v.frameNo == -1) {
                startAnimation();
            }
            
            initPlot();
            // startAnimation();
        }

        function redraw() {
            requestAnimationFrame(redraw);
            
            if (v.redrawPlot) {
                historyPlot(v.frameNo);
                v.redrawPlot = false;
            }
        }
        
        main();
        redraw();
    },
}
</script>

<style>
.center{
    display:flex;
    flex-direction: column;
    align-items: center;

}
</style>
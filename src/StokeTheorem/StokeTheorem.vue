<template>
    <div style="overflow: hidden;">
        <iv-visualisation :title="pageName" :vue_config="vue_config" :page_number="7">
            <template #hotspots>
                <iv-pane position="left" format="full">
                    <iv-sidebar-content :showPagination="true">
                        <iv-sidebar-section title="Stoke's Theorem" icon="book-open">
                            Stoke's theorem relates the integral of the curl of a vector field, <iv-equation-box class="in-line-eqn" :stylise="false" equation="\mathbf{V}" />, over an open surface <iv-equation-box class="in-line-eqn" :stylise="false" equation="S" /> to the path integral of the vector field around a loop <iv-equation-box class="in-line-eqn" :stylise="false" equation="C" /> defining the surface boundary. <br><br>

                            Mathematically, this can be written as 
                            <div class="center">
                                <iv-equation-box equation="\iint_{S} \nabla \times \vec{V} \cdot \vec{dS} = \oint_C \vec{V} \cdot \vec{dr}" />
                            </div>
                            Where <iv-equation-box class="in-line-eqn" :stylise="false" equation="dS" /> is the surface normal vector and <iv-equation-box class="in-line-eqn" :stylise="false" equation="dr" /> is the path tangent vector. <br><br>

                            As the right hand side of the equation is independent of the surface, the integral of the curl of a vector field over a surface is independent of the surface itself, but is only determined by the curve bounding it. <br><br>

                            Therefore, changing the surface by moving the first slider doesn't change the value of the integral, however, modifying the loop by moving the second slider does. <br><br>
                            
                        </iv-sidebar-section>
                    </iv-sidebar-content>
                </iv-pane>
                
                <iv-toggle-hotspot position="bottom" title="Properties">
                    <div style="width: 100%;">
                        <iv-slider id="sizeSlider" name="Size of surface" :min="0" :max="2" :step="0.1" :tick_step="0.2" :init_val="2" @sliderChanged="changeSize" />
                        <iv-slider id="shapeSlider" name="Shape of the loop" :min="2" :max="8" :step="1" :tick_step="2" :init_val="2" @sliderChanged="changeShape" />
                    </div>
                </iv-toggle-hotspot>

                <iv-fixed-hotspot position="right" transparent class="center">
                    <div>
                        <iv-button id="playButton" @click="togglePause">{{ buttonMessage }}</iv-button>
                    </div>
                </iv-fixed-hotspot>

            </template>

            <!-- Graph -->
            <div class="center" style="">
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

export default {
    name:"stoketheorem",
    data(){
        return {
            pageName:"StokeTheorem",
            vue_config,
            size: 2,
            shape: 2,
            frameNo: 0,
            redrawPlot: false,
            isPaused: true,
        };
    },
    methods: {
        changeSize(e) {
            this.size = e;
            this.redrawPlot = true;
        },
        changeShape(e) {
            this.shape = e;
            this.redrawPlot = true;
        },
        togglePause() {
            if (this.frameNo != 19) {
                this.isPaused = !this.isPaused;
            }
            
            
            if (!this.isPaused) {
                this.timer = setInterval(() => {
                    if (this.frameNo < 19) {
                        this.frameNo += 1;
                    } else {
                        clearInterval(this.timer);
                        this.isPaused = true;
                    }
                }, 100)
            } else {
                clearInterval(this.timer);
                if (this.frameNo == 19) {
                    this.frameNo = 0;
                    this.redrawPlot = true;
                    console.log("reset")
                }
            }
        },
    },
    computed: {
        buttonMessage() {
            console.log(this.frameNo);
            if (this.isPaused) {
                if (this.frameNo < 19) {
                    return "Play";
                } else {
                    return "Reset";
                }
            } 

            return "Pause";
        }
    },
    watch: {
        frameNo() {
            this.redrawPlot = true;
        }
    },
    mounted() {
        let v = this;
        // Colour definitions:
        let magenta = "#FF00FF",
            blue = "#0000FF",
            green = "#008000",
            // impBlue = "#003E74",
            black = "rgb(0,0,0)",
            white = "rgb(255,255,255)";
            // cyan = "rgb(0,146,146)", //1
            // lilac = "rgb(182,109,255)", //2
            // orange = "rgb(219,209,0)"; //3
            // red = "rgb(160, 0, 0)",
            // realOrange = "rgb(227,104, 0)";
        // Plot Utilities:
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
        * Represents a 3d pringle shaped loop with an arrow pointing anticlockwise.
        * @constructor
        * @param {float} radius - radius of the loop.
        * @param {array} centre - central point of the loop.
        * @param {float} height - distance of the centre of the loop away from the z=0 plane
        * @param {float} wave - number of periods of the sin wave on the loop, has to be even to close the loop smoothly, >= 2
        */
        function Pringles(radius, center) {
            this.radius = radius;
            this.center = center;


            this.gObject = function(color=black, width=7, dash="solid", height=1, wave=4,opacity=0.5) {
                let meshSize = 40; //multiple of 4! 40
                let phi = numeric.linspace(0, 2*Math.PI, meshSize);
                let theta = numeric.linspace(0,wave*Math.PI, meshSize);
                let x = [], y = [], z = [];

                for (let i=0, n=phi.length; i<n; ++i) {
                    x.push(this.radius*Math.cos(phi[i]) + this.center[0]);
                    y.push(this.radius*Math.sin(phi[i]) + this.center[1]);
                    z.push(height*(height*Math.sin(theta[i]) + this.center[2])+height);
                }
                
                let line2 = new Line([[x[0],y[0],z[0]],[x[7],y[7],z[7]]])
                let lineObject = [{
                    type: "scatter3d",
                    mode: "lines",
                    x: x,
                    y: y,
                    z: z,
                    line: {color: color, width: width, dash:dash},
                    surfaceaxis: 2,
                    surfacecolor: green,
                    opacity: opacity
                    },
                    line2.arrowHead(magenta,5)
                ]

                return lineObject;
            }

        }
        /**
        * Represents a hemisphere bounded by the pringle loop and an arrow pointing upwards.
        * @constructor
        * @param {float} radius - radius of the loop.
        */

        function Sphere(radius) {
            this.radius = radius;
            this.gObject = function(color1,color2, width=7, dash="solid", size, height=1, wave=4) { // eslint-disable-line no-unused-vars
                let meshSize = 40;
                let phi = numeric.linspace(0, 2*Math.PI, meshSize);
                let theta= numeric.linspace(0, 0.5*Math.PI, meshSize);
                let beta = numeric.linspace(0,wave*Math.PI, meshSize);

                this.x = [], this.y = [], this.z = [];
                for(let i = 0; i < meshSize; ++i) {
                    this.x[i] = [], this.y[i] = [], this.z[i] = [];
                    for(let j = 0; j < meshSize; ++j) {
                        this.x[i].push(this.radius*Math.cos(phi[i])*Math.sin(theta[j])+1);
                        this.y[i].push(this.radius*Math.sin(phi[i])*Math.sin(theta[j])+1);
                        this.z[i].push((size* this.radius*Math.cos(theta[j])+height*(Math.sin(theta[j])*Math.sin(theta[j])*Math.sin(theta[j]))*(height*Math.sin(beta[i]) + 1)*(Math.sin(theta[j]))+height));
                    }
                }

                let x1 = 1;
                let y1 = 1;
                let z1 = height*(Math.sin(0.5*Math.PI)*Math.sin(0.5*Math.PI)*Math.sin(0.5*Math.PI))*(height*Math.sin(beta[0]) + 1)*(Math.sin(0.5*Math.PI));
                let x2 = this.radius*Math.cos(phi[0])*Math.sin(theta[0])+1;
                let y2 = this.radius*Math.sin(phi[0])*Math.sin(theta[0])+1;
                let z2 =this.radius*Math.cos(theta[0])+(Math.sin(theta[0])*Math.sin(theta[0])*Math.sin(theta[0]))*(Math.sin(beta[0]) + 1)*(Math.sin(theta[0]))+6;
                let line1 = new Line([[x1,y1,z1],[x2,y2,z2]])
                let sphere = [{
                    type: "surface",
                    x: this.x,
                    y: this.y,
                    z: this.z,
                    showscale: false,
                    opacity: 0.6,
                    colorscale: [[0.0, color1], [1.0, color2]]
                },
                line1.gObject(black,3),
                line1.arrowHead(magenta,5)
                ]

                return sphere;
            }
        }

        //2D Objects:

        // Global variables:
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
            let data = [];
            for (let i = 0, n = animationFrames[0].data.length; i < n; ++i) {
                data.push({
                    type: animationFrames[0].data.type,
                });
            }
            for (let i = 0, n = extra.length; i < n; ++i) {
                data.push(extra[i]);
            }
            Plotly.newPlot("graph", data, (layout = layoutIn));
            reset();
        }

        /** resets animation. */
        function reset() {
            isPaused = true;
            animationIndex = 0;
            historyPlot(animationIndex);
            // document.getElementById(playID).value = isPaused ? "Play" : "Pause";
            updateSlider();
            return;
        }

        /**
        * plots index-th frame
        * @param {int} index - index of the frame
        */
        function historyPlot(index) {
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
            return;
        }

        /** Updates animation. */
        function update() {
            animationIndex++;
            if (animationIndex === animationLimit) {
                isPaused = true;
                // document.getElementById(playID).value = "Reset";
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
            $(sliderID).val(animationIndex);
            $(sliderID + "Display").text(animationIndex);
            v.frameNo = animationIndex;
        }

        /** Starts the animation. */
        function startAnimation() {
            if (animationIndex < animationLimit) {
                isPaused = !isPaused;
                // document.getElementById(playID).value = isPaused ? "Play" : "Pause";
                requestAnimationFrame(update);
            } else {
                reset();
            }
            return;
        }

        let layout = {
            width: 450, height: 500,
            margin: {l:0, r:0, t:0, b:0},
            hovermode: "closest",
            showlegend: false,
            scene: {
                xaxis: {range: [-4, 6], zeroline: true, autorange: false,},
                yaxis: {range: [-4, 6], zeroline: true, autorange: false,},
                zaxis: {range: [-6, 10], zeroline: true, autorange: false,},
                aspectratio: {x:1, y:1, z:1},
            }
        };
        let initialX = 2;
        let initialY = 2;

        function initPlot() {
            Plotly.purge("graph");

            //x slider controls the "size" parameter (height) of the sphere
            $("#xController").val(initialX);
            $("#xControllerDisplay").text(initialX);
            let x = v.size;

            //y slider controls the wave number(shape) of the sphere
            $("#yController").val(initialY);
            $("#yControllerDisplay").text(initialY);
            let y = v.shape;

            //initial plot with slider
            let data = [];

            let pringles = new Pringles(4, [1,1,1]);
            let cirface = new Sphere(4)

            data = data.concat(cirface.gObject(blue, white, 7, "solid", x,1,y))
            data = data.concat(pringles.gObject(black, 7, "solid",1,y));


            Plotly.newPlot("graph", data, layout);
            return;
        }

        //animation
        function updatePlot() {
            let data1 = [];
            let x = v.size;
            let y = v.shape;
            console.log(`x: ${v.size}, y: ${v.shape}`)
            let cirface = new Sphere(4)
            data1 = data1.concat(cirface.gObject(blue, blue, 7,"solid", x,1,y))
            let pringles = new Pringles(4, [1,1,1]);
            data1 = data1.concat(pringles.gObject(black, 7, "solid",1,y));



            let frames = [], data;

            //H is a set of height of the sphere
            let step1 = numeric.linspace(0, x, 20);
            let H = step1.reverse();


            //frames
            for (let i=0; i<20; ++i){
                if (i<19){
                    data = [];
                    let cirface = new Sphere(4)
                    data = data.concat(cirface.gObject(blue, blue, 7,"solid", H[i],1,y)) 
                    let pringles = new Pringles(4, [1,1,1]);
                    data = data.concat(pringles.gObject(black, 7, "solid", 1,y))

                }else{
                    //get rid of the sphere surface for the last frame
                    //have to push some empty objects to match the length of frames
                    data = [];
                    let pringles = new Pringles(4, [1,1,1]);
                    data = data.concat(pringles.gObject(black, 7, "solid", 1,y,1));
                    data.push({
                    type: "scatter3d",
                    mode: "lines",
                    x:[0,0],
                    y:[0,0],
                    z:[0,0],
                    lines: {width:0}
                    });
                    data.push({
                    type: "scatter3d",
                    mode: "lines",
                    x:[0,0],
                    y:[0,0],
                    z:[0,0],
                    lines: {width:0}
                    });
                    data.push({
                    type: "scatter3d",
                    mode: "lines",
                    x:[0,0],
                    y:[0,0],
                    z:[0,0],
                    lines: {width:0}
                    })
                }

                frames.push({data: data});
            }

            //load slider
            Plotly.animate(
                'graph',
                {data: data1},
                {
                    fromcurrent: true,
                    transition: {duration: 0,},
                    frame: {duration: 0, redraw: false,},
                    mode: "afterall"
                }
            );
            //load animation
            initAnimation("animate", frames, [], layout, 10, [0,20], true)
            console.log(frames);
        }

        //load main
        function main() {

            $("input[type=range]").each(function () {
                // let displayEl;
                $(this).on('input', function(){
                    $("#"+$(this).attr("id") + "Display").text( $(this).val() + $("#"+$(this).attr("id") + "Display").attr("data-unit") );
                    updatePlot();
                });

            });

            initPlot();
            updatePlot();
            if (v.frameNo == -1) {
                startAnimation();
            }
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
.center {
    display:flex;
    flex-direction: column;
    align-items: center;
    /* margin-top: 50px; */
}
.in-line-eqn {
    margin-top: -25px;
    margin-bottom: -25px;
}
</style>
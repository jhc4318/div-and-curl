<template>
    <div>
        <iv-visualisation :title="pageName" :vue_config="vue_config" :page_number="6">
            <template #hotspots>
                <iv-pane position="left" format="full">
                    <iv-sidebar-content :showPagination="true">
                        <iv-sidebar-section title="Green's Theorem">
                            Similarly, any arbitrary shape can be cut in to small pieces, with microscopic circulation existing on each of them. <br><br>

                            From the animation on the right hand side, every arrows inside the grid will cancel with an arrow with opposite
                            direction. In other words, all the <span style="color:#FF00FF"><b>magenta</b></span> arrows will
                            cancel each other and we are left with <span style="color:#000000"><b>black</b></span> arrows.<br><br>

                            This again demonstrates the relationship between the microscopic circulation and the macroscopic circulation. Therefore Green's theorem can
                            be apply on any positively oriented, piecewise smooth, simple closed arbitrary curves.              
                        </iv-sidebar-section>
                    </iv-sidebar-content>
                </iv-pane>

                <iv-fixed-hotspot position="bottom" transparent>
                    <iv-slider id="frameSlider" name="Frame #" :min="0" :max="9" :step="1" :tick_step="1" :init_val="0" :playButton="true" @sliderChangedbyPlay="changeSlider" @sliderChangedbyDragging="changeSlider" @sliderChangedbyClick="changeSlider" />
                </iv-fixed-hotspot>
            </template>

            <div class="center">
                <div id="graph" style="width:450px; height:450px;" />
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
    name:"generalisedgreentheorem",
    data(){
        return {
            pageName:"GeneralisedGreenTheorem",
            vue_config,
            isPaused: true,
            redrawPlot: false,
            frameNo: 0,
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
        // Colour definitions:
        let magenta = "#FF00FF",
            // blue = "#0000FF",
            // green = "#008000",
            // impBlue = "#003E74",
            black = "rgb(0,0,0)",
            // white = "rgb(255,255,255)",
            // cyan = "rgb(0,146,146)", //1
            // lilac = "rgb(182,109,255)", //2
            orange = "rgb(219,209,0)"; //3
            // orange2 = "rgb(240,150,0)"; //3

        // Plot Utilities:
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
            console.log("initAnimation called");
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
            console.log("reset called");
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
            console.log("historyPlot called");
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
            v.frameNo = animationIndex;
            $(sliderID + "Display").text(animationIndex);
        }

        /** Starts the animation. */
        function startAnimation() {
            console.log("startAnimation called");
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


        //represents 1/4 of the entire arbitrary shape
        //sep_x and sep_y are the distance of this 1/4 shape away from x axis and y axis
        function initArbitraryShape(phi1, phi2, sep_x, sep_y, x, y, innerArrow = true) {
            let Curve = []
            let xTemp = [], yTemp = [];
            //when combine the four parts, let innerArrow = false so the inner arrows can be hidden
            let innerColor = magenta;
            if(!innerArrow){
                innerColor = orange;
            }
            //add the side with straight lines
            let side ={
                type: "scatter",
                mode: "lines",
                x: [sep_x, sep_x, x+sep_x],
                y: [y+sep_y, sep_y, sep_y],
                line: {color: innerColor, width: 1},
                fill: 'toself',
                fillcolor: orange,
                opacity: 0.9
            };
            Curve.push(side)

            //add the curly side
            let Phi = numeric.linspace(phi1,phi2,20)
            for (let i=0; i<20; ++i){
                let radius = 1 - 0.7*(Math.cos(Phi[i]-Math.PI*0.25)*Math.sin(Phi[i]-Math.PI*0.25));
                xTemp.push(2*radius*Math.cos(Phi[i])+sep_x);
                yTemp.push(2*radius*Math.sin(Phi[i])+sep_y);
            }

            let curve ={
                    type: "scatter",
                    mode: "lines",
                    x: xTemp,
                    y: yTemp,
                    line: {color: black, width: 4},
                    fill: 'toself',
                    fillcolor: orange,
                    opacity: 0.9
                }
            Curve.push(curve)

            //add arrows on each side use if blocks to identify which 1/4 of the arbitrary shape is being drawn
            let half_x = (2*sep_x + x)*0.5
            let half_y = (2*sep_y + y)*0.5
            let grid1, grid2, y1, arr;
            if (sep_x*sep_y>0){
                grid1 = new Line2d([[sep_x,sep_y],[half_x, sep_y]])
                if (sep_x>0){
                    y1 = y + 1 + sep_y;
                }else if (sep_x<0){
                    y1 = y - 1 + sep_y;
                }
                grid2 = new Line2d([[sep_x, y1],[sep_x, half_y]])
                arr = new Line2d([[sep_x+x, sep_y],[xTemp[10],yTemp[10]]])
            }else if(sep_x*sep_y<0){
                grid1 = new Line2d([[x+sep_x,sep_y],[half_x, sep_y]])
                if (sep_y>0){
                    y1 = y - 1 + sep_y;
                }else if (sep_y<0){
                    y1 = y + 1 + sep_y;
                }
                grid2 = new Line2d([[sep_x, -y1],[sep_x, half_y]])
                arr = new Line2d([[sep_x, sep_y+y],[xTemp[10],yTemp[10]]])
            }
            Curve.push(arr.arrowHead(black,3))
            if (innerArrow){
                Curve.push(grid1.arrowHead(magenta,2));
                Curve.push(grid2.arrowHead(magenta,2));
            } else {
                Curve.push(grid1.arrowHead(magenta,0));
                Curve.push(grid2.arrowHead(magenta,0));
            }


            return Curve
        }

        // function blank(data1, number){
        //     for (let i=0; i<number; ++i){
        //         data1.push({
        //             type: "scatter",
        //             mode: "lines",
        //             lines: {width:0}
        //         })
        //     }
        //     return 0;
        // }


        //plot
        function updatePlot(){
            console.log("updatePlot called");
            let frames = [];
            //first frame -- four pieces being combined as an entire arbitrary shape
            let Curve1 = new initArbitraryShape(0, 0.5*Math.PI, 0.02, 0.02, 2.7, 1.3, false),
                Curve2 = new initArbitraryShape(0.5*Math.PI, Math.PI, -0.02, 0.02, -2.7, 1.3, false),
                Curve3 = new initArbitraryShape(Math.PI, 1.5*Math.PI, -0.02, -0.02, -2.7, -1.3, false),
                Curve4 = new initArbitraryShape(1.5*Math.PI, 2*Math.PI, 0.02, -0.02, 2.7, -1.3, false),
                curve = Curve4.concat(Curve2);
                curve = curve.concat(Curve1);
                curve = curve.concat(Curve3);

            frames.push({data: curve});

            //second stage -- separate the four part gradually
            for (let i=2; i<11; ++i){
                Curve1 = new initArbitraryShape(0, 0.5*Math.PI, 0.02*i, 0.02*i, 2.7, 1.3,true)
                Curve2 = new initArbitraryShape(0.5*Math.PI, Math.PI, -0.02*i, 0.02*i, -2.7, 1.3,true)
                Curve3 = new initArbitraryShape(Math.PI, 1.5*Math.PI, -0.02*i, -0.02*i, -2.7, -1.3,true)
                Curve4 = new initArbitraryShape(1.5*Math.PI, 2*Math.PI, 0.02*i, -0.02*i, 2.7, -1.3,true)
                curve = Curve4.concat(Curve2);
                curve = curve.concat(Curve1);
                curve = curve.concat(Curve3);

                frames.push({data: curve});
            }
            //load animation
            initAnimation("animate", frames, [], layout, 10)
        }



        //load main
        function main() {
            console.log("main called")
            updatePlot();
            // initGuidance(["graph","ani","green"]);
            if (v.frameNo == -1) {
                startAnimation();
            }
        }

        function redraw() {
            requestAnimationFrame(redraw);

            if (v.redrawPlot) {
                console.log("need redraw");
                // startAnimation();
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
</style>
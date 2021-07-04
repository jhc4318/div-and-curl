<template>
    <div>
        <iv-visualisation :title="pageName" :vue_config="vue_config" :page_number="4">
            <div class="center" style="padding-top: 50px;">
                <canvas id="graph" style="width:10px; height:10px; border: 5px solid black;"></canvas>
            </div>
        </iv-visualisation>
    </div>
</template>
<script>
import vue_config from '../../vue.config.js'
import P5 from 'p5'

export default {
    name:"circulation",
    data(){
        return {
            pageName:"Circulation",
            vue_config
        }
    },
    mounted() {   
        const script = p5 => {
            // These are your typical setup() and draw() methods
            p5.setup = () => {
                let canv = p5.createCanvas(p5.windowWidth, 600);
                canv.position(0, 10);
            };
            p5.draw = () => {
                p5.clear();
                p5.background(153);
                p5.ellipse(p5.mouseX, p5.mouseY, 40, 40);
            }; 
        }

        const p5canvas = new P5(script, 'canvas');
        console.log(p5canvas);
    },
}
</script>
<style>
.center{
    display: flex;
    flex-direction: column;
    place-items: center;
    /* padding-top: 100px; */
}
</style>
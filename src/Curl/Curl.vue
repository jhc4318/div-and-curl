<template>
    <div>
        <iv-visualisation :title="pageName" :vue_config="vue_config" :page_number="3">
            <template #hotspots>
                <iv-pane position="left" format="full">
                    <iv-sidebar-content :showPagination="true">
                        <iv-sidebar-section title="Curl" icon="book-open">
                            The curl of a vector field <iv-equation-box class="in-line-eqn" :stylise="false" equation="\mathbf{A}" /> is: <br>
                            <div class="center">
                                <iv-equation-box equation="\text{curl} \, \mathbf{A} = \nabla \times \mathbf{A}" />
                            </div>

                            <strong>N.B.</strong> The curl can be thought of as a measure of "spinning per unit area". <br><br>

                            Consider a 2D field <iv-equation-box class="in-line-eqn" :stylise="false" equation="\mathbf{A} = (P,Q,0)" />, we have the following definition: <br>
                            <div class="center">
                                <iv-equation-box equation="\iint_D (\nabla \times \mathbf{A}) \cdot \mathbf{\hat{k}} \,dS 
                                \\ = \\
                                \iint_D (\frac{\partial Q}{\partial x}(x, y) - \frac{\partial P}{\partial y}(x, y)) \,dx\,dy" />
                            </div>

                            This is known as <strong>microscopic circulation</strong>. <br><br>

                            The graph on the left hand side shows the spin of a small paddle wheel in two vector fields pointing in opposite directions. The speed and the direction of its spin depends on the strength and direction of the field where it is placed. <br><br>

                            <strong>N.B.</strong> Within the integrand, <iv-equation-box class="in-line-eqn" :stylise="false" equation="k" /> is the normal vector for the vector field and its direction is determined by the orientation of the closed curve, <iv-equation-box class="in-line-eqn" :stylise="false" equation="C" />. Commonly known as the <strong>right hand rule</strong>. A pictoral representation is shown below:
                            
                            <img src="../assets/rhr_thumb.png"
                            alt="Right hand rule"
                            style="display: flex; width: 100%;">
                        </iv-sidebar-section>

                        <iv-sidebar-section title="Instructions" theme="Lime">
                            The graph shows two vector fields pointing to the opposite direction. By putting your mouse anywhere on the graph, you can see the rotation of the paddle wheel under the effect of the vector field.
                            
                        </iv-sidebar-section>
                    </iv-sidebar-content>
                </iv-pane>
            </template>

            <!-- Graph -->
            <div class="center" id="graph" style="padding-top: 50px;">
    
            </div>
        </iv-visualisation>
    </div>
</template>
<script>
import vue_config from '../../vue.config.js'
import P5 from 'p5';

export default {
    name:"curl",
    data(){
        return {
            pageName:"Curl",
            vue_config
        }
    },
    mounted() {
        const script = p5 => {
            let arr1 = [];
            let arr2 = [];
            p5.setup = () => {
                let graph1 = p5.createCanvas(400,600);
                console.log(p5);
                // graph1.position(10, 10, 'fixed');
                //give graph1 an id
                graph1.parent("graph")
                //create a set of arrows pointing to the right
                for (let i=1; i<10; ++i){
                    for (let j=1; j<20; ++j){
                        arr1.push(new Arrow(20*i,20*j,j**0.25));

                }
                }
                //create a set of arrows pointing to the left
                for (let i=1; i<10; ++i){
                    for (let j=1; j<20; ++j){
                        arr2.push(new Arrow(20*i+200+0.5*j,20*j,j**0.25));

                }
                }
            }

            p5.draw = () => {
                p5.clear();
                // p5.background(153)
                //draw out all the arrows
                for (let k=0; k<arr1.length; ++k){
                    arr1[k].update();
                }
                for (let k=0; k<arr2.length; ++k){
                    arr2[k].update();
                }
                p5.push();

                //draw the paddle wheel, make it update with mouse position
                if (p5.mouseY < 400.5){
                    p5.translate(p5.mouseX, p5.mouseY);
                    if (p5.mouseX<200.5){
                        p5.rotate(-p5.frameCount*p5.mouseY / 2000.0);
                        p5.fill(200,10,255)
                    }else if (p5.mouseX>200.5){
                        p5.rotate(p5.frameCount*p5.mouseY / 2000.0);
                        p5.fill(200,255,10)
                    }
                    star(0, 0, 5, 20, 4);
                    p5.pop();
                }

                //draw the dot and cross that represent the direction of the curl
                p5.push();
                p5.fill(255,255,255)
                p5.arc(100, 500, 40, 40, 0, p5.TWO_PI)
                p5.push();
                p5.fill(200,10,255)
                p5.ellipse(100,500,20,20)

                p5.push();
                p5.fill(255,255,255)
                p5.arc(300, 500, 40, 40, 0, p5.TWO_PI)

                p5.push();
                p5.translate(300,500)
                p5.fill(200,255,10)
                p5.rotate(p5.QUARTER_PI);
                p5.rect(-15,-3,30,5);
                p5.pop();

                p5.push();
                p5.translate(300,500)
                p5.fill(200,255,10)
                p5.rotate(p5.PI-p5.QUARTER_PI);
                p5.rect(-15,-3,30,5);

            }

            /**
             * Represents an arrow.
             * @constructor
             * @param {float} x - x position of the first point on the arrow.
             * @param {float} y - y position of the first point on the arrow.
             * @param {float} length - define the size of the arrow.
             */
            function Arrow(x,y,length){
                this.x = x;
                this.y = y;
                this.length = length;
                this.update = function(){
                p5.push();
                p5.translate(this.x,this.y);
                if (x<200.5){
                    p5.rotate(0)
                    p5.fill(0,100,255)
                }else if (x>200.5){
                    p5.rotate(p5.PI)
                    p5.fill(255,100,0)
                }
                p5.beginShape();
                p5.vertex(0, -this.length);
                p5.vertex(5*this.length,-this.length);
                p5.vertex(5*this.length,-3*this.length);
                p5.vertex(9*this.length,0);
                p5.vertex(5*this.length,3*this.length);
                p5.vertex(5*this.length,this.length);
                p5.vertex(0,this.length);
                p5.endShape(p5.CLOSE);
                p5.pop();
                }
            }


            /**
             * Represents a star.
             * @constructor
             * @param {float} x - x position of the center of the star.
             * @param {float} y - y position of the center of the star.
             * @param {float} radius1 - length of the centre of the star to the inner p5.vertex.
             * @param {float} radius2 - length of the centre of the star to the outer p5.vertex.
             * @param {float} npoints - number of the points of the star.
             */
            function star(x,y,radius1, radius2, npoints) {
                let angle = p5.TWO_PI / npoints;
                let halfAngle = angle/2.0;
                p5.beginShape();
                for (let a = 0; a < p5.TWO_PI; a += angle) {
                    let sx = x + p5.cos(a) * radius2;
                    let sy = y + p5.sin(a) * radius2;
                    p5.vertex(sx, sy);
                    sx = x + p5.cos(a+halfAngle) * radius1;
                    sy = y + p5.sin(a+halfAngle) * radius1;
                    p5.vertex(sx, sy);
                }
                p5.endShape(p5.CLOSE);
            }
        }

        const p5graph = new P5(script, 'graph');
        console.log(p5graph);
    }
}
</script>
<style>
.center {
    display:flex;
    flex-direction: column;
    align-items: center;
}
.in-line-eqn {
    margin-top: -25px;
    margin-bottom: -25px;
}
</style>
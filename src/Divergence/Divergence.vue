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
            </template>
        </iv-visualisation>
    </div>
</template>
<script>
import vue_config from '../../vue.config.js'
// import MathJax from 'mathjax'

export default {
    name: "divergence",
    data() {
        return {
            pageName:"Divergence",
            vue_config,
        };
    },
    mounted() {
        // let v = this;
        // MathJax.typeset();
    }

}
</script>
<style>
.center{
    display:flex;
    flex-direction: column;
    align-items: center;

}
</style>
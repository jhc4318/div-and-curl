<template>
    <div>
        <iv-visualisation :title="pageName" :vue_config="vue_config" :page_number="1">
            <template #hotspots>
                <iv-pane position="left" format="full">
                    <iv-sidebar-content :showPagination="true">
                        <iv-sidebar-section title="Welcome" icon="book-open">
                            <strong>Welcome to an overview of the fundamental theorems of vector calculus!</strong><br><br>

                            Table of Contents
                            <ol>
                                <li>Introduction</li>
                                <li>Divergence</li>
                                <li>Curl</li>
                                <li>Circulation</li>
                                <li>Green's Theorem</li>
                                <li>Generalised Green's Theiren</li>
                                <li>Stoke's Theorem</li>
                            </ol>

                            Click "Next" to start looking at the visualisations or use the numbers at the top left to find a specific one.
                        </iv-sidebar-section>  
                    </iv-sidebar-content>
                </iv-pane>             
            </template>

            <div class="center" style="width: 60%; height: 60%; margin: auto; padding-top: 40vh;">
                <img src="../assets/ImpVis-logo.png" alt="placeholder">
            </div>  
        </iv-visualisation>
    </div>
</template>
<script>
import vue_config from '../../vue.config.js'
export default {
    name:"introduction",
    data(){
        return {
            pageName:"Introduction",
            vue_config
        }
    }
}
</script>
<style>
.center{
    display:flex;
    flex-direction: column;
    align-items: center;
    /* margin-top: 50px; */
}
</style>
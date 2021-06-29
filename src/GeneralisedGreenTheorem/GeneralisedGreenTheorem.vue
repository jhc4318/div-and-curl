<template>
    <div>
        <iv-visualisation :title="pageName" :vue_config="vue_config" :page_number="6">
            <div class="iv-welcome-message">
                <img src='@/assets/ImpVis-logo.png' alt="ImpVisLogo" height="50"/>
                <h1> Welcome to Imperial Visualisations!</h1>
                <p> Your page, GeneralisedGreenTheorem has succesfully been set up using the multiPage CLI template!</p>
            </div>
        </iv-visualisation>
    </div>
</template>
<script>
import vue_config from '../../vue.config.js'
export default {
    name:"generalisedgreentheorem",
    data(){
        return {
            pageName:"GeneralisedGreenTheorem",
            vue_config
        }
    }
}
</script>
<style>
.iv-welcome-message{
    display:flex;
    flex-direction: column;
    align-items: center;
    margin-top: 50px;
}
</style>
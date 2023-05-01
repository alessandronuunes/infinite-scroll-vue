<template>
    <div id="app">
        <InfiniteScroll class="infi-scroll"> </InfiniteScroll>
    </div>
</template>

<script setup>
import { defineAsyncComponent } from "vue";

const InfiniteScroll = defineAsyncComponent(() => import("./components/InifiniteScroll.vue"));

</script>

<style lang="scss">
#app {
    height: 80vh;
    margin-top: 10%;

    font-family: "Avenir", Helvetica, Arial, sans-serif;

    .infi-scroll {
        width: 100%;
        margin-left: auto;
        margin-right: auto;
        height: 100%;
    }
}
</style>

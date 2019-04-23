<template>
    <div id="app">
        <WaterFall :scale="imgScale" :loadConfig="loadConfig"></WaterFall>
    </div>
</template>

<script>
    import WaterFall from './tpls/vue-waterfall-x/index.vue';

    export default {
        name: 'App',
        components: {
            WaterFall
        },
        data () {
            return {
                imgScale: 0.333, // 卡片宽度（表示占屏幕占比，必须为小数）
                loadConfig: {
                    type: 'get', // 请求方式
                    url: '/front/project/list.json', // 请求接口
                    pageSize: 12  // 每页请求的数据量
                }
            }
        },
        created () {
        },
        mounted (){

        },
        methods: {

        }
    }
</script>

<style>
    @import './assets/css/main.css';
</style>
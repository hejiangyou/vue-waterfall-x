<template>
    <div class="waterfallCont" >
        <div v-for="(item, index) in list" class="cont" :key="index"
             :style="{width: (scale * 100) + '%', height: item.style.height + 'px', top: item.style.top + 'px', left: item.style.left + 'px', opacity: item.style.opacity}"
             ref="waterList">
            <img :src="item.cover" class="cover">
            <p class="name">{{item.title}}</p>
        </div>
        <div class="loadingComponent" v-show="loadingOk"></div>
    </div>
</template>

<script>
    import Axios from 'axios';

    export default {
        name: '',
        props: {
            scale: {
                type: Number,
                required: false,
                default: 0.5
            },
            loadConfig: {
                type: Object,
                required: true,
                default: {
                    type: 'get',
                    url: '',
                    pageSize: 10
                }
            }
        },
        data () {
            return {
                list: [],
                nodeList: [],
                positionArr: [],
                loadedNum: 0,
                loadedData: [],
                timer: null,
                loadingOk: true,
                page: 0
            }
        },
        created () {

        },
        mounted () {
            this.loadMoreFn();
            window.addEventListener('scroll', ()=>{
                this.scrollLoadFn();
            });
        },
        methods: {
            // 滚动函数
            scrollLoadFn (evt, el) {
                function throttle(method, context) {
                    clearTimeout(method.tId);
                    method.tId = setTimeout(function () {
                        method.call(context);
                    }, 500);
                };
                throttle(() => {
                    if (this.judegBomFn() && !this.loadingOk) {
                        this.loadMoreFn();
                    } else {
                        return false;
                    }
                });
            },
            // 判断滚动高度
            judegBomFn () {
                var documentHeight = document.body.scrollHeight;
                var winHeight = document.documentElement.clientHeight;
                var scrollHeight = document.documentElement.scrollTop || document.body.scrollTop;
                return (documentHeight - winHeight) <= scrollHeight;
            },
            // 图片预加载
            imgListPreLoadFn () {
                const _this = this;
                this.list = this.list.concat(this.loadedData); // 并入总数据
                for (let i = 0; i < this.loadedData.length; i++) {
                    let Img = new Image();
                    Img.src = this.loadedData[i]['cover'];
                    this.loadedData[i]['style'] = {};
                    Img.onload = function (e) {
                        _this.loadedNum++;
                    }
                }
            },
            // 计算每张图片的顶部位置
            calculationTopFn () {
                this.nodeList = this.$refs.waterList;
                const winWidth = document.documentElement.clientWidth;
                const ImgWidth = winWidth * this.scale;
                const num = parseInt(winWidth / ImgWidth); // 一行可以容下的图片数量
                for (let i = 0; i < this.list.length; i++) {
                    this.list[i]['style']['height'] = this.nodeList[i].offsetHeight;
                    if (i < num) {
                        // 第一行
                        this.list[i]['style']['left'] = (i % num) * ImgWidth;
                        this.list[i]['style']['top'] = 0;
                        this.positionArr[i] = this.nodeList[i].offsetHeight;
                    } else {
                        // 从第二行开始
                        let minHeightTop = Math.min.apply(null, this.positionArr);
                        this.list[i]['style']['top'] = minHeightTop;
                        // 获取最小值的索引，并更新
                        let minIndex = minHeightIndex(this.positionArr, minHeightTop);
                        this.list[i]['style']['left'] = minIndex * ImgWidth;
                        this.positionArr[minIndex] += this.nodeList[i].offsetHeight;
                    }
                    this.list[i]['style']['opacity'] = 1;
                }
                // 获取最小高度盒子的索引值
                function minHeightIndex(positionArr, minHeightTop) {
                    for (var i in positionArr) {
                        if (positionArr[i] == minHeightTop) {
                            return i;
                        }
                    }
                }
            },
            // 加载更多
            loadMoreFn () {
                this.loadingOk = true;
                this.page++;
                let req = {
                    method: this.loadConfig.type,
                    url: this.loadConfig.url
                };
                if (req.method == 'get') {
                    req['params'] = {
                        page: this.page,
                        pageSize: this.loadConfig.pageSize
                    };
                } else {
                    req['data'] = {
                        pageNum: this.page,
                        pageSize: this.loadConfig.pageSize
                    };
                }
                Axios(req).then((res) => {
                    this.loadedNum = 0; // 重置
                    this.loadedData = []; // 重置
                    let data = res.data.data.contents;
                    this.loadedData = JSON.parse(JSON.stringify(data));
                    this.imgListPreLoadFn();
                    this.judegImgLoadStatusFn();
                });
            },
            // 判断当前数据图片的加载状态
            judegImgLoadStatusFn () {
                this.timer = setInterval(() => {
                    if (this.loadedNum == this.loadedData.length) {
                        this.calculationTopFn();
                        this.loadingOk = false;
                        clearInterval(this.timer);
                        this.timer = null;
                    }
                }, 5);
            }
        }
    }
</script>

<style>
    .waterfallCont {
        position: relative;
        width: 100%;
        height: auto;
    }

    .cont {
        position: absolute;
        opacity: 0;
        background-color: #efefef;
        transition: all 0.5s ease;
        -webkit-transition: all 0.5s ease;
        -moz-transition: all 0.5s ease;
    }

    .cont .cover {
        display: block;
        width: 100%;
        height: auto;
    }

    .cont .name {
        display: block;
        width: 100%;
        height: auto;
        line-height: 20px;
        font-size: 12px;
        padding: 2px 8px;
        box-sizing: border-box;
    }

    /*loading*/
    .loadingComponent {
        margin-top: 40px;
        width: 16px;
        height: 16px;
        border-radius: 50%;
        display: inline-block;
        position: fixed;
        left: 50%;
        top: 50%;
        margin: -8px 0 0 -8px;
        animation: 3s infinite linear;
        -webkit-animation: 3s infinite linear;
        -ms-animation: 3s infinite linear;
    }

    .loadingComponent::after {
        content: '';
        width: 16px;
        height: 16px;
        border-radius: 50%;
        display: inline-block;
        position: absolute;
        left: 50%;
        top: 50%;
        margin: -8px 0 0 -8px;
        animation: 3s infinite linear;
        -webkit-animation: 3s infinite linear;
        -ms-animation: 3s infinite linear;
    }

    .loadingComponent::before {
        content: '';
        width: 16px;
        height: 16px;
        border-radius: 50%;
        display: inline-block;
        position: absolute;
        left: 50%;
        top: 50%;
        margin: -8px 0 0 -8px;
        animation: 3s infinite linear;
        -webkit-animation: 3s infinite linear;
        -ms-animation: 3s infinite linear;
    }

    .loadingComponent::before {
        background: #E84C3D;
        animation: loader2 1.2s infinite linear;
        -webkit-animation: loader2 1.2s infinite linear;
        -ms-animation: loader2 1.2s infinite linear;
    }

    .loadingComponent {
        background: #F1C40F;
        z-index: 100;
    }

    .loadingComponent::after {
        background: #2FCC71;
        animation: loader 1.2s infinite linear;
        -webkit-animation: loader 1.2s infinite linear;
        -moz-animation: loader 1.2s infinite linear;
        -ms-animation: loader 1.2s infinite linear;
    }

    @keyframes loader {
        0% {
            transform: translateX(20px);
        }

        50% {
            transform: translateX(-20px);
        }

        100% {
            transform: translateX(20px);
            z-index: 200;
        }
    }

    @-webkit-keyframes loader {
        0% {
            -webkit-transform: translateX(20px);
        }

        50% {
            -webkit-transform: translateX(-20px);
        }

        100% {
            -webkit-transform: translateX(20px);
            z-index: 200;
        }
    }

    @-ms-keyframes loader {
        0% {
            -moz-transform: translateX(20px);
        }

        50% {
            -moz-transform: translateX(-20px);
        }

        100% {
            -moz-transform: translateX(20px);
            z-index: 200;
        }
    }

    @keyframes loader2 {
        0% {
            transform: translateX(-20px);
            z-index: 200;
        }
        50% {
            transform: translateX(20px);
        }
        100% {
            transform: translateX(-20px);
        }
    }

    @-webkit-keyframes loader2 {
        0% {
            -webkit-transform: translateX(-20px);
            z-index: 200;
        }
        50% {
            -webkit-transform: translateX(20px);
        }
        100% {
            -webkit-transform: translateX(-20px);
        }
    }

    @-moz-keyframes loader2 {
        0% {
            -webkit-transform: translateX(-20px);
            z-index: 200;
        }
        50% {
            -webkit-transform: translateX(20px);
        }
        100% {
            -webkit-transform: translateX(-20px);
        }
    }

    @-ms-keyframes loader2 {
        0% {
            -moz-transform: translateX(-20px);
            z-index: 200;
        }
        50% {
            -moz-transform: translateX(20px);
        }
        100% {
            -moz-transform: translateX(-20px);
        }
    }
</style>
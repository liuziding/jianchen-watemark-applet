<template>
    <div class="pic-map" @touchmove.stop.prevent="" ref="picMap" @touchstart="touchstart" @touchmove="touchmove"
        @touchend="touchend">
        <div class="map-img" ref="mapImg" :style="{
            width:width,
            height:height,
            left:left,
            top:top
        }">
            <image class="map-pic" :style="{
            width:width,
            height:height,
        }" :src="picPath" mode="aspectFit" @load="onImageLoad"></image>
            <image class="map-marker" :style="{
                'left':item.longitude *  (width / parseFloat(imgSize.width) ),
                'top':(imgSize.height - item.latitude) *  (width / parseFloat(imgSize.width) )
            }" src="/static/default.png" v-for="(item,index) in facilities" :key="index"></image>
        </div>
    </div>
</template>
 
<script>
    const dom = weex.requireModule('dom')
    export default {
        data() {
            return {
                facilities: [],
                picPath: '',
                height: 0,
                width: 0,
                left: 0,
                top: 0,
                shrinkRatio: 1,
                imgSize: {
                    height: 0,
                    width: 0
                },
                touch: {
                    startx: 0,
                    starty: 0,
                    oLeft: 0,
                    oTop: 0,
                    isMove: true,
                    start: []
                },
                points: []
            };
        },
        components: {},
        computed: {},
        onLoad() {
            let _this = this;
            _this.$nextTick(function() {
                dom.getComponentRect(_this.$refs.picMap, option => {
                    _this.width = option.size.width;
                })
            })
        },
        onReady() {
            let _this = this;
            uni.$on('getPicPoints', data => {
                _this.facilities = data.facilities;
                _this.picPath = data.picPath
            })
        },
        onUnload() {
            uni.$off('selectFactory')
        },
        onShow() {},
        onReachBottom() {},
        methods: {
            onImageLoad(e) {
                let _this = this;
                dom.getComponentRect(_this.$refs.picMap, option => {
                    _this.width = option.size.width;
 
                    // 显示宽度 对比实际图片宽缩小倍数
                    _this.shrinkRatio = _this.width / parseFloat(e.detail.width)
                    // 实际高度 * 缩小倍率 = 展示高度
                    _this.height = e.detail.height * _this.shrinkRatio
                    // 记录图片实际宽高
                    _this.imgSize = {
                        height: e.detail.height,
                        width: e.detail.width
                    }
                })
            },
            touchstart(e) {
                let _this = this;
                //判断手指数
                let ttg = e.changedTouches[0];
 
                if (!ttg || !ttg.pageX) {
                    return;
                }
 
                //计算相对的位置
                this.touch.startx = ttg.pageX
                this.touch.starty = ttg.pageY
 
                dom.getComponentRect(_this.$refs.mapImg, option => {
                    _this.touch.oLeft = option.size.left;
                    _this.touch.oTop = option.size.top;
                })
 
                if (ttg.identifier >= 1) {
                    _this.touch.isMove = false
                } else {
                    _this.touch.isMove = true
                    _this.touch.start = [];
                }
 
                _this.touch.start.push(ttg);
            },
            touchmove(e) {
                let _this = this;
                if (_this.touch.isMove) {
                    let ttg = e.changedTouches[0];
 
                    if (!ttg || !ttg.pageX) {
                        return;
                    }
 
                    let x = ttg.pageX - _this.touch.startx;
                    let y = ttg.pageY - _this.touch.starty;
 
                    _this.left = x + _this.touch.oLeft;
                    _this.top = y + _this.touch.oTop;
                } else if (!_this.touch.isMove && _this.touch.start.length >= 2) {
                    //得到第二组两个点
                    let now = e.changedTouches;
                    //得到缩放比例， getDistance 是勾股定理的一个方法
                    let p2 = now[1];
                    let p1 = now[0];
 
                    if (!p2 || !p1 || !p2.pageX || !p1.pageX) {
                        return;
                    }
 
                    let x = (p2.pageX - p1.pageX)
                    let y = (p2.pageY - p1.pageY)
                    let value01 = Math.sqrt((x * x) + (y * y))
 
                    p2 = _this.touch.start[1]
                    p1 = _this.touch.start[0]
 
                    if (!p2 || !p1 || !p2.pageX || !p1.pageX) {
                        return;
                    }
 
                    x = (p2.pageX - p1.pageX)
                    y = (p2.pageY - p1.pageY)
                     
                    let value02 = Math.sqrt((x * x) + (y * y))
                    let scale = (value01 / value02);
                     
                    scale = scale.toFixed(2);
                    scale = (scale - 1) / 30 + 1;
 
                    let h = _this.height;
                    let w = _this.width;
 
                    if (h * scale > _this.imgSize.height) {
                        return false;
                    }
 
                    if (h * scale < 100 && w * scale < 100) {
                        return false;
                    }
 
                    _this.height = h * scale
                    _this.width = w * scale
                }
            },
            touchend(e) {
                let _this = this;
                let ttg = e.changedTouches[0];
 
                if (ttg.identifier >= 1) {
                    _this.touch.start = [];
                }
            }
        }
    };
</script>
 
<style lang="scss" scoped>
    .pic-map {
        background-color: rgba(155, 155, 155, 0.9);
 
    }
 
    .map-img {
        position: fixed;
 
        .map-pic {}
 
        .map-marker {
            position: absolute;
            width: 37.5px;
            height: 50px;
            margin-top: -50px;
            margin-left: -18.25px;
        }
    }
</style>

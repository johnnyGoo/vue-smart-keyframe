<template>
    <div ref="dom" :style="{position: 'absolute'}">
        <slot></slot>
    </div>
</template>
<style scoped>

</style>

<script>
    import keyframe from './keyframes'

    // 注册

    let defaultAttr = {opacity:1,x: 0, y: 0, z: 0, rotateX: 0, rotateY: 0, rotateZ: 0, scaleX: 1, scaleY: 1, scaleZ: 1}


    function buildKeys(frame, lastFrame) {

        for (let key in defaultAttr) {
            frame['_' + key] = (lastFrame && lastFrame.hasOwnProperty(key)) ? lastFrame[key] : frame[key] | defaultAttr[key]
            frame[key] = frame.hasOwnProperty(key) ? frame[key] : defaultAttr[key]
        }

        return frame;
    }

    function getValues(frame, d) {
        let obj = {}
        for (let key in defaultAttr) {
            obj[key] = d * (frame[key] - frame['_' + key]) + frame['_' + key];
        }
        return obj
        //d * (frame.x - frame._x) + frame._x, d * (frame.y - frame._y) + frame._y, d * (frame.rotateZ - frame._rotateZ) + frame._rotateZ

    }


    export default {
        name: 'vue-smart-keyframe',
        // 声明 props
        props: {
            time: {
                type: Number,
                defalut: 1
            },
            value: {
                type: Number,
                defalut: 0
            },
            base: {
                type: Number,
                defalut: 0
            },
            frames: {
                type: Array,
                defalut: function () {
                    return []
                }
            }
        },
        data: function () {
            return {
                __time: 1,
                updateFun: function () {

                }
            }
        },
        watch: {
            value: function (o, n) {

                this.update(n);
            }
        },
        methods: {
            update: function () {
                this.updateFun((this.value - this.base) / this.__time)

            }
        },
        mounted: function () {
            let self = this;
            self.__time = self.time | 1;

// From (0% -> 50%) move the div left 150px
// then from (50% -> 100%) move the div up 50px.

            function update(x, y, z, rotateX, rotateY, rotateZ, scaleX, scaleY, scaleZ,opacity) {
                self.$refs.dom.style.transform = `translateX(${x}px) translateY(${y}px) translateZ(${z}px) scaleX(${scaleX}) scaleY(${scaleY}) scaleZ(${scaleZ}) rotateX(${rotateX}deg) rotateY(${rotateY}deg) rotateZ(${rotateZ}deg)`;
                self.$refs.dom.style.transition = `transform 0.1s`;
                self.$refs.dom.style.opacity = opacity;

            }

            let frames = {};
            let frame, lastFrame;

            for (let i = 0; i < self.frames.length; i++) {
                frame = self.frames[i];
                buildKeys(frame, lastFrame);

                function buildFun(frame) {
                    function FrameFun(d) {
                        let obj = getValues(frame, d);
                        update(obj.x, obj.y, obj.z, obj.rotateX, obj.rotateY, obj.rotateZ, obj.scaleX, obj.scaleY, obj.scaleZ,obj.opacity)
                    }

                    return FrameFun
                }

                frames[frame.key] = buildFun(frame)
                if (i === 0) {
                    (function(frame) {

                            self.$nextTick(function () {
                                self.updateFun(0.01)
                                update(frame._x, frame._y, frame._z, frame._rotateX, frame._rotateY, frame._rotateZ, frame._scaleX, frame._scaleY, frame._scaleZ,frame._opacity)
                            })

                    })(frame);

                }
                lastFrame = frame

            }


            self.updateFun = keyframe(frames);

            self.$nextTick(function () {
                setTimeout(function () {
                    // self.$refs.dom.style.display='none'
                    // self.updateFun(0)
                    //   self.update()
                })
            })


        }
    }
</script>
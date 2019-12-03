<template>
    <div class="scrollbar" :style="{height,width}">
        <div class="scrollbar-content" :style="{width: `calc(${contentWidth} + ${getScrollWidth()}px)`}" @scroll="handleScroll">
            <div class="scrollbar-box">
                <slot/>
            </div>
        </div>
        <div class='scrollbar-bar' @click="handleClick">
            <div class="scrollbar-thumb" @mousedown="mousedownHandle" :style="{height: thumbHeight + '%', transform: 'translateY(' + moveY + '%)'}"></div>
        </div>
    </div>
</template>

<script>
    export default {
        name: "Scrollbar",
        props: {
            height: {
                type: String,
                required: true
            },
            width: {
                type: String,
                required: true
            }
        },
        computed:{
            contentWidth() {
                return this.width.includes("%") ? '100%' : this.width
            }
        },
        data() {
            return {
                cursorDown: false, //是否按下滚动条
                clickThumbAxis: 0, //鼠标点击滚动条的位置,
                thumbHeight: 0,
                moveY:0,
                wrap:null,
                bar: null,
                thumb:null,
                box:null
            }
        },
        mounted() {
            this.wrap = document.getElementsByClassName("scrollbar-content")[0];
            this.bar = document.getElementsByClassName("scrollbar-bar")[0];
            this.thumb = document.getElementsByClassName("scrollbar-thumb")[0]
            this.box = document.getElementsByClassName("scrollbar-box")[0]
            this.updateThumb()
        },
        methods: {
            getScrollWidth() {
                const outer = document.createElement("div");
                outer.style.width = '100px';
                outer.style.visibility = "hidden";
                outer.style.position = "absolute";
                outer.style.top = "-9999px";
                document.body.appendChild(outer);

                const widthNoScroll = outer.offsetWidth;
                outer.style.overflow = "scroll";

                const inner = document.createElement("div");
                inner.style.width = "100%";
                outer.appendChild(inner);

                const widthWithScroll = inner.offsetWidth;
                outer.parentNode.removeChild(outer);
                return widthNoScroll - widthWithScroll
            },

            updateThumb() {
                let heightPercentage = (this.wrap.clientHeight * 100 / this.box.scrollHeight);
                heightPercentage = heightPercentage > 100 ? 100 : heightPercentage
                this.thumbHeight = heightPercentage
            },
            mousedownHandle (e) {
                this.startDrag(e);
                this.clickThumbAxis = e.currentTarget.offsetHeight - (e.clientY - e.currentTarget.getBoundingClientRect().top);

            },
            handleScroll() {
                //通过计算出来的百分比，然后对滚动条执行translate移动
                this.moveY = (this.wrap.scrollTop * 100 / this.wrap.clientHeight);
            },

            handleClick(e) {
                //获得点击位置与滚动框顶部之间的距离
                const offset = Math.abs(e.target.getBoundingClientRect().top - e.clientY)
                //让点击位置处于滚动条的中间
                const thumbHalf = this.thumb.offsetHeight / 2;
                //计算出滚动条在滚动框的百分比位置
                const thumbPositionPercentage = (offset - thumbHalf) * 100 / this.wrap.offsetHeight;
                //通过改变scrollTop来操作。所有操作滚动条的最后一步都是通过handleScroll来实现
                this.wrap.scrollTop = (thumbPositionPercentage * this.wrap.scrollHeight / 100);
            },

            startDrag(e) {
                e.stopImmediatePropagation();
                e.stopPropagation();
                this.cursorDown = true;
                document.addEventListener("mousemove", this.mouseMoveDocumentHandler);
                document.addEventListener("mouseup", this.mouseUpDocumentHandler);

                document.onselectstart = null
            },

            mouseMoveDocumentHandler(e) {
                if (this.cursorDown === false) return;
                const prevPage = this.clickThumbAxis;

                if (!prevPage) return;

                //获得点击位置与 滚动框顶部 之间的距离
                const offset = (this.bar.getBoundingClientRect().top - e.clientY) * -1;


                //获得点击位置与 滚动条顶部 之间的距离
                const thumbClickPosition = this.thumb.offsetHeight - prevPage;
                //获得滚动条所处的百分比位置
                const thumbPositionPercentage = (offset - thumbClickPosition) * 100 / this.bar.offsetHeight;
                //计算出滚动条应该在滚动框中所处的位置，scrollTop

                this.wrap.scrollTop = thumbPositionPercentage * this.wrap.scrollHeight / 100;
                this.bar.style.opacity = 1
            },

            mouseUpDocumentHandler(e) {
                this.cursorDown = false;
                this.clickThumbAxis = 0;
                document.removeEventListener("mousemove", this.mouseMoveDocumentHandler);
                document.onselectstart = null;
                this.bar.style.opacity = 0
            }
        }
    }
</script>

<style scoped>
    * {
        margin: 0;
        padding: 0;
    }
    .scrollbar {
        position: relative;
        overflow: hidden;
        margin: 0;
        padding: 0;
    }
    .scrollbar-content {
        overflow: auto;
        height: 100%;
    }
    .scrollbar-bar {
        position: absolute;
        right: 0;
        bottom: 0;
        z-index: 1;
        opacity: 0;
        width: 6px;
        height: 100%;
        border-radius: 4px;
        background-color: rgba(0, 0, 0, .1);
        transition: opacity 120ms ease-out;
    }
    .scrollbar-thumb {
        position: relative;
        display: block;
        width: 100%;
        height: 0;
        border-radius: inherit;
        background-color: rgba(135, 141, 153, .3);
        transition: background-color .3s, transform .15s;
        cursor: pointer;
    }
    .scrollbar-thumb {
        background-color: rgba(54, 59, 70, 0.3);
    }

    .scrollbar:hover .scrollbar-bar {
        opacity: 1 !important;
        transition: opacity 340ms ease-out;
    }

</style>
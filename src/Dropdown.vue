<template>
    <div :class="[dropdownClass,{'v-dropdown-embed': embed,'animate-drop': !embed,'drop-up': dropUp}]"
         :style="[styleSheet]"
         v-show="show">
        <slot></slot>
    </div>
</template>

<script>
    export default {
        name: "v-dropdown",
        props:{
            position: {
                type: String,
                default: 'left'
            },
            embed: {
                type: Boolean,
                default: false
            },
            rightClick: {
                type: Boolean,
                default: false
            },
            x: Number,
            y: Number
        },
        data(){
            return {
                show: false,
                styleSheet: { top: '',left: '' },
                dropdownClass: 'v-dropdown-container',
                dropUp: false,
                lastCaller: null
            };
        },
        watch:{},
        methods: {
            visible(state, caller){
                if(typeof(state) === 'boolean' && this.show !== state){
                    this.show = state;
                    let that = this;
                    this.$nextTick(()=>{
                        if(that.show){
                            if(!that.embed){
                                that.adjust(caller);
                                that.lastCaller = caller;
                            }
                        }else this.$emit('show-change', false);
                    });
                }
            },
            getDir(caller){
                let pos = caller.getBoundingClientRect(),gap = 5, t = 0, u = false;
                let srcTop = this.rightClick ? this.y : pos.top;
                let menuPos = this.$el.getBoundingClientRect();
                let screenScrollTop = window.pageYOffset, viewHeight = document.documentElement.clientHeight;



                t = this.rightClick ? this.y : pos.top + pos.height + gap + screenScrollTop;
                let overDown = false, overUp = false;
                //list over screen
                if((t + menuPos.height) > (screenScrollTop + viewHeight)) overDown = true;
                if((srcTop - gap - menuPos.height) < 0) overUp = true;

                if(!overUp && overDown){
                    t = (srcTop - gap - menuPos.height) + screenScrollTop;
                    u = true;
                }

                return {top: t, up: u};
            },
            adjust(caller){
                let pos = caller.getBoundingClientRect(), gap = 5, t = 0, l = 0;
                let menuPos = this.$el.getBoundingClientRect();
                let info = this.getDir(caller);

                this.dropUp = info.up;
                t = info.top;

                if(this.rightClick){
                    l = this.x;
                }else{
                    switch (this.position){
                        case 'left':
                            l = pos.left;
                            break;
                        case 'center':
                            l = (pos.left + (pos.width / 2)) - (menuPos.width / 2);
                            break;
                        case 'right':
                            l = (pos.left + pos.width) - menuPos.width;
                            break;
                    }
                }

                this.styleSheet.top = t + 'px';
                this.styleSheet.left = l + 'px';
            },
            whole(e){
                let that = this;
                if(this.show){
                    // || val.outerHTML !== that.lastCaller.outerHTML)
                    let idx = e.path.findIndex(val=>val.className && val.className.includes(that.dropdownClass));
                    if(idx === -1) {
                        this.show = false;
                        this.$emit('show-change', false);
                    }
                }
            },
            MouseEventPolyfill(){
                if (!('path' in Event.prototype)) {
                    Object.defineProperty(Event.prototype, 'path', {
                        get: function () {
                            const path = [];
                            let currentElem = this.target;
                            while (currentElem) {
                                path.push(currentElem);
                                currentElem = currentElem.parentElement;
                            }
                            if (path.indexOf(window) === -1 && path.indexOf(document) === -1)
                                path.push(document);
                            if (path.indexOf(window) === -1) path.push(window);
                            return path;
                        }
                    });
                }
            }
        },
        mounted(){
            //console.log(this.$el.style.display)
            //console.log(this.caller)
            this.MouseEventPolyfill();

            if(this.embed) this.visible(true);
            else{
                document.body.appendChild(this.$el)

                this.$on('show', this.visible);
                this.$on('adjust', this.adjust);
                document.addEventListener('mousedown', this.whole);
            }
        },
        destroyed(){
            if(!this.embed) {
                this.$off('show', this.visible);
                this.$off('adjust', this.adjust);
                document.removeEventListener('mousedown', this.whole);
                this.$el.remove();
            }
        }
    }
</script>

<style lang="scss" scoped>
    div.v-dropdown-container{
        border: 1px solid #D6D7D7;
        margin: 0;
        padding: 0;
        display: inline-block;
        position: absolute;
        top:0;
        left:0;
        vertical-align: middle;
        box-sizing: border-box;

        /*background-color: #F5F5F5;*/
        background-color: white;
        border-radius: 2px;
        box-shadow: 0 3px 12px rgba(0,0,0,0.2);
        -moz-box-shadow: 0 3px 12px rgba(0,0,0,0.2);
        -webkit-box-shadow: 0 3px 12px rgba(0,0,0,0.2);
        z-index: 3000;
        &.sm_regular { width: auto;min-width: 150px; }
        &.sm_embed { position: relative; }
        /* Select only */
        & > .sm_select_ng { background: #fcc; }
        /*输入框设置了input-block-level样式时的特殊情况修复*/
        input.sm_input.input-block-level{
            box-sizing:border-box;
            height: 30px;
            line-height: 30px;
            min-height: 30px;
            width: 100%;
        }
        &.animate-drop {
            -webkit-animation: dropDownFadeInDown 300ms cubic-bezier(.23,1,.32,1);
            animation: dropDownFadeInDown 300ms cubic-bezier(.23,1,.32,1);
            &.drop-up {
                -webkit-animation: dropDownFadeInUp 300ms cubic-bezier(.23,1,.32,1);
                animation: dropDownFadeInUp 300ms cubic-bezier(.23,1,.32,1);
            }
        }

        &.v-dropdown-embed {
            position: relative;
            -webkit-box-shadow: 0 1px 8px rgba(0,0,0,.15);
            box-shadow: 0 1px 8px rgba(0,0,0,.15);
            z-index: 100;
        }
    }

    @keyframes dropDownFadeInDown {
        from{ opacity: 0;transform: translate3d(0, -20px, 0); }
        to{ opacity: 1;transform: translate3d(0, 0, 0); }
    }
    @keyframes dropDownFadeInUp {
        from{ opacity: 0;transform: translate3d(0, 20px, 0); }
        to{ opacity: 1;transform: translate3d(0, 0, 0); }
    }

</style>
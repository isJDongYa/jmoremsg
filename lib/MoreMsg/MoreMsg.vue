<!--
 * @Author: isJDongYa
 * @LastEditors: isJDongYa
 * @Date: 2020-12-07 16:32:14
 * @LastEditTime: 2020-12-08 15:59:36
 * @Description: 显示更多信息
-->
<template>
  <div 
    class="more-msg"
    @mouseenter="showMoreText" 
    @mouseleave="closeMoreText"
    @mousemove="moveMoreText"
  >
    <slot></slot>
    <div 
      class="more-msg-contain" 
      v-show="isShow"
      ref="moreMsg"
      :style="Object.assign(
        {width: computedSetup.width, height: computedSetup.height, background: computedSetup.background },
        position
      )"
    >
      <slot name="custom"></slot>
      {{ text }}
    </div>
  </div>
</template>
<script lang="ts">
import { VueConstructor } from 'vue';
import { Vue, Component, Prop } from 'vue-property-decorator'

import { Setup } from '@lib/types'

@Component
export default class MoreMsg extends Vue {

  private winWidth:number = document.body.clientWidth;
  private winHeigh:number = document.body.clientHeight;
  private isShow:boolean = false;
  private timer:number = 0;
  private position:{
    left:string;
    top:string;
  } = {
    left: '-40000px',
    top: '-20000px',
  };
  private moreTextWidth:number = 0;
  private moreTextHeight:number =  0;

  public install: any = undefined;

  private defaultSetup:Setup = {
        duration: 3000,
        width: 'auto',
        height: 'auto',
        background: '#fff'
  }

  @Prop({default: ''}) private text?:string;
  @Prop({default: false}) private show?:boolean;
  @Prop() 
  private setup?:Setup;
  @Prop({default: false}) private follow?:boolean;
    
    public get computedSetup():Setup {
      return Object.assign(this.defaultSetup, this.setup)
    }
    

    public showMoreText(e:MouseEvent):void {
      const duration = this.computedSetup?.duration;
      this.isShow = true;
      this.$nextTick(() => {
        this.moreTextWidth = (this.$refs.moreMsg as HTMLElement).offsetWidth
        this.moreTextHeight = (this.$refs.moreMsg as HTMLElement).offsetHeight
        this.isShow = false
      })
     
      this.timer = setTimeout(() => {
        this.isShow = true  
      }, duration);

    };

    public moveMoreText(e:MouseEvent):void {
      if(this.follow) {
        this.setPosition(e)
      } else {
        if (!this.isShow) {
          this.setPosition(e)
        }
      }
    };

    public setPosition(e:MouseEvent):void {
      let offset = 10
      let isRight = true
      let isBottom = true
      if(this.winWidth < e.clientX + offset + this.moreTextWidth) {
        isRight = false
      }
      if (this.winHeigh < e.clientY + offset + this.moreTextHeight) {
        isBottom = false
      }

      if (isRight) {
        this.position.left = e.clientX + offset + 'px'
      } else {
        this.position.left = e.clientX - this.moreTextWidth - offset + 'px'
      }

      if (isBottom) {
        this.position.top = e.clientY + offset + 'px'
      } else {
        this.position.top = e.clientY - this.moreTextHeight - offset + 'px'
      } 
    };
    
    public closeMoreText(e:MouseEvent):void {
      if(this.timer) {
        clearTimeout(this.timer)
      }
      this.isShow = false
    };

    public mounted():void {
      window.addEventListener('resize' , () => {
        this.winWidth = document.body.clientWidth
        this.winHeigh = document.body.clientHeight
      })
    };

}

</script>
<style lang="less" scoped>
.more-msg-contain {
  position: fixed;
  border-radius: 8px;
  font-size: 16px;
  background-color: #fff;
  padding: 10px;
  overflow: hidden;
}
</style>
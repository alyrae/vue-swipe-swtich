<template>
  <div class="container">
    <nav>
      <swiper ref="nav-swiper" :options="nav_options" class="nav-swiper" @click-slide="clickNavSlide">
        <swiper-slide v-for="item in nav" :key="item" class="nav-slide" ref="nav-slide">{{item}}</swiper-slide>
        <div class="bar" ref="bar" @transitionend="barTransitionEnd"></div>
      </swiper>
    </nav>
    <main>
      <swiper 
        ref="content-swiper" 
        :options="content_options" 
        class="content-swiper"
        @touchMove="touchMove" 
        @touchEnd="touchEnd"
        @transitionStart="contentTransitionStart"
      >
        <swiper-slide v-for="i in nav.length" :key="i" class="content-slide">
          <swiper 
            ref="scroll-swiper"
            class="scroll-swiper" 
            :options="scroll_options"
            @touchEnd="scrollTouchEnd(i - 1)"
            @touchMove="scrollTouchMove(i - 1)"
          >
            <swiper-slide class="scroll-slide" ref="scroll-slide">
              <div class="refresh" ref="refresh">下拉刷新</div>
              <div class="scroll-item" v-for="item in list" :key="item">slide{{i}}-{{item}}</div>
              <div class="loadmore" ref="loadmore">上拉加载</div>
            </swiper-slide>
          </swiper>
        </swiper-slide>
      </swiper>
    </main>
  </div>
</template>

<style>
* {
  margin: 0;
  padding: 0;
  user-select: none;
}
html, body {
  height: 100%;
}
.container {
  height: 100%;
  display: flex;
  flex-direction: column;
}
nav {
  flex-shrink: 0
}
.nav-swiper {  
  background: #fff;
  border-bottom: 1px solid #eee
}
.nav-slide {
  margin: 0 20px;
  line-height: 40px;
  width: fit-content;
}
.bar {
  height: 4px;
  background: #ff4891;
  position: absolute;
  bottom: 0;
  margin: 0 20px
}
main {
  flex: 1;
  overflow: hidden;
}
.content-swiper {
  height: 100%;
}
.content-slide {
  background: #eee
}
.scroll-swiper {
  height: 100%;
}
.scroll-slide {
  height: auto;
  background: #fff
}
.scroll-item {
  height: 99px;
  border-bottom: 1px solid #eee;
}
.refresh {
  width: 100%;
  position: absolute;
  bottom: 100%;
  line-height: 100px;
  color: #020202;
  text-align: center
}
.loadmore {
  width: 100%;
  position: absolute;
  top: 100%;
  line-height: 100px;
  color: #020202;
  text-align: center
}
</style>


<script>
/* eslint-disable */
import { Swiper, SwiperSlide, directive } from "vue-awesome-swiper";
import "swiper/css/swiper.css";
export default {
  data() {
    return {
      nav: Object.freeze([
        "分类1",
        "分类2",
        "分类3",
        "分类4",
        "分类5",
        "分类6",
        "分类7",
        "分类8",
        "分类9",
        "分类10",
        "分类11",
      ]),
      nav_options: Object.freeze({
        freeMode: true,
        slidesPerView: "auto"
      }),
      content_options: Object.freeze({
        watchSlidesProgress: true,
        resistanceRatio: 0
      }),
      scroll_options: Object.freeze({
        direction: 'vertical',
        freeMode: true,
        slidesPerView: "auto",
        mousewheel: true,
      }),
      list: Array.from(new Array(15).keys())
    };
  },
  components: {
    Swiper,
    SwiperSlide
  },
  directives: {
    swiper: directive
  },
  computed: {
    navSwiper() {
      return this.$refs["nav-swiper"].$swiper;
    },
    contentSwiper() {
      return this.$refs["content-swiper"].$swiper;
    },
    navsInfo() {
      if (!this.$refs["nav-slide"]) return [];
      let arr = this.$refs["nav-slide"].map(item => {
        let { clientWidth, offsetLeft } = item.$el;
        return { clientWidth, offsetLeft };
      });
      return arr;
    },
    bar() {
      return this.$refs.bar;
    },
    scrollSwipers() {
      return this.$refs['scroll-swiper'].map(item => item.$swiper)      
    },
    scrollSlide() {
      return this.$refs['scroll-slide'].map(item => item.$el)
    },
    refresh() {
      return this.$refs.refresh
    },
    loadmore() {
      return this.$refs.loadmore
    }
  },
  methods: {
    sleep(time) {
      return new Promise(resolve => {
        let id = setTimeout(() => {
          resolve()
          clearTimeout(id)
        }, time || 0)
      })
    },
    barTransitionEnd() {
      this.bar.style.transition = ''
    },
    barTransitionTo(i) {
      let { offsetLeft, clientWidth } = this.navsInfo[i];
      this.bar.style.transition = "300ms";
      this.bar.style.width = `${clientWidth}px`;
      this.bar.style.transform = `translate3d(${offsetLeft - 20}px, 0, 0)`;
    },
    clickNavSlide(i) {
      if (i != this.contentSwiper.activeIndex) {
        this.barTransitionTo(i);
        this.contentTransitionStart(i)
        this.contentSwiper.slideTo(i, 0)
      }
    },
    touchMove() {
      let {activeIndex, progress} = this.contentSwiper
      let a = Math.floor(progress * (this.nav.length - 1))
      let b = progress - a / (this.nav.length - 1)
      let x1 = this.navsInfo[a].offsetLeft - 20
      let x2 = b * (this.navsInfo[a].clientWidth + 40) * (this.nav.length - 1)
      if (a + 1 <= this.navsInfo.length - 1) {
        let w1 = this.navsInfo[a].clientWidth
        let diff = this.navsInfo[a+1].clientWidth - this.navsInfo[a].clientWidth
        let w = w1 + b * diff * (this.nav.length - 1)
        this.bar.style.width = `${w}px`
      }
      this.bar.style.transform = `translate3d(${x1 + x2}px,0,0)`
    },
    touchEnd() {
      let {progress} = this.contentSwiper
      if (progress == 0) {
        this.bar.style.transition = "300ms";
        this.bar.style.width = `${this.navsInfo[0].clientWidth}px`; 
      } else if (progress == 1) {
        this.bar.style.transition = "300ms";
        this.bar.style.width = `${this.navsInfo[this.navsInfo.length - 1].clientWidth}px`;
      }
    },
    contentTransitionStart(i) {
      i = i || this.contentSwiper.activeIndex
      this.barTransitionTo(i)
      let {offsetLeft: activeLeft,clientWidth:activeWidth} = this.navsInfo[i]
      let {clientWidth} = this.navSwiper.$el[0]
      let lastNav = this.navsInfo[this.navsInfo.length - 1]
      let scrollWidth = lastNav.offsetLeft + lastNav.clientWidth + 20
      let navWidth = this.bar.clientWidth
      let navLeft = this.bar.offsetLeft
      if (activeLeft < (clientWidth - activeWidth) / 2) {
        this.navSwiper.translateTo(0)
      } else if (activeLeft - (scrollWidth - clientWidth) > (clientWidth - activeWidth) / 2) {
        this.navSwiper.translateTo(clientWidth - scrollWidth)
      } else {
        this.navSwiper.translateTo((clientWidth - activeWidth) / 2 - activeLeft)
      }      
    },
    async scrollTouchEnd(i) {    
      let swiper = this.scrollSwipers[i]
      let {translate, height: swiperHeight} = swiper
      let {clientHeight: slideHeight} = this.scrollSlide[i]      
      if (translate > 100) { // 刷新
        swiper.touchEventsData.isTouched = false
        swiper.allowTouchMove = false
        this.refresh[i].innerHTML = '刷新中...'
        swiper.setTranslate(100)
        swiper.setTransition(300)
        await this.sleep(1000)
        let newData = ['new1', 'new2', 'new3','new4', 'new5', 'new6','new7', 'new8', 'new9','new10', 'new11', 'new12','new13', 'new14', 'new15']
        this.list = newData
        swiper.allowTouchMove = true
        this.refresh[i].innerHTML = '刷新完成'
      } else if (swiperHeight - translate > slideHeight + 100) { // 加载
        swiper.touchEventsData.isTouched = false
        swiper.allowTouchMove = false
        this.loadmore[i].innerHTML = '加载中...'
        swiper.setTranslate(swiperHeight - slideHeight - 100)
        swiper.setTransition(300)
        await this.sleep(1000)
        let newData = ['more1', 'more2', 'more3','more4', 'more5', 'more6','more7', 'more8', 'more9','more10', 'more11', 'more12','more13', 'more14', 'more15']
        this.list = [...this.list, ...newData]
        swiper.allowTouchMove = true
        this.refresh[i].innerHTML = '加载完成'
      }
    },
    scrollTouchMove(i) {
      let swiper = this.scrollSwipers[i]
      let {translate, height: swiperHeight} = swiper
      let {clientHeight: slideHeight} = this.scrollSlide[i]      
      if (translate > 100) {
        this.refresh[i].innerHTML = '释放刷新'
      } else if (swiperHeight - translate > slideHeight + 100) {
        this.loadmore[i].innerHTML = '释放加载'        
      } else {
        this.refresh[i].innerHTML = '下拉刷新'
        this.loadmore[i].innerHTML = '上拉加载'
      }
    }
  },
  mounted() {
    this.barTransitionTo(0);
  }
};
</script>


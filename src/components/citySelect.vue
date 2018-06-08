<template>
  <div class="nav-menu" >
    <div class="list-box" ref="list">
      <ul class="city-list">
        <li class="list-item" v-for="(item, index) in city" :key="index" ref="listGroup">
          <h2 class="item-title">
            {{item.title}}
          </h2>
          <div class="item-list" v-for="(items, indexs) in item.items" :key="indexs">
            {{items.city}}
          </div>
        </li>
      </ul>
    </div>
    <div class="shortcut"
    @touchmove.stop.prevent="onShortcutTouchMove"
    @touchstart="onShortcutTouchStart">
        <ul>
          <li class="item" v-for="(item, index) in shortcutList"
          :class="{'current': currentIndex == index}"
          :data-index="index"
          :key="index">
            {{item}}
          </li>
        </ul>
      </div>
      <div ref="fixed" v-show="fixedTitle" class="list-fixed">
        <h2>{{fixedTitle}}</h2>
      </div>
  </div>
</template>
<script>
import BScroll from 'better-scroll';
export default {
  data() {
    return {
      city: [],
      currentIndex: 0,
      scrollY: -1,
      diff: -1
    };
  },
  created() {
    this._getCity();
    this.listHeight = [];
    this.touch = {};
  },
  computed: {
    shortcutList() {
      return this.city.map((group) => {
        return group.title.substr(0, 1);
      });
    },
    fixedTitle() {
      if (this.scrollY > 0) {
        return '';
      }
      return this.city[this.currentIndex] ? this.city[this.currentIndex].title : '';
    }
  },
  methods: {
    _getCity() {
      this.$http.get('api/city').then(response => {
        response = response.body;
        this.city = this._sortCity(response.data);
        this.$nextTick(() => {
          this.scroll = new BScroll(this.$refs.list, {
            // 实时派发滑动事件
            probeType: 3,
            click: true
          });
          // 注册滑动事件监听
          this.scroll.on('scroll', (pos) => {
            this.scrollY = pos.y;
          });
        });
      });
    },
    _sortCity(list) {
      let map = {};
      list.forEach((item, index) => {
        const key = item.Cindex;
        if (!map[key]) {
          map[key] = {
            title: key,
            items: []
          };
        }
        map[key].items.push({
          city: item.Cname
        });
      });
      let ret = [];
      for (let key in map) {
        let val = map[key];
        if (val.title.match(/[a-zA-Z]/)) {
          ret.push(val);
        }
      }
      ret.sort((a, b) => {
        return a.title.charCodeAt(0) - b.title.charCodeAt(0);
      });
      return ret;
    },
    onShortcutTouchStart(e) {
      let anchorIndex = this.getData(e.target, 'index');
      let firstTouch = e.touches[0];
      this.touch.y1 = firstTouch.pageY;
      this.touch.anchorIndex = anchorIndex;
      this._scorllTo(anchorIndex);
    },
    onShortcutTouchMove(e) {
      let firstTouch = e.touches[0];
      this.touch.y2 = firstTouch.pageY;
      let delta = (this.touch.y2 - this.touch.y1) / 18 | 0;
      let anchorIndex = parseInt(this.touch.anchorIndex) + delta;
      this._scorllTo(anchorIndex);
    },
    _scorllTo(index) {
      if (!index && index !== 0) {
        return;
      }
      if (index < 0) {
        index = 0;
      } else if (index > this.listHeight.length - 2) {
        index = this.listHeight.length - 2;
      }
      this.scrollY = -this.listHeight[index];
      this.scroll.scrollToElement(this.$refs.listGroup[index], 400);
    },
    getData(el, name, val) {
      const prefix = 'data-';
      if (val) {
        return el.setAttribute(prefix + name, val);
      }
      return el.getAttribute(prefix + name);
    },
    _calculateHeight() {
      this.listHeight = [];
      const list = this.$refs.listGroup;
      let height = 0;
      this.listHeight.push(height);
      for (let i = 0; i < list.length; i++) {
        let item = list[i];
        height += item.clientHeight;
        this.listHeight.push(height);
      }
    }
  },
  watch: {
    city() {
      setTimeout(() => {
        this._calculateHeight();
      }, 20);
    },
    scrollY(newY) {
      const listHeight = this.listHeight;
      // 当滚动到顶部，newY > 0;
      if (newY > 0) {
        this.currentIndex = 0;
        return;
      }
      // 在中间部分滚动
      for (let i = 0; i < listHeight.length - 1; i++) {
        let height1 = listHeight[i];
        let height2 = listHeight[i + 1];
        if (-newY >= height1 && -newY < height2) {
          this.currentIndex = i;
          this.diff = height2 + newY;
          return;
        }
      }
      // 滚动到底部且-newY大于最后一个元素的上限
      this.currentIndex = listHeight.length - 2;
    },
    diff(newVal) {
      let fixedTop = (newVal > 0 && newVal < 43) ? newVal - 42 : 0;
      if (this.fixedTop === fixedTop) {
        return;
      }
      this.fixedTop = fixedTop;
      this.$refs.fixed.style.transform = `translate3d(0, ${fixedTop}px, 0)`;
    }
  }
};
</script>
<style lang="stylus">
.nav-menu
  position absolute
  width 100%
  top 0
  bottom 0
  .list-box
    position absolute
    top 0
    bottom 0
    width 100%
    .city-list
      .list-item
        .item-title
          background #c0c0c0
          color #fff
          padding 5px
          padding-left 10px
          margin 0
        .item-list
          padding 5px
          padding-left 15px
  .shortcut
    position absolute
    z-index 30
    right 0
    top 50%
    transform translateY(-50%)
    width 20px
    padding 20px 0
    border-radius 10px
    text-align center
    background rgba(0, 0, 0, .3)
    .item
      padding 3px
      line-height 1
      color rgba(255, 255, 255, .5)
      &.current
        color #ffcd32
  .list-fixed
    position absolute
    top -2px
    left 0
    width 100%
    z-index 20
    h2
      margin 0
      height 45px
      line-height 43px
      padding-left 10px
      background #c0c0c0
      color #fff
</style>

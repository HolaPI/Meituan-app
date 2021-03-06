<template>
  <div class="ratings" ref="ratingView">
    <div class="ratings-wrapper">
      <div class="overview">
        <div class="overview-left">
          <div class="comment-score">
            <p class="score">{{ratings.comment_score}}</p>
            <p class="text">商家评分</p>
          </div>
          <div class="other-score">
            <div class="quality-score item" v-show="ratings.food_score">
              <span class="text">口味</span>
              <app-star :score="ratings.food_score" class="star"></app-star>
              <span class="score">{{ratings.food_score}}</span>
            </div>
            <div class="pack-score" v-show="ratings.pack_score">
              <span class="text">包装</span>
              <app-star :score="ratings.pack_score" class="star"></app-star>
              <span class="score">{{ratings.pack_score}}</span>
            </div>
          </div>
        </div>
        <div class="overview-right">
          <div class="delivery-score">
            <p class="score">{{ratings.delivery_score}}</p>
            <p class="text">配送评分</p>
          </div>
        </div>
      </div>

      <!-- separator -->
      <app-split></app-split>
      <div class="content">
        <div class="rating-select" v-if="ratings.tab">
          <span
            class="item"
            :class="{'active': selectType == 2}"
            @touchstart="selectTypeFn(2)"
          >{{ratings.tab[0].comment_score_title}}</span>
          <span
            class="item"
            :class="{'active': selectType == 1}"
            @touchstart="selectTypeFn(1)"
          >{{ratings.tab[1].comment_score_title}}</span>
          <span class="item" :class="{'active': selectType == 0}" @touchstart="selectTypeFn(0)">
            <img v-if="selectType != 0" src="./img/icon_sub_tab_dp_normal@2x.png">
            <img v-if="selectType == 0" src="./img/icon_sub_tab_dp_highlighted@2x.png">
            {{ratings.tab[2].comment_score_title}}
          </span>
        </div>
        <div class="labels-view">
          <span
            class="item"
            v-for="(item, index) in ratings.labels"
            :key="index"
            :class="{'highlight': item.label_star > 0}"
          >{{item.content}}&nbsp;{{item.label_count}}</span>
        </div>
        <ul class="rating-list">
          <li class="comment-item" v-for="(comment, index) in selectComments" :key="index">
            <div class="comment-header">
              <img :src="comment.user_pic_url" v-if="comment.user_pic_url">
              <img src="./img/anonymity.png" v-if="!comment.user_pic_url">
            </div>
            <div class="comment-main">
              <div class="user">{{comment.user_name}}</div>
              <div class="time">{{formatDate(comment.comment_time)}}</div>
              <div class="star-wrapper">
                <app-star :score="comment.order_comment_score"></app-star>
              </div>
              <div class="c_content">{{comment.comment}}</div>
              <div class="img-wrapper" v-if="comment.comment_pics.length > 0">
                <div
                  :class="{'multiPics': comment.comment_pics.length > 1}"
                  v-for="(pic, index) in comment.comment_pics"
                  :key="index"
                >
                  <img :src="pic.url">
                </div>
              </div>
            </div>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
import Split from "../split/Split.vue";
import Star from "../star/Star.vue";
import BScroll from "better-scroll";

const ALL = 2;
const PICTURE = 1;
const COMMENT = 0;
export default {
  data() {
    return {
      ratings: {},
      //set type "ALL" initially, or errors occur
      selectType: ALL
    };
  },
  computed: {
    selectComments() {
      if (this.selectType == ALL) {
        return this.ratings.comments;
      } else if (this.selectType == PICTURE) {
        let pic_arr = [];
        this.ratings.comments.forEach(comment => {
          if (comment.comment_pics.length > 0) {
            pic_arr.push(comment);
          }
        });
        return pic_arr;
      } else {
        return this.ratings.comments_dp.comments;
      }
    }
  },
  components: {
    "app-split": Split,
    "app-star": Star
  },
  methods: {
    selectTypeFn(type) {
      this.selectType = type;
    },
    formatDate(time) {
      let date = new Date(time * 1000);
      let fmt = "yyyy.MM.dd";
      if (/(y+)/.test(fmt)) {
        // 年
        let year = date.getFullYear().toString();
        fmt = fmt.replace(RegExp.$1, year);
      }
      if (/(M+)/.test(fmt)) {
        // 月
        let mouth = date.getMonth() + 1;
        if (mouth < 10) {
          mouth = "0" + mouth;
        }
        fmt = fmt.replace(RegExp.$1, mouth);
      }
      if (/(d+)/.test(fmt)) {
        // 日
        let mydate = date.getDate();
        if (mydate < 10) {
          mydate = "0" + mydate;
        }
        fmt = fmt.replace(RegExp.$1, mydate);
      }
      return fmt;
    }
  },
  created() {
    fetch("/mock/ratings.json")
      .then(res => {
        return res.json();
      })
      .then(response => {
        if (response.code == 0) {
          this.ratings = response.data;
          this.$nextTick(() => {
            if (!this.scroll) {
              this.scroll = new BScroll(this.$refs.ratingView, {
                touchend: true
              });
            } else {
              this.scroll.refresh();
            }
          });
        }
      });
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.ratings {
  position: absolute;
  left: 0;
  top: 191px;
  bottom: 0;
  width: 100%;
  overflow: hidden;
}
.ratings .ratings-wrapper .overview {
  padding: 20px 0 18px 0;
  display: flex;
}
.ratings .ratings-wrapper .overview .overview-left {
  flex: 1;
  padding-left: 26px;
}
.ratings .ratings-wrapper .overview .overview-left .comment-score {
  float: left;
  width: 48px;
  text-align: center;
  margin-right: 26px;
}
.ratings .ratings-wrapper .overview .overview-left .comment-score .score {
  font-size: 23px;
  font-weight: bold;
  color: #ffb000;
  margin-bottom: 9px;
}
.ratings .ratings-wrapper .overview .overview-left .comment-score .text {
  font-size: 11px;
  color: #666;
}
.ratings .ratings-wrapper .overview .overview-left .other-score {
  float: left;
  margin-top: 3px;
}
.ratings .ratings-wrapper .overview .overview-left .other-score .item,
.ratings .ratings-wrapper .overview .overview-left .other-score .pack-score {
  height: 11px;
}
.ratings .ratings-wrapper .overview .overview-left .other-score .item {
  margin-bottom: 14px;
}
.ratings .ratings-wrapper .overview .overview-left .other-score .item .text,
.ratings
  .ratings-wrapper
  .overview
  .overview-left
  .other-score
  .pack-score
  .text {
  font-size: 11px;
  color: #666;
  margin-right: 11px;
  float: left;
}
.ratings .ratings-wrapper .overview .overview-left .other-score .item .star,
.ratings
  .ratings-wrapper
  .overview
  .overview-left
  .other-score
  .pack-score
  .star {
  float: left;
  margin-right: 11px;
}
.ratings .ratings-wrapper .overview .overview-left .other-score .item .score,
.ratings
  .ratings-wrapper
  .overview
  .overview-left
  .other-score
  .pack-score
  .score {
  font-size: 11px;
  color: #ffb000;
  float: left;
}
.ratings .ratings-wrapper .overview .overview-right {
  flex: 0 0 100px;
  text-align: center;
  border-left: 1px solid #9d9d9d;
}
.ratings .ratings-wrapper .overview .overview-right .delivery-score .score {
  font-size: 19px;
  font-weight: 500;
  color: #999;
  margin-bottom: 10px;
  margin-top: 3px;
}
.ratings .ratings-wrapper .overview .overview-right .delivery-score .text {
  font-size: 11px;
  color: #999;
}
.ratings .ratings-wrapper .content {
  padding: 16px;
}
.ratings .ratings-wrapper .content .rating-select {
  width: 100%;
  box-sizing: border-box;
  font-size: 0;
  border: 1px solid #ffb000;
  border-right: 0;
  margin-bottom: 11px;
  border-radius: 3px;
}
.ratings .ratings-wrapper .content .rating-select .item {
  width: 33.3%;
  display: inline-block;
  height: 33px;
  line-height: 33px;
  font-size: 14px;
  text-align: center;
  border-right: 1px solid #ffb000;
  box-sizing: border-box;
  color: #ffb000;
}
.ratings .ratings-wrapper .content .rating-select .item:last-child img {
  height: 14px;
  vertical-align: middle;
}
.ratings .ratings-wrapper .content .rating-select .item.active {
  background: #ffb000;
  color: #000;
}

.ratings .ratings-wrapper .content .labels-view .item {
  display: inline-block;
  height: 27px;
  line-height: 27px;
  padding: 0 10px;
  font-size: 12px;
  background: #f4f4f4;
  margin-right: 6px;
  margin-bottom: 6px;
  border-radius: 3px;
  color: #999999;
}

.ratings .ratings-wrapper .content .labels-view .item.highlight {
  color: #656565;
}

.ratings .ratings-wrapper .content .rating-list {
}

.ratings .ratings-wrapper .content .rating-list .comment-item {
  padding: 16px 16px 16px 0;
  border-bottom: 1px solid #f4f4f4;
  width: 100%;
  box-sizing: border-box;
  display: flex;
}

.ratings .ratings-wrapper .content .rating-list .comment-item .comment-header {
  flex: 0 0 35px;
  margin-right: 11px;
}

.ratings
  .ratings-wrapper
  .content
  .rating-list
  .comment-item
  .comment-header
  img {
  width: 35px;
  height: 35px;
  border-radius: 50%;
}

.ratings .ratings-wrapper .content .rating-list .comment-item .comment-main {
  flex: 1;
}

.ratings
  .ratings-wrapper
  .content
  .rating-list
  .comment-item
  .comment-main
  .user {
  width: 50%;
  float: left;
  font-size: 11px;
  color: #333333;
}

.ratings
  .ratings-wrapper
  .content
  .rating-list
  .comment-item
  .comment-main
  .time {
  width: 50%;
  float: right;
  text-align: right;
  font-size: 9px;
  color: #666666;
}

.ratings
  .ratings-wrapper
  .content
  .rating-list
  .comment-item
  .comment-main
  .star-wrapper {
  float: left;
  margin-top: 12px;
  margin-bottom: 15px;
  width: 100%;
}

.ratings
  .ratings-wrapper
  .content
  .rating-list
  .comment-item
  .comment-main
  .star-wrapper
  .text {
  color: #999999;
  font-size: 11px;
  float: left;
}

.ratings
  .ratings-wrapper
  .content
  .rating-list
  .comment-item
  .comment-main
  .star-wrapper
  .star {
  float: left;
  margin-left: 7px;
}

.ratings
  .ratings-wrapper
  .content
  .rating-list
  .comment-item
  .comment-main
  .c_content {
  font-size: 13px;
  line-height: 19px;
  float: left;
  width: 100%;
}

.ratings
  .ratings-wrapper
  .content
  .rating-list
  .comment-item
  .comment-main
  .c_content
  i {
  color: #576b95;
}

.ratings
  .ratings-wrapper
  .content
  .rating-list
  .comment-item
  .comment-main
  .img-wrapper {
  margin-top: 9px;
  float: left;
}
.ratings
  .ratings-wrapper
  .content
  .rating-list
  .comment-item
  .comment-main
  .img-wrapper
  div {
  display: inline-block;
  width: 160px;
  height: 160px;
  overflow: hidden;
}
.ratings
  .ratings-wrapper
  .content
  .rating-list
  .comment-item
  .comment-main
  .img-wrapper
  div.multiPics {
  display: inline-block;
  width: 80px;
  height: 80px;
  margin: 0 5px 5px 0;
  overflow: hidden;
}
.ratings
  .ratings-wrapper
  .content
  .rating-list
  .comment-item
  .comment-main
  .img-wrapper
  img {
  width: 100%;
}
</style>

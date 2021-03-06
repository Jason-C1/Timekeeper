<template>
  <div class="content">
    <transition
        enter-active-class="animated slideInUp"
        leave-active-class="animated slideOutDown"
    >
      <section class="todo" v-if="show">
        <div class="header">
          <div class="title">今日待办</div>
        </div>
        <div class="body">
          <swipeout>
            <div :key="index" v-for="(item, index) in todayTodoList">
              <swipeout-item>
                <div slot="right-menu">
                  <swipeout-button
                      @click.native="confirmDeleteItem(index)"
                      background-color="#f33"
                  >
                    删除
                  </swipeout-button>
                </div>
                <div class="item flex-box" slot="content">
                  <div
                      @click="completeItem(index)"
                      class="l"
                      v-if="item.item_state === 0"
                  >
                    <i aria-hidden="true" class="fa fa-square-o"> </i>
                  </div>
                  <div
                      @click="restartItem(index)"
                      class="l"
                      v-if="item.item_state !== 0"
                  >
                    <i aria-hidden="true" class="fa fa-check-square-o"> </i>
                  </div>
                  <div class="c">
                    <div class="name-line">
                      <div @click="goDetails(index)" class="name">
                        {{ item.item_name }}
                      </div>
                    </div>
                    <div class="level-line">
                      <div class="category-name">
                        #{{ item.category.category_name }}
                      </div>
                      <Label
                          backgroundActive="#ff3333"
                          size="sm"
                          text="不重要"
                          textActive="重要"
                          v-bind:active="item.item_importance_level >= 5"
                      />
                      <Label
                          size="sm"
                          textActive="紧急"
                          v-bind:active="item.item_emergency_level >= 5"
                      />
                    </div>
                  </div>
                  <div class="r">
                    <i aria-hidden="true" class="fa fa-angle-double-left"> </i>
                  </div>
                </div>
              </swipeout-item>
            </div>
          </swipeout>
          <div>
            <div class="nofound" v-if="todayTodoList.length === 0">
              您已完成所有待办「事项」<br/>
              可通过「上方」或「管理页」的「创建」按钮添加更多「事项」
            </div>
          </div>
        </div>
      </section>
    </transition>
    <section class="create-task">
      <router-link
          :to="{ name: 'createItem' }"
          class="start-btn flex-box"
          exact
          tag="div"
      >
        <div class="slogan">
          创建一个事项
        </div>
        <div class="r">
          <i aria-hidden="true" class="fa fa-paper-plane"> </i>
        </div>
      </router-link>
    </section>
  </div>
</template>
<script>
  import {mapGetters, mapMutations} from "vuex";
  import Label from "../../components/common/Label.vue";
  import {Swipeout, SwipeoutButton, SwipeoutItem} from "vux";

  export default {
    name: "home",
    components: {
      Label,
      Swipeout,
      SwipeoutItem,
      SwipeoutButton
    },
    data() {
      return {
        msg: "Welcome to Your Vue.js App",
        show: true
      };
    },
    computed: {
      ...mapGetters(["tokenInfo", "todayTodoList"])
    },
    mounted() {
      this.getItemList();
    },
    methods: {
      goDetails(index) {
        this.$router.push({
          name: "itemDetails",
          params: {
            syncKey: index
          }
        });
      },
      confirmDeleteItem(item_sync_key) {
        this.$vux.confirm.show({
          title: "确认删除",
          content: "确定要删除吗？删除就没啦~",
          onCancel: () => {
          },
          onConfirm: () => {
            this.deleteItem(item_sync_key);
          }
        });
      },
      deleteItem(item_sync_key) {
        let requestData = {item_sync_key, ...this.$store.getters.tokenInfo};
        this.$startRequest(
          "/item/deleteItem",
          requestData,
          res => {
            this.$vux.toast.text("删除成功", "top");
            this.getItemList();
          },
          error => {
            if (error.msg) {
              this.$vux.toast.text(error.msg, "top");
            } else {
              this.$vux.toast.text("网络错误", "top");
            }
          }
        );
      },
      completeItem(item_sync_key) {
        let item = this.todayTodoList[item_sync_key];
        let requestData = {item_sync_key, ...this.tokenInfo};
        this.$startRequest(
          "/item/completeItem",
          requestData,
          res => {
            let newItem = {};
            newItem[res.data.item_sync_key] = res.data;
            this.addItemList(newItem);
          },
          error => {
            if (error.msg) {
              this.$vux.toast.text(error.msg, "top");
            } else {
              this.$vux.toast.text("网络错误", "top");
            }
          }
        );
      },
      restartItem(item_sync_key) {
        let item = this.todayTodoList[item_sync_key];
        let requestData = {item_sync_key, ...this.tokenInfo};
        this.$startRequest(
          "/item/restartItem",
          requestData,
          res => {
            let newItem = {};
            newItem[res.data.item_sync_key] = res.data;
            this.addItemList(newItem);
          },
          error => {
            if (error.msg) {
              this.$vux.toast.text(error.msg, "top");
            } else {
              this.$vux.toast.text("网络错误", "top");
            }
          }
        );
      },
      getItemList() {
        let requestData = this.tokenInfo;
        this.$startRequest(
          "/item/getTodoList",
          requestData,
          res => {
            this.setItemList(res.data);
          },
          error => {
            if (error.msg) {
              this.$vux.toast.text(error.msg, "top");
            } else {
              this.$vux.toast.text("网络错误", "top");
            }
          }
        );
      },
      ...mapMutations(["addItemList", "setItemList"])
    },
  };
</script>

<style lang="less" scoped>
  .content {
    height: 100%;

    .create-task {
      left: 0;
      top: 0;
      width: 100%;
      position: absolute;

      .start-btn {
        margin: 30px 15px 15px;
        padding: 15px;
        border-radius: 50px;
        border: @home-start-input-border;
        color: @home-start-input-font-color;
        line-height: 1;
        font-size: 18px;
        vertical-align: middle;

        .slogan {
          text-align: left;
          flex: 1 0 auto;
          line-height: 1;
          font-size: 18px;
          vertical-align: middle;
        }

        i {
          flex: 0 0 auto;
        }
      }
    }

    .todo {
      padding-top: 120px;
      padding-bottom: 120px;
      min-height: 100%;
      flex-direction: column;
      display: flex;

      .header {
        // background: #f5f5f5;
        color: @home-todo-list-title-color;
        flex: 0 0 auto;
        width: 100%;
        border-radius: 10px 10px 0 0;
        margin-bottom: 20px;

        .title {
          display: inline-block;
          // line-height: 2;
          padding: 0 20px;
          font-size: 22px;
          text-indent: 20px;
          border-bottom: @input-font-color;
        }
      }

      .body {
        flex: 1 0 auto;

        .item {
          // margin-left: 20px;
          background: @dark-gray;
          text-align: center;
          padding: 6px 0;
          font-size: 25px;
          // border-radius: 40px 0 0 40px;
          // box-shadow: 0 0 5px 4px #f5f5f5;
          color: @home-todo-item-font-color;

          .l {
            padding: 10px 20px;
          }

          .c {
            text-align: left;

            .name-line {
              .name {
                display: inline-block;
              }

              font-size: 18px;
              line-height: 2;
            }

            .level-line {
              font-size: 16px;
              line-height: 1.2;
              color: @white;
              display: flex;

              .category-name {
                font-style: italic;
                opacity: 0.6;
              }

              & > div {
                margin-right: 15px;
              }
            }
          }

          .r {
            padding-right: 15px;
            opacity: 0.6;
          }
        }

        .nofound {
          padding: 20px 20px 6px 20px;
          font-style: italic;
          opacity: 0.8;
          font-size: 16px;
        }
      }
    }
  }
</style>

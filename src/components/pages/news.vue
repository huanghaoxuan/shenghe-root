<template>
  <div>
    <Button type="primary" style="margin-top: 25px" to="/newNewsInfo"
      >发布新公告</Button
    >
    <div style="width: 80%;margin-top: 25px;margin-left: 10%">
      <Table
        stripe
        :columns="columns"
        :data="newsInfo"
        @on-row-click="showContent"
      ></Table>
      <Modal
        title="请确认是否进行对该条公告进行编辑"
        v-model="updateModal"
        class-name="vertical-center-modal"
        @on-ok="updateNewsInfo"
      >
        <p>仅该公告创建者可对该条公告进行修改！</p>
      </Modal>
      <Modal
        title="请确认是否进行删除"
        v-model="removeModal"
        class-name="vertical-center-modal"
        @on-ok="remove"
      >
        <p>
          删除后将
          <font color="#ed4014">无法恢复</font>
          ！
        </p>
        <p>删除后将无法查看该条信息！</p>
      </Modal>
      <div style="margin: 10px;overflow: hidden">
        <div style="float: right;">
          <Page
            :total="this.total"
            :page-size="10"
            @on-change="onChange"
            show-elevator
          ></Page>
        </div>
      </div>
    </div>
    <div>
      <Drawer :closable="false" width="640" v-model="drawer.showDrawer">
        <div align="center">
          <h3>{{ drawer.title }}</h3>
          <Divider />
          <p>{{ drawer.content }}</p>
        </div>
      </Drawer>
    </div>
  </div>
</template>
<script>
export default {
  data() {
    return {
      drawer: { showDrawer: false, title: null, content: null },
      index: null,
      updateModal: false,
      removeModal: false,
      total: null,
      newsInfo: [],
      columns: [
        {
          title: "名字",
          key: "title"
        },
        {
          title: "公告时间",
          key: "time",
          width: 200
        },
        {
          title: "操作",
          key: "action",
          width: 150,
          align: "center",
          render: (h, params) => {
            return h("div", [
              h(
                "Button",
                {
                  props: {
                    type: "primary",
                    size: "small"
                  },
                  style: {
                    marginRight: "5px"
                  },
                  on: {
                    click: () => {
                      this.index = params.index;
                      this.updateModal = true;
                    }
                  }
                },
                "编辑"
              ),
              h(
                "Button",
                {
                  props: {
                    type: "error",
                    size: "small"
                  },
                  on: {
                    click: () => {
                      this.index = params.index;
                      this.removeModal = true;
                    }
                  }
                },
                "删除"
              )
            ]);
          }
        }
      ]
    };
  },
  mounted() {
    this.findNewsInfoListByPage(1);
    this.axios
      .get("/newsInfo/count", {
        params: {}
      })
      .then(
        function(res) {
          console.log(res.data);
          this.total = res.data;
          //控制台打印请求成功时返回的数据
          //bind(this)可以不用
        }.bind(this)
      )
      .catch(
        function(err) {
          if (err.response) {
            console.log(err.response);
            //控制台打印错误返回的内容
          }
          //bind(this)可以不用
        }.bind(this)
      );
  },
  methods: {
    onChange(event) {
      this.findNewsInfoListByPage(event);
    },
    findNewsInfoListByPage(page) {
      this.axios
        .get("/newsInfo/List", {
          params: { page: page }
        })
        .then(
          function(res) {
            console.log(res.data);
            this.newsInfo = res.data;
            //控制台打印请求成功时返回的数据
            //bind(this)可以不用
          }.bind(this)
        )
        .catch(
          function(err) {
            if (err.response) {
              console.log(err.response);
              //控制台打印错误返回的内容
            }
            //bind(this)可以不用
          }.bind(this)
        );
    },
    remove() {
      if (this.$store.state.gonghao != this.newsInfo[this.index].gonghao) {
        this.$Message.error("非当前当前用户发布的公告不允许删除");
      } else {
        this.axios
          .get("/newsInfo/remove", {
            params: {
              ID: this.newsInfo[this.index].id
            }
          })
          .then(
            function(res) {
              console.log(res.data);
              this.$router.go(0);
              //控制台打印请求成功时返回的数据
              //bind(this)可以不用
            }.bind(this)
          )
          .catch(
            function(err) {
              if (err.response) {
                console.log(err.response);
                //控制台打印错误返回的内容
              }
              //bind(this)可以不用
            }.bind(this)
          );
      }
    },
    showContent(data) {
      this.drawer.title = data.title;
      this.drawer.content = data.content;
      this.drawer.showDrawer = true;
    },
    updateNewsInfo() {
      if (this.newsInfo[this.index].gonghao != this.$store.state.gonghao) {
        this.$Message.error("非当前当前用户发布的公告不允许修改");
      } else {
        this.$store.state.newsInfo = this.newsInfo[this.index];
        this.$router.push("/updateNewsInfo");
      }
    }
  }
};
</script>

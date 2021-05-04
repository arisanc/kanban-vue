<template>
  <div id="app" :style="{ background: config.bg }">
    <section class="section">
      <h2>{{ datas.title }}</h2>
      <Progress
        :transitionDuration="5000"
        :radius="50"
        :strokeWidth="10"
        :value="datas.precent"
      >
        <template v-slot:footer>
          <b>任务进度</b>
        </template>
      </Progress>
    </section>
    <Kanban
      :stages="statuses"
      :blocks="blocks"
      @update-block="updateBlock"
      ref="kanban"
      @previous="previous"
      @next="next"
      :cardStyle.sync="cardStyle"
    >
      <div v-for="stage in statuses" :slot="stage.label" :key="stage.label">
        <h2>
          {{ stage.label }}
          <a>{{ getCount(stage.label) }}</a>
        </h2>
      </div>
      <div v-for="item in blocks" :slot="item.id" :key="item.id">
        <div></div>
        <div class="kb-card-title">
          {{ item.title }}
        </div>
        <div class="kb-card-tags"></div>
        <div
          class="kb-card-content"
          :style="{ '-webkit-line-clamp': config.contentLimit || 3 }"
          v-if="item.content"
        >
          {{ item.content }}
        </div>

        <div class="kb-card-progress">
          <progress-bar
            style="width: 30%"
            :bar-color="getColor(item)"
            :bg-color="item.bg || '#ccc'"
            :val="'' + item.progress"
            v-if="item.progress"
            :text="item.progress + '%'"
            :title="item.progress + '%'"
            text-position="bottom"
            bar-transition="all 2s ease"
            size="small"
            text-align="left"
          ></progress-bar>
          <div class="kb-card-dead-time" v-if="item.time">
            <img
              class="kb-card-dead-time-icon kb-icon-user"
              src="data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiA/PjwhRE9DVFlQRSBzdmcgIFBVQkxJQyAnLS8vVzNDLy9EVEQgU1ZHIDEuMS8vRU4nICAnaHR0cDovL3d3dy53My5vcmcvR3JhcGhpY3MvU1ZHLzEuMS9EVEQvc3ZnMTEuZHRkJz48c3ZnIGVuYWJsZS1iYWNrZ3JvdW5kPSJuZXcgMCAwIDQ4IDQ4IiBoZWlnaHQ9IjQ4cHgiIGlkPSJMYXllcl8xIiB2ZXJzaW9uPSIxLjEiIHZpZXdCb3g9IjAgMCA0OCA0OCIgd2lkdGg9IjQ4cHgiIHhtbDpzcGFjZT0icHJlc2VydmUiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyIgeG1sbnM6eGxpbms9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkveGxpbmsiPjxwYXRoIGNsaXAtcnVsZT0iZXZlbm9kZCIgZD0iTTQzLjEwMiwxNC4xMTNjLTAuMzMxLTAuNTc5LTAuNjg0LTEuMTQzLTEuMDY0LTEuNjg4ICBDNDMuMjA4LDExLjcyNyw0NCwxMC40NjIsNDQsOWMwLTIuMjA5LTEuNzkxLTQtNC00Yy0xLjQ2MiwwLTIuNzI3LDAuNzkyLTMuNDI1LDEuOTYzYy0wLjU0NS0wLjM4MS0xLjEwOC0wLjczMy0xLjY4OC0xLjA2NCAgQzM1LjkzOSw0LjE2NywzNy44MjYsMyw0MCwzYzMuMzEzLDAsNiwyLjY4Nyw2LDZDNDYsMTEuMTc0LDQ0LjgzMiwxMy4wNjEsNDMuMTAyLDE0LjExM3ogTTQ0LDI1YzAsNS4xNzEtMS45NzksOS44NjgtNS4yLDEzLjQxOSAgbDQuOTE1LDQuOTE1YzAuMzgxLDAuMzgxLDAuMzgxLDAuOTk5LDAsMS4zODFjLTAuMzgyLDAuMzgxLTEsMC4zODEtMS4zODEsMEwzNy40MTksMzkuOEMzMy44NjgsNDMuMDIxLDI5LjE3MSw0NSwyNCw0NSAgcy05Ljg2OC0xLjk3OS0xMy40MTgtNS4ybC00LjkxNiw0LjkxNWMtMC4zODEsMC4zODEtMSwwLjM4MS0xLjM4MSwwYy0wLjM4MS0wLjM4Mi0wLjM4MS0xLDAtMS4zODFMOS4yLDM4LjQxOSAgQzUuOTc5LDM0Ljg2OCw0LDMwLjE3MSw0LDI1QzQsMTMuOTU0LDEyLjk1NCw1LDI0LDVTNDQsMTMuOTU0LDQ0LDI1eiBNMjQsN0MxNC4wNTksNyw2LDE1LjA1OSw2LDI1YzAsOS45NDEsOC4wNTksMTgsMTgsMTggIHMxOC04LjA1OSwxOC0xOEM0MiwxNS4wNTksMzMuOTQxLDcsMjQsN3ogTTI1LDI4Ljg1OFYzOGMwLDAuNTUzLTAuNDQ3LDEtMSwxcy0xLTAuNDQ3LTEtMXYtOS4xNDJjLTEuNzIzLTAuNDQ2LTMtMS45OTctMy0zLjg1OCAgYzAtMC43NDksMC4yMTktMS40NDEsMC41NzgtMi4wNDFsLTQuMjkzLTQuMjkyYy0wLjM4MS0wLjM4MS0wLjM4MS0xLDAtMS4zODFjMC4zODItMC4zODEsMS0wLjM4MSwxLjM4MSwwbDQuMjkzLDQuMjkyICBDMjIuNTU5LDIxLjIxOSwyMy4yNTEsMjEsMjQsMjFjMi4yMDksMCw0LDEuNzkxLDQsNEMyOCwyNi44NjEsMjYuNzIzLDI4LjQxMiwyNSwyOC44NTh6IE0yNCwyM2MtMS4xMDQsMC0yLDAuODk2LTIsMnMwLjg5NiwyLDIsMiAgczItMC44OTYsMi0yUzI1LjEwNCwyMywyNCwyM3ogTTgsNUM1Ljc5MSw1LDQsNi43OTEsNCw5YzAsMS40NjIsMC43OTMsMi43MjcsMS45NjMsMy40MjVjLTAuMzgxLDAuNTQ1LTAuNzMzLDEuMTA5LTEuMDY0LDEuNjg4ICBDMy4xNjgsMTMuMDYxLDIsMTEuMTc0LDIsOWMwLTMuMzEzLDIuNjg3LTYsNi02YzIuMTc0LDAsNC4wNjEsMS4xNjcsNS4xMTMsMi44OTljLTAuNTgsMC4zMzEtMS4xNDQsMC42ODMtMS42ODgsMS4wNjQgIEMxMC43MjcsNS43OTIsOS40NjEsNSw4LDV6IiBmaWxsLXJ1bGU9ImV2ZW5vZGQiLz48L3N2Zz4="
            />
            {{ item.time }}
          </div>
        </div>
        <div class="kb-card-more">
          <div class="kb-card-worker">
            <img
              class="kb-icon-user"
              src="data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiA/Pjxzdmcgdmlld0JveD0iMCAwIDMyIDMyIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciPjx0aXRsZS8+PGcgZGF0YS1uYW1lPSJMYXllciAyIiBpZD0iTGF5ZXJfMiI+PHBhdGggZD0iTTE2LDE3LjVBNy41LDcuNSwwLDEsMSwyMy41LDEwLDcuNSw3LjUsMCwwLDEsMTYsMTcuNVptMC0xNEE2LjUsNi41LDAsMSwwLDIyLjUsMTAsNi41MSw2LjUxLDAsMCwwLDE2LDMuNVoiLz48cGF0aCBkPSJNMjYsMjguNUg2YS41LjUsMCwwLDEtLjUtLjVWMjJhLjY4LjY4LDAsMCwxLDAtLjE0Yy40OC0xLjY2LDItMi44MSwzLjQtMy44Mi4zMy0uMjUuNjQtLjQ4LjkzLS43MmEuNTEuNTEsMCwwLDEsLjY4LjA1QTcuMTcsNy4xNywwLDAsMCwxNiwxOS41YTcuMTEsNy4xMSwwLDAsMCw1LjQ1LTIuMTMuNTEuNTEsMCwwLDEsLjY4LS4wNWMuMjkuMjQuNi40Ny45My43MSwxLjM3LDEsMi45MiwyLjE3LDMuNCwzLjgzYS42OC42OCwwLDAsMSwwLC4xNHY2QS41LjUsMCwwLDEsMjYsMjguNVptLTE5LjUtMWgxOVYyMi4wN2MtLjQxLTEuMy0xLjc5LTIuMzMtMy0zLjIzbC0uNjEtLjQ2QTguMjQsOC4yNCwwLDAsMSwxNiwyMC41YTguMjksOC4yOSwwLDAsMS01Ljg5LTIuMTJsLS42Mi40NmMtMS4yMi45LTIuNiwxLjkzLTMsMy4yM1oiLz48L2c+PC9zdmc+"
            />
            {{ item.worker }}
          </div>
        </div>
      </div>
      <div
        v-for="stage in statuses"
        :key="stage.label"
        :slot="`footer-${stage.label}`"
        style="position: relative"
        class="kb-add-block-container"
        :data-status="stage.label"
        @click.prevent="() => addBlock(stage.label)"
      >
        <a class="kb-add-block">+ 添加</a>
      </div>
    </Kanban>
  </div>
</template>

<script>
import faker from "faker";
import { debounce } from "lodash";
import Kanban from "./components/Kanban";
import Progress from "easy-circular-progress";
import ProgressBar from "vue-simple-progress";

export default {
  name: "app",
  components: {
    Kanban,
    Progress,
    ProgressBar,
  },
  computed: {
    precent() {},
  },
  data() {
    return {
      datas: {
        title: "文档系统开发进度",
        //color:颜色
        //bg:背景颜色
        //name:名字
        //children:子模块
        tasks: [
          {
            label: "判断中",
            color: "white",
            background: "rgb(205, 155, 228)",
            tasks: [
              {
                title: "1",
                worker: "吕汉钊",
              },
              {
                title: "1.1",
                worker: "吕汉钊",
              },
              {
                title: "1.2",
                worker: "吕汉钊",
              },
              {
                title: "1.3",
                worker: "吕汉钊",
              },
            ],
          },
          { label: "计划中", color: "white", background: "#FB7D44" ,tasks: [
              {
                title: "2",
                worker: "吕汉钊",
              },
              {
                title: "2.1",
                worker: "吕汉钊",
              },
              {
                title: "2.2",
                worker: "吕汉钊",
              },
              {
                title: "2.3",
                worker: "吕汉钊",
              },
            ],},
          { label: "正在做", color: "white", background: "#2A92BF" ,tasks: [
              {
                title: "3",
                worker: "吕汉钊",
              },
              {
                title: "3.1",
                worker: "吕汉钊",
              },
              {
                title: "3.2",
                worker: "吕汉钊",
              },
              {
                title: "3.3",
                worker: "吕汉钊",
              },
            ],},
          {
            label: "即将完成",
            color: "rgb(252, 86, 50)",
            background: "#F4CE46",
          },
          { label: "已完成", color: "white", background: "#00B961" ,tasks: [
              {
                title: "4",
                worker: "吕汉钊",
              },
              {
                title: "4.1",
                worker: "吕汉钊",
              },
              {
                title: "4.2",
                worker: "吕汉钊",
              },
              {
                title: "4.3",
                worker: "吕汉钊",
              },
            ],},
          { label: "已废弃", color: "white", background: "rgb(60, 64, 67)" ,tasks: [
              {
                title: "5",
                worker: "吕汉钊",
              },
              {
                title: "5.1",
                worker: "吕汉钊",
              },
              {
                title: "5.2",
                worker: "吕汉钊",
              },
              {
                title: "5.3",
                worker: "吕汉钊",
              },
            ],},
          { label: "吕汉钊", color: "white", background: "rgb(205, 155, 228)" ,tasks: [
              {
                title: "6",
                worker: "吕汉钊",
              },
              {
                title: "6.1",
                worker: "吕汉钊",
              },
              {
                title: "6.2",
                worker: "吕汉钊",
              },
              {
                title: "6.3",
                worker: "吕汉钊",
              },
            ],},
        ],
        tags: [],
        okTags: ["光头"],
      },

      prograssColor: [
        { value: 25, r: 200, g: 0, b: 0, a: 0.2 }, //201, 0, 0
        { value: 50, r: 251, g: 125, b: 68, a: 0.5 }, //251, 125, 68
        { value: 75, r: 205, g: 155, b: 228, a: 0.5 }, //205, 155, 228
        { value: 95, r: 42, g: 146, b: 191, a: 0.5 }, //42, 146, 191
        { value: 100, r: 0, g: 185, b: 97, a: 0.5 }, //0, 185, 97
      ],
      cardStyle: {
        background: "rgba(255,255,255,.9)",
      },
      config: {
        bg: "#111",
        contentLimit: 2,
        opacity: ".4",
      },
      flag: 0,
      hrefs: [],
      tags: [
        {
          id: 9,
          times: 14,
          name: "紧急",
          type: 0,
          value: null,
          color: "",
          bg: "rgba(253, 226, 226, 1)",
        },
        {
          id: 10,
          times: 32,
          name: "计划中",
          type: 0,
          value: null,
          color: "",
          bg: "rgba(233, 233, 235, 1)",
        },
        {
          id: 11,
          times: 11,
          name: "正在做",
          type: 0,
          value: null,
          color: "rgba(255, 255, 255, 1)",
          bg: "rgba(30, 144, 255, 1)",
        },
        {
          id: 12,
          times: 5,
          name: "即将完工",
          type: 0,
          value: null,
          color: "rgba(21, 21, 21, 1)",
          bg: null,
        },
        {
          id: 13,
          times: 5,
          name: "已完成",
          type: 0,
          value: null,
          color: "rgba(210, 232, 239, 1)",
          bg: "rgba(77, 128, 47, 1)",
        },
        {
          id: 14,
          times: 16,
          name: "测试",
          type: 0,
          value: null,
          color: "rgba(255, 255, 255, 1)",
          bg: "rgba(235, 181, 99, 0.68)",
        },
        {
          id: 15,
          times: 5,
          name: "测试成功",
          type: 0,
          value: null,
          color: null,
          bg: "rgba(255, 215, 0, 0.24)",
        },
        {
          id: 16,
          times: 13,
          name: "完成",
          type: 0,
          value: null,
          color: null,
          bg: "rgba(95, 199, 73, 0.41)",
        },
        {
          id: 17,
          times: 4,
          name: "基本完成",
          type: 0,
          value: null,
          color: null,
          bg: "rgba(240, 249, 235, 1)",
        },
        {
          id: 18,
          times: 0,
          name: "警告",
          type: 0,
          value: null,
          color: "rgba(47, 46, 42, 1)",
          bg: "rgba(244, 213, 145, 1)",
        },
        {
          id: 19,
          times: 1,
          name: "测试失败",
          type: 0,
          value: null,
          color: "rgba(210, 181, 199, 1)",
          bg: "rgba(199, 21, 133, 1)",
        },
      ],
      statuses: [
        { label: "判断中", color: "white", background: "rgb(205, 155, 228)" },
        { label: "计划中", color: "white", background: "#FB7D44" },
        { label: "正在做", color: "white", background: "#2A92BF" },
        { label: "即将完成", color: "rgb(252, 86, 50)", background: "#F4CE46" },
        { label: "已完成", color: "white", background: "#00B961" },
        { label: "已废弃", color: "white", background: "rgb(60, 64, 67)" },
        { label: "吕汉钊", color: "white", background: "rgb(205, 155, 228)" },
        { label: "何成康", color: "white", background: "#FB7D44" },
        { label: "都烦", color: "white", background: "#2A92BF" },
        { label: "草资源", color: "rgb(252, 86, 50)", background: "#F4CE46" },
        { label: "光头", color: "white", background: "#00B961" },
        { label: "杨打破", color: "white", background: "rgb(60, 64, 67)" },
      ],
      blocks: [
        {
          id: 0,
          status: "计划中",
          title: "吔屎",
          worker: "吕汉钊",
          tags: [18, 19],
          finish: true,
          process: 0,
          progress: 60,
          time: "2021/5/4",
          content:
            "这是我写入的内容，如果超过了两行就会被自动加入胜率u饿啊上的阿松阿松阿松大阿萨十大阿松大阿斯顿打撒阿松大阿达阿松大啊",
        },
        {
          id: 1,
          status: "计划中",
          title: "123",
          worker: "吕汉钊",
        },
        {
          id: 2,
          status: "计划中",
          title: "123",
          worker: "吕汉钊",
        },
        {
          id: 3,
          status: "计划中",
          title: "123",
          worker: "吕汉钊",
        },
        {
          id: 4,
          status: "计划中",
          title: "123",
          worker: "吕汉钊",
        },
        {
          id: 5,
          status: "计划中",
          title: "123",
          worker: "吕汉钊",
        },
        {
          id: 6,
          status: "计划中",
          title: "123",
          worker: "吕汉钊",
        },
        {
          id: 7,
          status: "计划中",
          title: "123",
          worker: "吕汉钊",
        },
        {
          id: 10,
          status: "已废弃",
          title: "吔屎",
          worker: "吕汉钊",
          tags: [18, 19],
          finish: true,
          process: 0,
        },
        {
          id: 11,
          status: "已废弃",
          title: "123",
          worker: "吕汉钊",
        },
        {
          id: 12,
          status: "已废弃",
          title: "123",
          worker: "吕汉钊",
        },
        {
          id: 13,
          status: "已废弃",
          title: "123",
          worker: "吕汉钊",
        },
        {
          id: 14,
          status: "已废弃",
          title: "123",
          worker: "吕汉钊",
        },
        {
          id: 15,
          status: "已废弃",
          title: "123",
          worker: "吕汉钊",
        },
        {
          id: 16,
          status: "已废弃",
          title: "123",
          worker: "吕汉钊",
        },
        {
          id: 17,
          status: "已废弃",
          title: "123",
          worker: "吕汉钊",
        },
      ],
    };
  },
  created() {
    //创建css属性
    let css = ``;
    this.statuses.forEach((status) => {
      css += `ul[data-status=${status.label}]{color:${status.background};}.kb-add-block-container[data-status=${status.label}]:hover{background:${status.background};}.kb-add-block-container[data-status=${status.label}]:hover .kb-add-block{color:${status.color} !important;}`; //[data-status=${status.label}]
    });
    this.loadCssCode(css);
  },
  methods: {
    getColor(val) {
      for (let i = 0; i < this.prograssColor.length; i++) {
        if (val.progress <= this.prograssColor[i].value) {
          let obj = this.prograssColor[i];
          val.bg = `rgba(${obj.r},${obj.g},${obj.b},${
            obj.a || this.opacity || ".2"
          })`;
          val.color = `rgb(${obj.r},${obj.g},${obj.b})`;
          return val.color;
        }
      }
    },
    getHrefs() {
      this.hrefs = [];
      let hrefs = document.getElementsByClassName("kb-href");
      if (!hrefs[0]) {
        return;
      }
      let base = hrefs[0].getClientRects()[0].top;
      this.hrefs.push(0);
      for (let i = 1; i < hrefs.length; i++) {
        this.hrefs.push(hrefs[i].getClientRects()[0].top - base);
      }
    },
    next() {
      this.getHrefs();
      this.$refs.kanban.$el.style.overflowY = "scroll";
      this.$refs.kanban.$el.scrollTop = this.hrefs[this.flag + 1]
        ? this.hrefs[this.flag + 1]
        : this.hrefs[this.hrefs.length - 1];
      console.log("this.hrefs[this.flag+1]", this.hrefs[this.flag + 1]);
      if (this.hrefs[this.flag + 1]) {
        this.flag++;
      } else {
        this.flag = this.hrefs.length - 1;
      }
      //this.$refs.kanban.$el.scrollTop += innerHeight*0.83
      this.$refs.kanban.$el.style.overflowY = "hidden";
    },
    previous() {
      this.getHrefs();
      this.$refs.kanban.$el.style.overflowY = "scroll";
      this.$refs.kanban.$el.scrollTop = this.hrefs[this.flag - 1]
        ? this.hrefs[this.flag - 1]
        : this.hrefs[0];
      console.log("this.hrefs[this.flag-1]", this.hrefs[this.flag - 1]);
      if (this.hrefs[this.flag - 1]) {
        this.flag--;
      } else {
        this.flag = 0;
      }

      this.$refs.kanban.$el.style.overflowY = "hidden";
    },
    getCount(labelName) {
      return this.blocks.filter((block) => block.status === labelName).length;
    },
    updateBlock: debounce(function (id, status) {
      this.blocks.find((b) => b.id === Number(id)).status = status;
    }, 0),
    addBlock: debounce(function (stage) {
      this.blocks.push({
        id: this.blocks.length,
        status: stage,
        title: "新工作",
      });
    }, 0),
    loadCssCode(code) {
      var style = document.createElement("style");
      style.type = "text/css";
      style.rel = "stylesheet";
      //for Chrome Firefox Opera Safari
      style.appendChild(document.createTextNode(code));
      //for IE
      //style.styleSheet.cssText = code;
      var head = document.getElementsByTagName("head")[0];
      head.appendChild(style);
    },
  },
  mounted() {
    this.getHrefs();
  },
};
</script>

<style lang="scss">
@import "./assets/kanban.scss";

$on-hold: #fb7d44;
$in-progress: #2a92bf;
$needs-review: #f4ce46;
$approved: #00b961;

* {
  box-sizing: border-box;
}

body {
  background: "white";
  color: rgb(201, 86, 86);
  font-family: "Lato";
  font-weight: 300;
  line-height: 1.5;
  -webkit-font-smoothing: antialiased;
}

.drag-column {
  max-height: 85vh;
  margin-bottom: 5vh;
  border-radius: 10px;
  border: 1px solid #ccc;
  .drag-column-header > div {
    width: 100%;
    h2 > a {
      float: right;
    }
  }

  .drag-column-footer > div {
    margin-left: 10px;
    a {
      text-decoration: none;
      color: #555;
      &:hover {
        text-decoration: underline;
      }
    }
  }

  &-on-hold {
    .drag-column-header,
    .is-moved,
    .drag-options {
      background: $on-hold;
    }
  }

  &-in-progress {
    .drag-column-header,
    .is-moved,
    .drag-options {
      background: $in-progress;
    }
  }

  &-needs-review {
    .drag-column-header,
    .is-moved,
    .drag-options {
      background: $needs-review;
    }
  }

  &-approved {
    .drag-column-header,
    .is-moved,
    .drag-options {
      background: $approved;
    }
  }
}

.section {
  padding: 20px;
  text-align: center;

  a {
    color: "white";
    text-decoration: none;
    font-weight: 300;
  }

  h4 {
    font-weight: 400;
    a {
      font-weight: 600;
    }
  }
}

.drag-container {
  max-height: 85vh;
  overflow-y: hidden;
  scroll-behavior: smooth;
}

.is-moving {
  transform: rotate(10deg) scale(1.2) !important;
}

.drag-list {
  flex-wrap: wrap;
}

.drag-inner-list {
  overflow-y: overlay;
  overflow-y: scroll;
  max-height: 73vh;
}

::-webkit-scrollbar {
  /*滚动条整体样式*/
  width: 0;
  position: absolute;
  padding-right: 4px;
  background-color: white;
}

* {
  font-weight: 500;
}
::-webkit-scrollbar-thumb {
  /*滚动条里面小方块*/
  -webkit-border-radius: 4px;
  border-radius: 4px;
  background-color: rgba(221, 221, 221, 0.9);
}
::-webkit-scrollbar-thumb:window-inactive {
  /*滚动条里面轨道*/
  background-color: rgba(221, 221, 221, 0.9);
}

.kb-add-block-container {
  border-radius: 15px;
  margin: 10px 30% !important;
  position: relative;
  height: 40px;
  display: flex;
  justify-content: center;
  justify-items: center;
  border: 1px solid rgb(1, 153, 96);
  text-align: center;
  cursor: pointer;
}

.kb-add-block {
  color: rgb(1, 153, 96) !important;
  text-decoration: none !important;
  border-radius: 10px;
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  font-size: 1.2em;
}
</style>

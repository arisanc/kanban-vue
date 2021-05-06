<template>
  <div id="app" :style="{ background: config.bg }">
    <section class="section" :style="{color:config.progressColor}">
      <h2>{{ datas.title }}</h2>
      <Progress
        :transitionDuration="1000"
        :radius="50"
        :strokeWidth="10"
        :value="precent"
        :strokeColor="getBgByValue(precent==99.999?0:precent)"
      >
        <template v-slot:footer>
          <b>任务进度</b>
        </template>
      </Progress>
      <div style="position:absolute;bottom:5%;left:2%;">Power by arisanc</div>
    </section>
    <Kanban
      :kanbanList.sync="datas.kanbanList"
      :tagConfigs.sync="datas.tags"
      :blocks="blocks"
      @update-block="updateBlock"
      ref="kanban"
      @previous="previous"
      @next="next"
      @addBlock="addBlock"
      :cardStyle.sync="cardStyle"
      :config.sync="config"
    >  
    </Kanban>

  </div>
         
</template>

<script>
import faker from "faker";
import { debounce } from "lodash";
import Kanban from "./components/Kanban";
import Progress from "easy-circular-progress";

import index from '../example/index.json'

export default {
  name: "app",
  components: {
    Kanban,
    Progress,
  },
  computed: {
    precent(){
      let ok = this.blocks.filter(block=>{
        return (this.datas.okTags.indexOf(block.status)!=-1)
      }).length
      if(Math.round(ok / this.blocks.length *10000) /100 ===0){
        return 99.999 //因为插件存在bug，使用99.999作为标记（其实指向0）
      }

      return  Math.round(ok / this.blocks.length *10000) /100 || 0
    },
    blocks(){
      let result = []
      this.datas.kanbanList.forEach(kanban=>{
        kanban.tasks.forEach(task=>{task.status=kanban.label})
        result.push(...kanban.tasks)
      })
      return result
    }
  },
  data() {
    return {
      newId: 1,
      datas: {
        title: "",
        kanbanList: [],
        tags: [],
        okTags: [],
      },

      progressColor: [
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
        bg: "rgb(242, 234, 218)",
        contentLimit: 5,
        opacity: ".4",
        kanbanBg:"rgb(245, 245, 245)",
        taskBg:"#fff",
        progressColor:'rgb(95, 60, 35)',
        progressBg:"rgb(138, 46, 59)"
      },
      flag: 0,
      hrefs: [],
      statuses: [
      
      ],
    };
  },
  created() {
    this.datas = index
    this.addId()
  },
  methods: {
    getBgByValue(val) {
      for (let i = 0; i < this.progressColor.length; i++) {
        if (val <= this.progressColor[i].value) {
          let obj = this.progressColor[i];
         return `rgba(${obj.r},${obj.g},${obj.b},${
            obj.a || this.opacity || ".2"
          })`;
        }
      }
      return ''
    },
    getColor(val) {
      for (let i = 0; i < this.progressColor.length; i++) {
        if (val.progress <= this.progressColor[i].value) {
          let obj = this.progressColor[i];
          val.bg = `rgba(${obj.r},${obj.g},${obj.b},${
            obj.a || this.opacity || ".2"
          })`;
          val.color = `rgb(${obj.r},${obj.g},${obj.b})`;
          return val.color;
        }
      }
    },
    addId(){
      for (let kanban of this.datas.kanbanList) {
      for (let task of kanban.tasks) {
        task.id = this.newId;
        this.newId++;
      }
    }
    },
    addBlock(kanban){
      kanban.tasks.push({id:this.newId,status:kanban.label,title:'新任务',content:'请输入任务内容',worker:this.user})
      this.newId++
    },
    addKanban(name){

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
      if (this.hrefs[this.flag - 1]) {
        this.flag--;
      } else {
        this.flag = 0;
      }

      this.$refs.kanban.$el.style.overflowY = "hidden";
    },
    updateBlock: debounce(function (id,oldKanbanId, newKanbanId,index) {
      //去除旧的内容
      let oldTask= this.datas.kanbanList[+oldKanbanId].tasks
      //增加新的内容
      let newTask= this.datas.kanbanList[+newKanbanId].tasks
      for(let i=0;i<oldTask.length;i++){
        if(oldTask[i].id==id){
          newTask.splice(index,0,oldTask[i])
          oldTask.splice(i,1)
          break
        }
      }
    }, 0),
  },
  props:{
    user:{
      type:String,
      default:'admin'
    }
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
  max-height: 95vh;
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
  max-height: 95vh;
  overflow-y: hidden;
  scroll-behavior: smooth;
}

.is-moving {
  transform: rotate(10deg) scale(1.2) !important;
}

.drag-list {
  flex-wrap: wrap;
  min-height: 95vh;
}

.drag-inner-list {
  overflow-y: overlay;
  overflow-y: scroll;
  max-height: 88vh;
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
  margin: 0 30% 10px 30% !important;
  position: relative;
  height: 40px;
  display: flex;
  justify-content: center;
  justify-items: center;
  text-align: center;
  cursor: pointer;
}

.kb-add-block {
  text-decoration: none !important;
  border-radius: 10px;
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  font-size: 1.2em;
}
</style>

<template>
  <div id="todolist">
    <div class="header">
      <div
        class="header-checkbox"
        @click="changeAllStatus(allChecked)"
        :class="{ allChecked: allChecked }"
      ></div>
      <input
        type="text"
        placeholder="请输入内容"
        v-model="iptValue"
        ref="ipt"
      />
    </div>
    <div class="container">
      <div class="container-row" v-for="(item, index) of Lists" :key="item.id">
        <div
          class="container-checkbox"
          @click="changeStatus(index, item.done)"
          :class="{ checked: item.done }"
        />
        <div
          class="container-content"
          @click="modifyToDo($event)"
          :class="{ current: item.done }"
        >
          {{ item.todo }}
        </div>
        <input
          class="container-content-ipt"
          type="text"
          v-model="item.todo"
          @keyup="enterChange($event, item.todo, item.id)"
        />
        <div class="container-close" @click="removeToDo(item.id)">X</div>
      </div>
    </div>
    <div class="footer">
      <div class="footer-item">待完成 {{ $store.state.toDoNum }}</div>
      <div class="footer-item" @click="all()">全部</div>
      <div class="footer-item" @click="todo()">待办</div>
      <div class="footer-item" @click="done()">已完成</div>
      <div class="footer-clear" @click="clearAll()">清除选中内容</div>
    </div>
  </div>
</template>

<script>
export default {
  name: "ToDoList",

  data() {
    return {
      iptValue: "",
      lists: [],
      Lists: [],
      // $store.state.toDoNum: 0,
      allChecked: false,
      id: 0,
    };
  },

  methods: {
    /**
     * 删除当前待做事件
     */
    removeToDo(id) {
      const index = this.getIndex(id);
      if (!this.lists[index].done) {
        this.$store.state.toDoNum--;
      }
      this.lists.splice(index, 1);
      this.ifShow();
      this.all();
    },

    /**
     * 获取index
     */
    getIndex(id) {
      const vals = this.lists.filter((e) => {
        return e.id === id;
      });
      // 数组解构
      const [val] = vals;
      return this.lists.indexOf(val);
    },

    /**
     * 修改事件状态
     */
    changeStatus(index, done) {
      this.Lists[index].done = !done;
      this.ifShow();
      this.Lists[index].done ? this.$store.state.toDoNum-- : this.$store.state.toDoNum++;
      // 判断全选
      const unchecked = this.lists.filter((e) => {
        return e.done === false;
      });
      this.allChecked = !unchecked.length ? true : false;
    },

    /**
     * 判断是否显示“清除选中内容”
     */
    ifShow() {
      const doneNum = this.lists.filter((e) => {
        return e.done === true;
      });
      document.querySelector(".footer-clear").style.display = doneNum.length
        ? "block"
        : "none";
    },

    /**
     * 修改全部事件状态
     */
    changeAllStatus(allChecked) {
      this.allChecked = !allChecked;
      if (this.allChecked) {
        for (var j = 0; j < this.Lists.length; j++) {
          this.Lists[j].done = true;
        }
        this.$store.state.toDoNum = 0;
        document.querySelector(".footer-clear").style.display = "block";
      } else {
        for (var k = 0; k < this.Lists.length; k++) {
          this.Lists[k].done = false;
        }
        // 修改待完成数量
        let eNum = 0;
        this.Lists.forEach((e) => {
          if (!e.done) {
            eNum++;
          }
        });
        this.$store.state.toDoNum = eNum;
        document.querySelector(".footer-clear").style.display = "none";
      }
    },

    /**
     * 修改事件内容
     */
    modifyToDo(e) {
      e.target.nextSibling.style.display = "block";
      e.target.style.display = "none";
    },

    /**
     * 完成事件修改
     */
    enterChange(e, todo, id) {
      const index = this.getIndex(id);
      if (e.keyCode === 13) {
        if (todo === " " || todo === "") {
          this.lists.splice(index, 1);
          this.$store.state.toDoNum--;
        }
        e.target.previousSibling.style.display = "block";
        e.target.style.display = "none";
      }
    },

    /**
     * 待办事件
     */
    todo() {
      this.Lists = this.lists.filter(e => {
        return !e.done;
      });
    },

    /**
     * 已完成事件
     */
    done() {
      this.Lists = this.lists.filter(e => {
        return e.done;
      });
    },

    /**
     * 全部事件
     */
    all() {
      this.Lists = this.lists;
    },

    /**
     * 全部清除
     */
    clearAll() {
      const clearArr = this.lists.filter((e) => {
        return e.done === true;
      });
      for (var i = 0; i < clearArr.length; i++) {
        const index = this.getIndex(clearArr[i].id);
        this.lists.splice(index, 1);
      }
      this.all();
      this.allChecked = false;
      document.querySelector(".footer-clear").style.display = "none";
    },
  },

  mounted() {
    const that = this;
    this.all();

    /**
     * 新增待做事件
     */
    this.$refs.ipt.addEventListener("keyup", function (e) {
      if (e.keyCode === 13) {
        if (that.iptValue !== "" && that.iptValue !== " ") {
          that.lists.push({
            id: that.id++,
            todo: that.iptValue,
            done: false,
          });
          that.iptValue = "";
          that.$store.state.toDoNum++;
          that.allChecked = false;
          that.all();
          document.querySelector(".footer-clear").style.display = "none";
        }
      }
    });
  },
};
</script>


<style lang="scss">
@mixin alignItems {
  align-items: center;
}

$checked: #ccc;

.container-row {
  display: flex;
  @include alignItems;
  margin: 5px auto;
  width: 300px;
  height: 45px;
  border: 1px solid $checked;
}

.container-content {
  margin: 0 15px;
}

.container-close {
  cursor: pointer;
  width: 25px;
  height: 25px;
  line-height: 25px;
}

.container-content-ipt {
  display: none;
}

.footer {
  display: inline-flex;
  @include alignItems;
  margin: 15px auto;
}

.footer-item,
.footer-clear {
  margin: 0 15px;
  cursor: pointer;
}

.footer-clear {
  display: none;
}

.current {
  color: red;
}

.container-checkbox,
.header-checkbox {
  width: 20px;
  height: 20px;
  border: 1px solid $checked;
  cursor: pointer;
}

.checked {
  border: none;
  background-color: $checked;
}

.header {
  display: inline-flex;
  @include alignItems;
  margin: 0 auto;
}

.allChecked {
  background-color: $checked;
}
</style>
<template>
  <!-- 页面布局 -->
  <div class="layout">
    <!-- 取消按钮，点击时返回上一页 -->
    <div class="cancel" @click="$router.back()">取消</div>
     <!-- 标签页组件，用于切换记录类型 -->
    <Tabs
      class-prefix="type"
      :data-source="recordTypeList"
      :value.sync="record.type"
    />
    <!-- 标签组件，用于选择记录标签 -->
    <Tags
      :type.sync="record.type"
      class="tagArea"
      @update:value="record.tags = $event"
    />
    // 数字键盘和记录输出区域
    <div class="numberPad">
      <div class="notes-output">
        <!-- 选择图标 -->
        <div class="noteIcon"><Icon name="remark" /></div>
        <!-- 备注文字 -->
        <div class="notes">备注:</div>
        <!-- 备注输入区域 -->
        <div class="notesInput">
          <Notes
            class-prefix="notes"
            :value.sync="record.notes"
            placeholder="在这里输入备注..."
          />
        </div>
        <!-- 记录输出 -->
        <div class="output">{{ output }}</div>
      </div>
      <!-- 数字键盘按钮区域 -->
      <div class="buttons">
        <!-- 数字按钮 -->
        <div class="button" @click="inputContent">7</div>
        <div class="button" @click="inputContent">8</div>
        <div class="button" @click="inputContent">9</div>
        <!-- 日期选择按钮 -->
        <div class="button date">
           <!-- 日期选择器组件 -->
          <DatePicker
            placement="left-end"
            :open="open"
            :value.sync="record.createdAt"
            type="date"
            @on-change="handleChange"
            @on-clickoutside="handleAuthors"
          >
           <!-- 日期显示区域 -->
            <a class="abc" href="javascript:void(0)" @click="handleClick">
               <!-- 今天图标 -->
              <Icon
                v-if="day(record.createdAt).isSame(day(new Date()), 'day')"
                name="day"
              ></Icon>
              <template
                v-if="day(record.createdAt).isSame(day(new Date()), 'day')"
              >
                今天
              </template>
               <!-- 显示日期 -->
              <template v-else>{{ record.createdAt.split("T")[0] }}</template>
            </a>
          </DatePicker>
        </div>
        <!-- 其他数字按钮 -->
        <div class="button" @click="inputContent">4</div>
        <div class="button" @click="inputContent">5</div>
        <div class="button" @click="inputContent">6</div>
        <div class="button" @click="increase">
           <!-- 加号图标 -->
          <Icon name="plus" />
        </div>
        <div class="button" @click="inputContent">1</div>
        <div class="button" @click="inputContent">2</div>
        <div class="button" @click="inputContent">3</div>
        <div class="button" @click="decrease">
          <!-- 减号图标 -->
          <Icon name="minus" />
        </div>
        <div class="button" @click="inputContent">.</div>
        <div class="button" @click="inputContent">0</div>
        <!-- 删除按钮 -->
        <div class="button delete" @click="remove">
          <!-- 删除图标 -->
          <Icon name="delete" />
        </div>
         <!-- 完成按钮或等号按钮 -->
        <div v-if="equalSymbol" class="button ok" @click="saveRecord">完成</div>
        <div v-else class="button" @click="equalOutput">
          <!-- 等号图标 -->
          <Icon name="equal" />
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
// 引入依赖
import Vue from "vue";
import { Component } from "vue-property-decorator";
import { DatePicker } from "view-design";
import day from "dayjs";
import recordTypeList from "@/constants/recordTypeList";

@Component({
   // 注册组件
  components: {
    DatePicker,
    Notes: () => import("../components/Money/Notes.vue"),
    Tabs: () => import("../components/Money/Tabs.vue"),
    Tags: () => import("../components/Money/Tags.vue"),
  },
})
export default class KeepAccounts extends Vue {
  // 记录对象初始化
  record: RecordItem = {
    tags: {} as Tag,
    notes: "",
    type: "-",
    amount: 0,
    createdAt: day(new Date()).format("YYYY-MM-DDTHH:mm:ss"),
  };
  // 记录类型列表
  recordTypeList = recordTypeList;
  // dayjs 函数
  day = day;
  // 组件创建时的钩子函数
  created() {
    this.$store.commit("fetchRecords");
  }
  // 记录输出
  output: string = "0";
  // 处理数字按钮点击事件
  inputContent(event: MouseEvent) {
    const div = event.target as HTMLDivElement;
    const input = div.textContent as string;

    if (this.output.length === 10) {
      return;
    }
    if (this.output === "0") {
      if ("0123456789".indexOf(input) >= 0) {
        this.output = input;
      } else {
        this.output += input;
      }
      return;
    }
    if (this.output.indexOf(".") >= 0 && input === ".") {
      return;
    }
    if (this.output.indexOf(".") >= 0) {
      let arr = this.output.split(".");
      if (arr[1].length >= 2) {
        return;
      }
    }
    this.output += input;
  }
  // 删除按钮点击事件
  remove() {
    if (this.output.length <= 1) {
      this.output = "0";
      return;
    }
    this.output = this.output.slice(0, -1);
  }
  equalNumber = 0;
  symbol = true;
  equalSymbol = true; 
  // 加号按钮点击事件
  increase() {
    this.equalNumber = parseFloat(this.output);
    this.output = "0";
    this.symbol = true;
    this.equalSymbol = false;
  }
  // 减号按钮点击事件
  decrease() {
    this.equalNumber = parseFloat(this.output);
    this.output = "0";
    this.symbol = false;
    this.equalSymbol = false;
  }
  // 等号按钮点击事件
  equalOutput() {
    if (this.symbol) {
      this.output = (this.equalNumber + parseFloat(this.output)).toString();

      let arr = this.output.split(".");
      if (arr[1] && arr[1].length > 2) {
        let array2 = arr[1].split("");
        this.output = arr[0] + "." + array2[0] + array2[1];
      }

      this.equalSymbol = true;

      return;
    }
    const a = this.equalNumber - parseFloat(this.output);

    if (a < 0) {
      alert("不支持负数哦");
      this.output = "0";
    } else {
      this.output = a.toString();
      let arr = this.output.split(".");
      if (arr[1] && arr[1].length > 2) {
        let array2 = arr[1].split("");
        this.output = arr[0] + "." + array2[0] + array2[1];
      }
    }
    this.equalSymbol = true;
  }
   // 保存记录按钮点击事件
  saveRecord() {
    this.record.amount = parseFloat(this.output);
    if (!this.record.tags.name) {
      return window.alert("请选择一个标签");
    }
    if (this.record.amount === 0) {
      return window.alert("请输入金额");
    }
    this.$store.commit("createRecord", this.record);
    if (this.$store.state.createRecordError === null) {
      this.$router.replace("/");
      this.record.notes = "";
    }
    this.output = "0";
  }
  // 日期选择相关
  open = false;
  handleClick() {
    this.open = !this.open;
  }
  handleChange(date: string) {
    this.record.createdAt = date + day(new Date()).format("THH:mm:ss");
    this.open = false;
  }
  handleAuthors() {
    this.open = false;
  }
}
</script>

<style lang="scss" scoped>
@import "~@/assets/style/helper.scss";
.layout {
  position: relative;
  > .cancel {
    position: absolute;
    top: 4vw;
    right: 4vw;

    height: 24px;
    width: 48px;
    line-height: 1;
    font-size: 14px;
    text-align: right;
  }
  ::v-deep .type-tabs {
    position: relative;
  }
  ::v-deep .type-tabs-item {
    height: 6vh;
    font-size: 18px;
    background: #fdd844;
    border-bottom: 1px solid transparent;
    &.selected {
      &::after {
        content: "";
        position: absolute;
        width: 50%;
        height: 2px;
        bottom: 0;
        background: black;
      }
    }
  }
  .tagArea {
    height: 61vh;
  }
  .numberPad {
    height: 33vh;
    position: fixed;
    bottom: 0;
    .notes-output {
      line-height: 1;
      font-family: Consolas, monospace;
      padding: 0 1vh;
      height: 5vh;
      display: flex;
      flex-direction: row;
      background: #f7f5f6;
      > .noteIcon {
        display: flex;
        align-items: center;
        justify-content: center;
        height: 5vh;
        width: 8vw;
        > .icon {
          width: 18px;
          height: 18px;
        }
      }
      > .notes {
        display: flex;
        align-items: center;
        justify-content: center;
        height: 5vh;
        width: 12vw;
        line-height: 1;
        font-size: 12px;
        padding: 0 0.5vw;
      }
      > .notesInput {
        display: flex;
        ::v-deep .notes-tabs {
          height: 5vh;
          width: 40vw;
          font-size: 12px;
        }
      }
      > .output {
        display: flex;
        align-items: center;
        flex-direction: row-reverse;
        height: 5vh;
        width: 38vw;
        line-height: 1;
        font-size: 24px;
        padding-right: 2vw;
        margin: auto;
      }
    }
    .buttons {
      display: flex;
      flex-wrap: wrap;
      flex-direction: row;
      > .button {
        display: flex;
        align-items: center;
        justify-content: center;
        width: 25%;
        height: 7vh;
        &.ok {
          background: #fdd844;
        }
        &.delete {
          height: 7vh;

          > .icon {
            width: 24px;
            height: 24px;
          }
        }
        &.date {
          height: 7vh;
        }
        background: #f7f5f6;
        border: 0.5px solid rgb(231, 230, 230);
      }
    }
  }
  .abc {
    color: black;
  }
}
</style>

<template>
  <div class="tags">
    <ul class="current">
      <li
        class="labels"
        v-for="tag in taglist"
        :key="tag.id"
        :class="{ selected: previousTag === tag }"
        @click="toggleTag(tag)"
      >
        <div class="icon-radius">
          <Icon :name="tag.iconName" />
        </div>
        {{ tag.name }}
      </li>
      <router-link to="/labels" class="labels link">
        <div class="icon-radius link-radius">
          <Icon name="settings" />
        </div>
        设置
      </router-link>
    </ul>
  </div>
</template>

<script lang="ts">
import { TagHelper } from "@/mixins/TagHelper";
import { mixins } from "vue-class-component";
import { Component, Prop } from "vue-property-decorator";
import Icon from "../Icon.vue";

@Component({
  components: { Icon },
})
export default class Tags extends mixins(TagHelper) {
   @Prop({required: true}) readonly type!: string;
  //为了保证在本组件下不修改其他组件传来的值

  get taglist() {
    if (this.type === '-'){
      return this.$store.state.paymentList;
      }
    else{
      return this.$store.state.incomeList;
      }
  }
  get tagType() {
    return this.type;
  }
  created() {
    this.$store.commit("fetchTags");
  }
  previousTag: Tag = { id: "", name: "" , iconName: ""};

  toggleTag(tag: Tag) {
    if (this.previousTag.id === '') {
      this.previousTag = tag;
      this.$emit("update:value", this.previousTag);
      return;
    }
    if (this.previousTag !== tag) {
      this.previousTag = tag;
      this.$emit("update:value", this.previousTag);
      return;
    }
  }
  createTag(){
    //TODO 后期标签可选
      const name = window.prompt('请输入标签名')
      if (!name){
         return window.alert('标签不能为空');
      }
      this.$store.commit('createTag', {name,iconName:'add'});

  }
}
</script>

<style lang="scss" scoped>
@import "~@/assets/style/helper.scss";
.tags {
  flex-grow: 1;
  font-size: 14px;
  padding: 0px;
  display: flex;
  flex-direction: column;
  overflow: auto;
  background: white;
  > .current {
    display: flex;
    flex-wrap: wrap;
    flex-direction: row;
    > .labels {
      background: #fffffd;
      width: 25%;
      height: 80px;
      font-size: 12px;
      display: flex;
      flex-direction: column;
      justify-items: center;
      align-items: center;
      padding-top: 12px;
      &.selected {
        .icon-radius {
          background: #fdd844;
          color: black;
          .icon {
            color: #000000;
          }
        }
      }
      .icon-radius {
        background: #f7f5f6;

        width: 48px;
        height: 48px;
        border-radius: 50%;
        .icon {
          margin-left: 6px;
          margin-top: 6px;
          width: 36px;
          height: 36px;
          color: #636060;
        }
      }
        
      &.link{
        color: black;
        .link-radius{
            > .icon{
              color: black;

            }

        }
      }
    }
  }
  > .new {
    padding-top: 16px;
    button {
      background: transparent;
      border: none;
      color: #999;
      border-bottom: 1px solid;
      padding: 0 4px;
    }
  }
}
</style>

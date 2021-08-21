<template>
  <div class="tabs-wrapper">
    <Icon name="whale"></Icon>
    <ul class="tabs">
      <li v-for="item in dataSource" :key="item.value" class="tabs-item"
          :class="liClass(item)" @click="select(item)">{{item.text}}</li>
    </ul>
  </div>
</template>

<script lang="ts">
  import Vue from 'vue';
  import {Component, Prop} from 'vue-property-decorator';

  type DataSourceItem = {text: string, value: string};

  @Component
  export default class Tabs extends Vue {
    @Prop({required: true, type: Array}) dataSource!: DataSourceItem[];
    @Prop(String) readonly value!: string;
    @Prop(String) classPrefix?: string;

    liClass(item: DataSourceItem) {
      return {
        selected: item.value === this.value,
        [this.classPrefix+'-tabs-item']: this.classPrefix
      }
    }

    select(item: DataSourceItem) {
      this.$emit('update:value', item.value);
    }
  }
</script>

<style lang="scss" scoped>
  @import "~@/assets/style/helper.scss";

  .tabs-wrapper {
    position: relative;
    .icon {
      width: 36px;
      height: 36px;
      position: absolute;
      color: white;
      top: 32px;
      left: 16px;
      transform: translateY(-50%);
    }
  }
  .tabs {
    background: $color-highlight;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 20px;
    color: white;

    &-item {
      width: 20%;
      height: 64px;
      display: flex;
      justify-content: center;
      align-items: center;
      position: relative;
      &.selected {
        font-weight: bolder;
      }
      &.selected::after {
        content: '';
        position: absolute;
        bottom: 0;
        left: 0;
        width: 100%;
        height: 4px;
        background: whitesmoke;
      }
    }
  }
</style>
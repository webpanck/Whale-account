<template>
  <label class="formItem">
    <template v-if="type === 'date'">
      <Icon name="date"></Icon>
      <span class="name">{{this.fieldName}}</span>
      <input :type="type || 'text'"
             :value="x(value)"
             @input="onValueChanged($event.target.value)"
             :placeholder="placeholder">
    </template>
    <template v-else>
      <Icon name="note"></Icon>
      <span class="name">{{this.fieldName}}</span>
      <input :type="type || 'text'"
             :value="value"
             @input="onValueChanged($event.target.value)"
             :placeholder="placeholder">
    </template>
  </label>
</template>

<script lang="ts">
  import Vue from 'vue';
  import {Component, Prop, Watch} from 'vue-property-decorator';
  import dayjs from 'dayjs';

  @Component
  export default class FormItem extends Vue {
    @Prop({default: ''}) readonly value!: string;
    @Prop({required: true}) fieldName!: string;
    @Prop() placeholder?: string;
    @Prop() type?: string;

    onValueChanged(value: string) {
      this.$emit('update:value', value);
    }
    x(isoString: string) {
      return dayjs(isoString).format('YYYY-MM-DD');
    }
  }
</script>

<style lang="scss" scoped>
  @import "~@/assets/style/helper.scss";

  .formItem {
    font-size: 14px;
    display: flex;
    padding-left: 16px;
    align-items: center;
    .icon {
      color: darken($color-highlight, 20%);
    }
    .name {
      padding-right: 16px;
    }
    input {
      height: 40px;
      flex-grow: 1;
      background: transparent;
      border: none;
    }
  }
</style>
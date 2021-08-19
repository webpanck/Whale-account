<template>
  <Layout class-prefix="layout">
    <NumberPad :value.sync="record.amount" @submit="saveRecord"></NumberPad>
    <Tabs :data-source="recordTypeList" :value.sync="record.type"></Tabs>
    <div class="notes">
      <FormItem @update:value="onUpdateNotes" :value="record.notes"
                field-name="备注" placeholder="在这里输入备注"></FormItem>
    </div>
    <div class="createdAt">
      <FormItem :value.sync="record.createdAt" type="date" field-name="日期"></FormItem>
    </div>
    <Tags @update:value="onUpdateTags"></Tags>
  </Layout>
</template>

<script lang="ts">
  import Vue from 'vue'
  import NumberPad from '@/components/Money/NumberPad.vue';
  import FormItem from '@/components/Money/FormItem.vue';
  import Tags from '@/components/Money/Tags.vue';
  import {Component} from 'vue-property-decorator';
  import recordTypeList from '@/constants/recordTypeList';
  import Tabs from '@/components/Tabs.vue';

  window.localStorage.setItem('version', '0.0.1');

  @Component({
    components: {Tabs, Tags, FormItem, NumberPad}
  })
  export default class Money extends Vue{
    get recordList() {
      return this.$store.state.recordList;
    }
    recordTypeList = recordTypeList;
    record: RecordItem = {
      tags: [], notes: '', type: '-', amount: 0, createdAt: new Date().toISOString()
    };
    created() {
      this.$store.commit('fetchRecords');
    }
    onUpdateNotes(value: string) {
      this.record.notes = value;
    }
    onUpdateTags(value: Tag[]) {
      this.record.tags = value;
    }
    saveRecord() {
      if(!this.record.tags || this.record.tags.length === 0) {
        return window.alert('请选择至少一个标签');
      }
      this.$store.commit('createRecord', this.record);
      if(this.$store.state.createRecordError === null) {
        window.alert('添加成功！');
        this.record.notes = '';
      }
    }
  }
</script>

<style lang="scss" scoped>
  ::v-deep .layout-content {
    display: flex;
    flex-direction: column-reverse;
  }
  .notes {
    padding: 12px 0;
  }
</style>

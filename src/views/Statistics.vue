<template>
    <Layout>
      <Tabs class-prefix="type" :data-source="recordTypeList" :value.sync="type"></Tabs>
      <Tabs class-prefix="interval" :data-source="intervalList" :value.sync="interval"></Tabs>
      <ol>
        <li v-for="(group, index) in result" :key="index">
          <h3 class="title">{{group.title}}</h3>
          <ol>
            <li class="record" v-for="item in group.items" :key="item.id">
              <span>{{tagString(item.tags)}}</span>
              <span class="notes" :style="{marginRight:'auto'}">{{item.notes}}</span>
              <span>¥{{item.amount}}</span>
            </li>
          </ol>
        </li>
      </ol>
    </Layout>
</template>

<script lang="ts">
  import Vue from 'vue';
  import {Component} from 'vue-property-decorator';
  import Tabs from '@/components/Tabs.vue';
  import intervalList from '@/constants/intervalList';
  import recordTypeList from '@/constants/recordTypeList';

  @Component({
    components: {Tabs}
  })
  export default class Statistics extends Vue {
    tagString(tags: Tag[]) {
      return tags.length === 0 ? '无' : tags.join(',');
    }
    get recordList() {
      return this.$store.state.recordList;
    }
    get result() {
      const {recordList} = this;
      type HashTableValue = {title: string, items: RecordItem[]};
      const hashTable: {[key: string]: HashTableValue} = {};
      for(let i=0; i<recordList.length; i++) {
        const [date, time] = recordList[i].createdAt!.split('T');
        hashTable[date] = hashTable[date] || {title: date, items: []};
        hashTable[date].items.push(recordList[i]);
      }
      console.log(hashTable);
      return hashTable;
    }
    beforeCreate() {
      this.$store.commit('fetchRecords');
    }

    type = '-';
    interval = 'day';
    intervalList = intervalList;
    recordTypeList = recordTypeList;
  }
</script>

<style lang="scss" scoped>
  ::v-deep {
    .type-tabs-item {
      background: white;
      &.selected {
        background: #c4c4c4;
        &::after {
          display: none;
        }
      }
    }
    .interval-tabs-item {
      height: 48px;
    }
  }
  %item {
    padding: 0 16px;
    min-height: 40px;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  .title {
    @extend %item;
  }
  .record {
    @extend %item;
    background: white;
  }
  .notes {
    margin-right: auto;
    margin-left: 16px;
    color: #999;
  }
</style>
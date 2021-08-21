<template>
    <Layout>
      <Tabs class-prefix="type" :data-source="recordTypeList" :value.sync="type"></Tabs>
      <div class="chart-wrapper" ref="chartWrapper">
        <Chart class="chart" :options="chartOptions"></Chart>
      </div>
      <ol v-if="groupedList.length > 0">
        <li v-for="(group, index) in groupedList" :key="index">
          <h3 class="title">
            {{beautify(group.title)}}
            <span>¥{{group.total}}</span>
          </h3>
          <ol>
            <li class="record" v-for="item in group.items" :key="item.id">
              <span>{{tagString(item.tags)}}</span>
              <span class="notes" :style="{marginRight:'auto'}">{{item.notes}}</span>
              <span>¥{{item.amount}}</span>
            </li>
          </ol>
        </li>
      </ol>
      <div v-else class="noResult">
        目前没有任何记录哦～
      </div>
    </Layout>
</template>

<script lang="ts">
  import Vue from 'vue';
  import {Component} from 'vue-property-decorator';
  import Tabs from '@/components/Tabs.vue';
  import recordTypeList from '@/constants/recordTypeList';
  import dayjs from 'dayjs';
  import clone from '@/lib/clone';
  import Chart from '@/components/Chart.vue';
  import _ from 'lodash';

  @Component({
    components: {Tabs, Chart}
  })
  export default class Statistics extends Vue {
    beautify(string: string) {
      const day = dayjs(string);
      const now = dayjs();
      if(day.isSame(now, 'day')) {
        return '今天';
      } else if(day.isSame(now.subtract(1, 'day'), 'day')) {
        return '昨天';
      } else if(day.isSame(now.subtract(2, 'day'), 'day')) {
        return '前天';
      } else if(day.isSame(now, 'year')) {
        return day.format('M月D日');
      } else {
        return day.format('YYYY年M月D日');
      }
    }
    tagString(tags: Tag[]) {
      const tagNames:string[] = [];
      if(tags.length === 0) {
        return '无';
      } else {
        tags.forEach(tag => {
          tagNames.push(tag.name);
        })
        return tagNames.join(',');
      }
    }
    get keyValueList() {
      const today = new Date();
      const array = [];
      for(let i=0; i<= 29; i++) {
        const dateString = dayjs(today).subtract(i, 'day').format('YYYY-MM-DD');
        const found = _.find(this.groupedList, {title: dateString});
        array.push({key: dateString, value: found ? found.total : 0});
      }
      array.reverse();
      return array;
    }
    get chartOptions() {
      const keys = this.keyValueList.map(item => dayjs(item.key).format('M-D'));
      const values = this.keyValueList.map(item => item.value);
      return {
        grid: {
          left: 0,
          right: 0
        },
        xAxis: {
          type: 'category',
          data: keys,
          axisTick: {alignWithLabel: true},
          axisLine: {lineStyle: {color: '#666'}}
        },
        yAxis: {
          type: 'value',
          show: false
        },
        tooltip: {
          show: true,
          formatter: '{b}: {c}'
        },
        series: [{
          data: values,
          type: 'line',
          symbolSize: 15,
          itemStyle: {color: '#666'}
        }]
      }
    }
    mounted() {
      const div = (this.$refs.chartWrapper as HTMLDivElement);
      div.scrollLeft = div.scrollWidth;
    }
    get recordList() {
      return this.$store.state.recordList;
    }
    get groupedList() {
      const {recordList} = this;
      if(recordList.length === 0) {return [];}
      const newList = clone(recordList)
          .filter((r:RecordItem) => r.type === this.type)
          .sort((a: RecordItem, b: RecordItem) => dayjs(b.createdAt).valueOf() - dayjs(a.createdAt).valueOf());
      if(newList.length === 0) {return [] as Result;}
      type Result = {title: string, total?: number, items: RecordItem[]}[];
      const result: Result = [{title: dayjs(newList[0].createdAt).format('YYYY-MM-DD'), items: [newList[0]]}];
      for(let i=1; i<newList.length; i++) {
        const current = newList[i];
        const last = result[result.length - 1];
        if(dayjs(last.title).isSame(dayjs(current.createdAt), 'day')) {
          last.items.push(current);
        } else {
          result.push({title: dayjs(current.createdAt).format('YYYY-MM-DD'), items: [current]});
        }
      }
      result.map(group => {
        group.total = group.items.reduce((sum, item) => sum + item.amount, 0);
      })
      return result;
    }
    beforeCreate() {
      this.$store.commit('fetchRecords');
    }

    type = '-';
    recordTypeList = recordTypeList;
  }
</script>

<style lang="scss" scoped>
  .chart {
    width: 430%;
    &-wrapper {
      overflow: auto;
    }
  }
  ::v-deep {
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
  .noResult {
    padding: 16px;
    text-align: center;
  }
</style>
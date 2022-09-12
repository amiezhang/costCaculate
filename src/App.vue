<template>
  <div class="title">总费用</div>
  <div class="content">
    <input style="width: 100%" type="number" v-model="totalFee" placeholder="¥总费用" />
  </div>
  <div class="lou" v-for="(lou, key) in list" :key="key">
    <div class="title">
      {{nameMap[key]}}楼
      <span :class="getReduceDu(key) < 0 ? 'title-tips error' : 'title-tips'">层总用量:{{getReduceDu(key)}}</span>
      <span class="add" @click="() => add(key)">添加表</span>
    </div>
    <div v-for="(biao, biaoKey) in lou" :key="biaoKey" class="content">
      <span>表{{biaoKey+1}}</span>
      <span>上期:</span>
      <input type="number" v-model="lou[biaoKey][0]" placeholder="上期度数" />
      <span>本期:</span>
      <input type="number" v-model="lou[biaoKey][1]" placeholder="本期度数" />
      <span class="delete" @click="() => remove(key, biaoKey)">删除表</span>
    </div>
  </div>
  <div style="font-size: .6rem">
    结果
    <span class="add" @click="copy">复制</span>
  </div>
  <div class="result">
    <div>楼总度数：{{sumDu}}, 楼总费用：¥ {{totalFee || '0.00'}}</div>
    <div>每度电：¥ {{(sumDu && totalFee) ? pricePerDu : '-'}}</div>
    <div v-for="(lou, key) in dataPerLou" :key="key" :class="lou.louSum < 0 ? 'error' : ''">
      {{nameMap[key]}}楼: 层总度数：{{lou.louSum}}，层费用：¥ {{lou.priceSum}}
    </div>
  </div>

</template>

<script>
export default {
  name: 'App',
  computed: {
    sumDu() {
      return this.list.reduce((sum, lou) => {
        return sum + lou.reduce((louSum, [pre, cur]) => {
          const curDu = cur - pre;
          return louSum + (Number.isNaN(curDu) ? 0 : curDu)
        }, 0)
      }, 0)
    }, 
    pricePerDu() {
      const price = (this.totalFee || 0) / this.sumDu;
      return (price || 0).toFixed(3);
    }, 
    dataPerLou() {
      return this.list.map((lou) => {
        const louSum = lou.reduce((sum, [pre, cur]) => {
          const curDu = cur - pre;
          return sum + (Number.isNaN(curDu) ? 0 : curDu);
        }, 0);
        return {louSum, priceSum: (louSum * this.pricePerDu || 0).toFixed(2)}
      })
    }
  },
  data() {
    return {
      nameMap: ['一', '二', '三', '四', '五'],
      totalFee: undefined,
      list: [[[]], [[], [], []], [[]], [[]], [[]]]
    }
  },
  methods: {
    getReduceDu(louKey) {
      return this.list[louKey].reduce((sum, biao) => {
        return sum + ((biao[1] - biao[0]) || 0)
      }, 0).toFixed(2);
    },
    add(louKey) {
      this.list[louKey].push([]);
    },
    remove(louKey, key) {
      this.list[louKey].splice(key, 1);
    },
    copy() {
      let bool = document.execCommand('copy')
      if (!bool) {
          return
      } else {
        const val = document.querySelector('.result').innerText;
        const inputEle = document.createElement('textarea');
        document.body.appendChild(inputEle);
        inputEle.value = val;
        inputEle.select();
        document.execCommand('copy');
        document.body.removeChild(inputEle);
        alert('复制完成')
      }
    }
  }
}
</script>

<style lang="less">
body {
  .title {
    font-size: .5rem;
    display: flex;
    align-items: center;
    .title-tips {
      font-size: .4rem;
      margin: 0 .6rem;
    }
  }
  .error {
    color: red;
    text-decoration: underline;
  }
  .add {
    color: blue;
    font-size: .3rem;
  }
  .delete {
    color: red;
    font-size: .3rem;
  }
  input {
    height: .8rem;
    font-size: .3rem;
  }
  .content {
    display: flex;
    align-items: center;
    justify-content: space-between;
    font-size: .3rem;
  }
  .lou {
    margin-bottom: .3rem;
    input {
      width: 1.7rem;
    }
    &:not(:first-child) {
      .content {
        margin-bottom: .2rem;
      }
    }
  }
  .result {
    font-size: .36rem;
  }
}
</style>

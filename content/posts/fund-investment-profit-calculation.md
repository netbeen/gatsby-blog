---
title: 基金投资收益率计算
date: '2021-11-14'
description: '本文分析了集中常见的基金收益率计算方法，比较了不同计算方法的适用场景的差异，引申出年化收益率是基金投资者需要最重点关注的指标，最后给出了如何计算年化收益率的简便方法'
tags: ['fund']
category: etc
---

## 前言

你在阅读本文前需要了解的基础知识：基金投资基本经验

## 为什么要计算基金投资收益率

很多人看到本文的标题，就想笑，感觉这又是一篇水文，实际上并不是。我们做任何事情，都需要讲究效率，而投资收益率之所以重要，是因为投资收益率就是投资的效率。这个效率会影响我们日常生活里的诸多决策，比如：

> 当月的工资到手后，是存入银行买理财？还是投入股市去折腾？

这个问题的答案其实很简单，取决于你投资的效率，将资金配置在投资效率最高的渠道，显而易见可以获得最大的收益。由于我的投资方式以基金投资为主，所以会格外看中基金投资收益率

## 收益率的计算

### 单利计算

但凡是接触过任何形式投资的初阶投资者，都知道一个最基础的收益率计算公式，假设收益率用 `r` 表示，总成本用 `C` 表示：

```text
r = 总盈利 / C
``` 

这公式计算的收益率被称为 `单利`，固然正确，但是它的适用场景很有限，仅在总成本在期初一次性投入，本息一次性在期末提取时才生效，这显然不适合基金投资，因为基金投资很少是一锤子买卖，大多需要长期分段买入

### 复利计算

稍微进阶点的投资者，大多听说过 `复利` 这个概念，这与各路理财大V们的大力宣传分不开，相传爱因斯坦曾经提及"复利是世界第八大奇迹"，相比单利，复利可以在长时间跨度里，更加准确地衡量投资效率，但是复利的计算是比较复杂的，假设每次买入的金额分别为 `c1,c2,c3,...,cn`，每一笔资金的持有天数为 `t1,t2,t3,...,tn`，日收益率用`r` 表示 

```text
总盈利 = c1(1+r)^t1 + c2(1+r)^t2 + c3(1+r)^t3 + ... + cn(1+r)^tn
```

由于上述方程中，总盈利、`c1,c2,c3,...,cn` 、 `t1,t2,t3,...,tn` 都为已知量，所以这是一个 一元 `t` 次方程，如果 `t` 的值不大，还可以用对应的求根公式求解，当 `t` 值很大的时候，求根公式求解会相当低效，一般是利用这个形式的方程的单调递增的特性，利用程序，使用 `二分法逼近取极限` 的方式，不断逼近真实的 `r` ，最终将得到一个与真实 `r` 误差在可接受范围内的结果

上述的算法可以算出多次买入，一次性清仓的交易方式下的投资收益率。过去几年中，我也使用这个方法去计算投资收益率来指导我自己的投资决策，渐渐地我发现这个方式有一些死板，由于算法限制，要求我必须在卖出时一次性清仓，这就要求卖出的时间点预估必须准确，但这往往也是很困难的。而A股市场经常会出现长时间内，在一个区间上蹿下跳的"震荡猴盘"现象，这就给了 `网格交易` 策略一定的舞台，可以定期来回收割。于是我就开始思考，如何持续性买入卖出的情况下，依旧准确算出投资收益率呢？

### 转换思路

无论是简单的单利计算法，还是复杂的复利计算法，其本质都是去试图解答 `每一块钱，在投入的这段时间内，赚了多少钱` 这个问题。之所以单利计算法简单，是因为单利计算法假设所有时间点上的资金投入是固定的，而复利计算法需要去适应资金投入的变化，最终都是为了求解出资金在 `时间` 和 `金额` 两个维度的 `占用` 情况。可以画一个二维直角坐标系，x轴表示时间，y轴表示金额，将y轴金额看做x的函数，即f(x)，那么这个 `占用` 情况实际上就是 `在 [0, tn] 内，求 f(x) 的定积分` ，由于这个函数f(x)是一个无规律曲线，很难用数学公式表达，但这对计算机来说却很简单

得益于基金的特性，每天只有一个净值，所以一天中的市值是恒定的，所以将上述函数f(x)按天细分，将每日的市值累加，就可以得到一个非常接近上述 `定积分` 的数字，即
```text
r = 总盈利 / ∑(每日市值)
```

## 基金净值的计算

光知道如何计算收益率还不够，中国的基金里还有另一个制度会影响收益率的计算，买过基金的朋友都清楚，基金每天会产生一个价格，购买基金时根据这个价格成交，这个价格就是 `单位净值`，而大家在雪球等交易平台上都可以看到两个净值，除了 `单位净值` 之外，还有一个 `累计净值` ，之所以会出现这两个净值的差异是因为基金公司会有两项吸引初阶韭菜投资者的骚操作，这两个操作都是为了降低基金的单位净值，以达成吸引更多投资者的目的，这些策略虽然不会对我的投资决策构成影响，但是确实会影响收益率，所以在计算的时候也需要考虑

### 基金分红

基金分红的本质就是基金通过将一部分盈利以现金形式返给投资者，达到降低基金单位净值的目的。当发生基金分红时

- 落袋盈利增加
- 市值下降
- 单位净值下降

### 基金拆分

基金拆分相比基金分红更为直接，直接将基金按照特定的比例1拆n，达到降低基金单位净值的目的

- 单位净值下降
- 持仓数量上升
- 市值不变

## 快速计算收益率的库 fund-tools

为了解决如下的几个问题：

- 基金投资收益率计算复杂
- 基金的分红拆分逻辑会进一步增加第一个问题的复杂度
- 基金数据爬取困难

我开发了一个库，名为 fund-tools 
- NPM: [https://www.npmjs.com/package/fund-tools](https://www.npmjs.com/package/fund-tools) 
- GitHub: [https://github.com/netbeen/fund-tools](https://github.com/netbeen/fund-tools) 

整合了 基金数据爬虫、分红&拆分自动化处理、收益率&年化收益率计算、手续费计算 等功能，可以运行在浏览器环境和 Node.js runtime 里，简单介绍一下使用方法，更详细的API介绍请查阅 README：

### 安装

```shell
npm install --save fund-tools
```

### 使用

```javascript
import { 
    fetchUnitPriceByIdentifier, 
    fetchSplitByIdentifier, 
    fetchDividendByIdentifier, 
    calcReturn,
    OPERATION_DIRECTION_BUY,
    OPERATION_DIRECTION_SELL
} from 'fund-tools';
import * as dayjs from 'dayjs';

// 以 160119 中证500为例
const fundIdentifier = '160119'

// 通过爬虫抓取基金历史数据，包括单位净值、分红、拆分
const unitPrices = await fetchUnitPriceByIdentifier(fundIdentifier);
const dividends = await fetchDividendByIdentifier(fundIdentifier);
const splits = await fetchSplitByIdentifier(fundIdentifier);

// 录入自己的历史交易
const operations = [{
    date: dayjs('2021-06-09'), volume: 2958.54, direction: OPERATION_DIRECTION_BUY, commission: 5.99
}, {
    date: dayjs('2021-07-12'), volume: 3378.46, direction: OPERATION_DIRECTION_BUY, commission: 6.54
}, {
    date: dayjs('2021-08-09'), volume: 3957.39, direction: OPERATION_DIRECTION_BUY, commission: 7.53
}, {
    date: dayjs('2021-11-09'), volume: 3197.19, direction: OPERATION_DIRECTION_SELL, commission: 5.99
}];

const calcResult = calcReturn(unitPrices, dividends, splits, operations);
console.log(calcResult);
/**
  {
      unitPrice: 1.597, // 当前单位净值
      unitCost: 1.6110785712273878, // 当前单位净值
      volume: 13491.58, // 持仓数量
      totalCommission: 26.050000000000004, // 总显性手续费
      totalDividend: 0, // 总分红
      positionReturn: -189.94217000000165,  // 持仓收益
      positionCost: 21735.995430000003, // 持仓总成本
      positionValue: 21546.05326,   // 总市值
      positionRateOfReturn: -0.008738600015430793,  // 持仓收益率
      exitReturn: 0,    // 落袋收益
      totalReturn: -189.94217000000165, // 总收益
      totalAnnualizedRateOfReturn: -0.054593138863008296    // 总年化收益率
  }
 */
```

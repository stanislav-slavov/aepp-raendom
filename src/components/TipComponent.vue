<template>
  <div class="tip__item pt-3 pb-3 mb-3">
    <div class="container">
    <div class="row">
      <div class="col-sm left">
        <div class="note">
          {{tip.note}}
        </div>
        <div><span class="amount">+ {{ tip.amount }}</span><span class="sender-info"> æids from {{ tip.sender }}</span></div>
        <div class="url pt-2">
          {{tip.url}}
        </div>
        <div class="clearfix">
          <div class="date float-left mr-3">{{ new Date(tip.received_at).toLocaleString('en-US', { hourCycle: 'h24' }) }}</div>
          <div class="date float-left mr-3 heart"><i class="fas fa-heart"></i> {{ tip.amount }}</div>
        </div>
      </div>
      <div class="col-sm">
        {{tip.url}}
      </div>
    </div>
  </div>
  </div>
</template>

<script>

export default {
name: "TipComponent",
props: ['tip'],
}
</script>

<style lang="scss">
.tip__item{
  background-color: #0B0909;
  .container{
    margin-top: 0;
  }
  .note{
    color: #F0B800;
    font-size: .75rem;
    height: 3rem;
    overflow: auto;
    font-weight: 700;
  }
  .url{
    font-size: .5rem;
  }
  .col-sm.left{
    text-align: right;
  }
  .amount{
    font-weight: 700;
    color: #F0B800;
    font-size: .75rem;
  }
  .sender-info{
    font-size: .75rem;
  }
  .date{
    font-size: .6125rem;
    font-weight: 700;
    text-align: left;
  }
  .fas.fa-heart{
    color: #F0B800;
  }
 
}
</style>

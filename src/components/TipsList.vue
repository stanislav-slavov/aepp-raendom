<template>
  <div>
    <div class="content__wrapper">

      <div class="container mt-0">
        <div class="row head-info-padded">
          <h2>Information is power. First-hand information is even better.</h2>

          <p>
            Get real-time global updates about the corona virus sourced from beyond just news outlets: conversations by real people on the ground, all in one place.
          </p>
          <br>
          <p>
            This information hub enables users to incentivize informative and accurate updates about the corona virus from all over the world—whether created by news organizations or regular people, like you. Learn more about how you can start contributing to a more truthful web
          </p>
        </div>
      </div>

      <div class="content__container">
        <div class="actions-line p-3 clearfix">
          <div class="float-left pt-1">NEWS</div>
          <div class="float-right">
            <a class="pr-3" v-on:click="sortLatest()">LATEST</a>
            <a class="pr-3" v-on:click="sortHighestRated()">HIGHEST RATED</a>
          </div>
          <div class="float-right"><input class="search__input p-1" v-model="searchTerm" type="text" placeholder="Search for a tip record..." id="tips-search"></div>
        </div>
        <div class="spinner-border text-light" v-show="!client || showLoading" role="status">
          <span class="sr-only">Loading...</span>
        </div>
        <tip-component v-for="(tip,index) in filteredTips" :key="index" v-bind:tip="tip"></tip-component>
        <div class="no-results"  v-if="filteredTips !== null && filteredTips.length == 0"><span>There are no results found</span></div>
      </div>
    </div>
  </div>
</template>

<script>
  import 'prismjs'
  import 'prismjs/themes/prism.css'
  import 'prismjs/components/prism-reason.min.js'
  import 'vue-prism-editor/dist/VuePrismEditor.css' // import the styles
  import PrismEditor from 'vue-prism-editor'

  import {Universal} from '@aeternity/aepp-sdk/es/ae/universal'
  import * as Crypto from '@aeternity/aepp-sdk/es/utils/crypto'
  import {AeButton, AeInput, AeLabel, AeList, AeListItem, AeCheck} from '@aeternity/aepp-components'

  import CONTRACT_TIP_ANY from '../../contracts/tip_any_basic.aes';

  import BiggerLoader from './BiggerLoader'
  import TipComponent from './TipComponent'

  export default {
    name: 'TipsList',
      components: {
        AeInput,
        AeButton,
        AeLabel,
        AeList,
        AeListItem,
        AeCheck,
        PrismEditor,
        BiggerLoader,
        TipComponent
      },
      data() {
        return {
          nodeUrl: "https://mainnet.aeternal.io",
          keypair: Crypto.generateKeyPair(),
          contractCode: CONTRACT_TIP_ANY,
          contractInstance: null,
          contractAddress: 'ct_cT9mSpx9989Js39ag45fih2daephb7YsicsvNdUdEB156gT5C',
          address: null,
          client: null,
          showLoading: true,
          loadingProgress: "",
          allErrors: [],
          tips: null,
          searchTerm: ''
        }
      },
      computed: {
        filteredTips() {
          if(this.searchTerm.trim().length == 0){
            return this.tips
          }
          let term = this.searchTerm.toLowerCase();

          let urlSearchResults = this.tips.filter(tip => {
            if(typeof tip.url !== 'undefined'){
              return tip.url.toLowerCase().includes(term)
            }
            return false
          })
          let senderSearchResults = this.tips.filter(tip => {
             if(typeof tip.sender !== 'undefined'){
              return tip.sender.toLowerCase().includes(term)
            }
            return false
          })
          let noteSearchResults = this.tips.filter(tip => {
            if(typeof tip.note !== 'undefined'){
              return tip.note.toLowerCase().includes(term)
            }
            return false
          })
          //We convert the result array to Set in order to remove duplicate records
          let convertResultToSet = new Set([...urlSearchResults, ...senderSearchResults, ...noteSearchResults]);
          return [...convertResultToSet];
        }
      },
      methods: {
        handleContractError(message) {
          this.showLoading = false;

          const matchFunctionNotFound = message.toString().match(/\.(\w*) is not a function/);
          console.log(matchFunctionNotFound);
          if (matchFunctionNotFound) message = matchFunctionNotFound[1] + " function not defined (maybe commented out?)";

          this.allErrors.push(message);
        },
        async getContractInstanceAtAddress(contractAddress) {
          try {
            this.contractInstance = await this.client.getContractInstance(this.contractCode, { contractAddress });
          } catch (e) {
            console.log(e)
          }
        },
        async getContractState() {
          try {
            const contractState = await this.contractInstance.methods.get_state();
            return contractState.decodedResult;
          } catch (e) {
            console.log(e)
          }
        },
        convertToAE(balance) {
          return +(balance / 10 ** 18).toFixed(7);
        },
        parseTips(tips) {
          let temp = [];
          tips.forEach(t => {
            let p = t[1]
            p.url = t[0][0]
            p.amount = this.convertToAE(p.amount)
            temp.push(p)
          });
          return temp;
        },
        sortLatest() {
          // sort by timestamp
          this.tips.sort((a,b) => (a.received_at < b.received_at) ? 1 : -1)
        },
        sortHighestRated() {
          // sort by most tipped amount combined
          this.tips.sort((a, b) => (a.amount < b.amount) ? 1 : -1)
        }
      },
      async created() {
          this.loadingProgress = "fetching tips";
          this.client = await Universal({
              url: this.nodeUrl,
              internalUrl: this.nodeUrl,
              keypair: this.keypair,
              networkId: 'ae_mainnet',
              compilerUrl: "https://compiler.aepps.com"
          });

          await this.getContractInstanceAtAddress(this.contractAddress);
          this.tips = await this.getContractState();
          this.tips = this.parseTips(this.tips.tips);
          this.showLoading = false;
      },
  }
</script>

<style lang="scss">
  @import './styles'; 

  .content__wrapper{
    background-color: #26221E;
    width: 100vw;
    color: white;
    min-height: 100vw;
    .content__container{
      width: 70vw;
      margin: 0 auto;
      background-color: #323232;
      position: relative;
      .spinner-border.text-light,.no-results{
        position: fixed;
        margin: auto;
        top: 0; left: 0; bottom: 0; right: 0;
      }
      .actions-line{
        background-color: #323232;
        input{
          font-size: .75rem;
          margin-right: 1rem;
          width: 15rem;
        }
        h4{
          margin-bottom: 0;
        }
        a{
          font-size: .6125rem;
          &:hover{
            cursor: pointer;
            color: #F0B800;
          }
          &:active{
            color: red;
          }
        }
      }
    }
    .no-results{
      text-align: center;
      span{
        position: relative;
        top: 50%;
        transform: translateY(-50%);
      }
    } 
  }
 
  #app-content {
    margin-top: 2rem;
    max-width: 1200px;
    padding: 0 20px 20px;
  }

  ul {
    list-style: none;
  }

  .errors div {
    padding: 0.5rem 1rem;
    background: rgba(255, 0, 0, 0.6);
    color: white;
    margin: 1rem 0;
    display: flex;
    align-items: center;
    text-align: left;
  }

  .errors div::before {
    content: "!";
    font-size: 2rem;
    font-weight: bold;
    margin-right: 1rem;
  }

  .overlay-loader {
    top: 0;
    left: 0;
    bottom: 0;
    right: 0;
    z-index: 100;
    width: 100%;
    position: absolute;
    display: flex;
    justify-content: center;
    align-items: flex-start;
    padding-top: 150px;
    background-color: rgba(255, 255, 255, 0.85);
    overflow: hidden;
  }

  input.ae-input {
    box-sizing: border-box;
  }

  .status-label {
    padding: 2px;
    border-radius: 2px;
  }

  .status-label.unclaimed {
    color: orange;
  }

  .status-label.repaid {
    color: limegreen;
  }

  .sender {
    font-weight: bold;
  }

  .actions {
    width:50%;
    margin-top: 5px;
  }

  .tipWebsiteHeader {
      margin-bottom:20px;
  }

  .ae-divider {
    background-color: #bbbbbb !important;
  }

  .domainFavicon {
    width:32px;
    margin-right:15px;
  }

  .noFavicon {
    font-size:0.8rem;
    height:32px;
    word-break: break-word;
  }

  .domainInfo {
    flex-grow:1;
  }

  .domain { 
    position:relative;
    cursor: pointer;
  }

  .domain:hover .full-domain {
      display:block;
  }

  .identicon {
    padding-top: 2px;
  }

  .full-domain {
    display:none;
    position:absolute;
    left:0;
    right: 0;
    top: 115%;
    background: #001833;
    color: #fff;
    padding: 5px;
    border-radius: 6px;
    word-break: break-word;
    white-space: normal;
    z-index: 15;
    font-size: 0.8rem;
  }

  .full-domain:after {
    content:"";
    border: solid black;
    border-width: 0 3px 3px 0;
    display: inline-block;
    padding: 3px;
    transform: rotate(-135deg);
    -webkit-transform: rotate(-135deg);
    -ms-transform: rotate(-135deg);
    position: absolute;
    top: -4px;
    left: 23px;
    background: inherit;
  }

  h6 {
    margin:0;
    word-break: break-word;
  }

  p {
    margin:0;
    font-weight:normal;
  }

  .ae-icon {
    color: #fff !important;
    width: 20px !important;
    height: 20px !important;
    font-size: 0.8rem;
    margin-right: 2px;
    border: 2px solid #dae1ea;
  }

  .balance {
    color: red;
    font-size: 0.5em;
    font-weight:bold;
  }

  .textarea {
    min-height: 60px;
  }

  .ae-badge {
    border-radius:20px;
    width:20%;
    justify-content:center;
    cursor: pointer;
    background:#e4e4e4;
  }

  .ae-badge.primary {
    background: #e4e4e4;
    color:#fff;
  }

  .ae-badge.alternative {
    background: #e4e4e4;
    color:#fff;
  }

  .sendTip { 
    margin-top:25px;
  }

  .tipSlider {
    margin:25px 0;
  }

  .container {
    margin-top: 3rem;
  }

  .hideSlider {
    opacity: 0.3;
  }

  .amount-container{
    position:relative;
  }

  .hideSlider .sliderOver {
    position:absolute;
    z-index:50;
    left:0;
    right:0;
    bottom:0;
    top:0;
  }

  .balanceInfo {
    margin-top:15px;
  }

  .btn-50 {
    width:50%;
  }

  .ae-address {
    font-weight: bold !important;
    color:#000;
  }

  .balance {
    font-size: 1rem;
    font-weight: bold;
    color:#000;
  }

  .balance:after {
    content:'AE'
  }

  small {
    font-size:.8rem;
  }

  .tip-note {
    -webkit-line-clamp: 2;
  }

  .mt-0 {
    margin-top: 0;
  }

  .mb-0 {
    margin-bottom: 0;
  }

  .tip-url {
    -webkit-line-clamp: 1;
  }
  .tip-url a{
    width: 32rem;
    display: block;
  }

  .tip-url, .tip-note{
    display: -webkit-box;
    overflow: hidden;
    text-overflow: ellipsis;
    -webkit-box-orient: vertical;
  }

  .search__container{
    text-align: left;
    padding: 0 .75rem .75rem .75rem;
  }
  .search__input{
    width: 20rem;
    padding: .375rem .75rem;
    font-size: 1rem;
  }
  .table th:first-child, .table td:first-child,
  .table th:nth-child(3), .table td:nth-child(3),
  .table th:nth-child(4), .table td:nth-child(4),
  .table th:nth-child(5), .table td:nth-child(5)
  {
    width: 10rem;
  }
  .table th:nth-child(2), .table td:nth-child(2){
    width: 35rem;
  }

  .head-info-padded {
    padding-top: 3rem;
    padding-bottom: 3rem;
    padding-left: 5rem;
    padding-right: 5rem;
  }
</style>

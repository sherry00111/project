<template>
  <div>
    <nav-header></nav-header>    
    <nav-bread>
        <span>Goods</span>
    </nav-bread>
    <div class="accessory-result-page accessory-page">
        <div class="container">
            <div class="filter-nav">
            <span class="sortby">Sort by:</span>
            <a href="javascript:void(0)" class="default cur">Default</a>
            <a href="javascript:void(0)" class="price" @click="sortGoods()">Price <svg class="icon icon-arrow-short" v-bind:class="{'sort-up':!sortFlag}"><use xlink:href="#icon-arrow-short"></use></svg></a>
            <a href="javascript:void(0)" class="filterby stopPop" @click="showFilterPop">Filter by</a>
            </div>
            <div class="accessory-result">
            <!-- filter -->
            <div class="filter stopPop" id="filter" v-bind:class="{'filterby-show':fifterBy}">
                <dl class="filter-price">
                    <dt>Price:</dt>
                    <dd><a href="javascript:void(0)" v-bind:class="{'cur':priceChecked=='all'}" @click="priceChecked='all'">All</a></dd>
                    <dd v-for="(price,index) in priceFilter" :key="index">
                        <a href="javascript:void(0)"  @click="setPriceFilter(index)" v-bind:class="{'cur':priceChecked==index}">{{price.startPrice}} - {{price.endPrice}}</a>
                    </dd>                
                </dl>
            </div>

            <!-- search result accessories list -->
            <div class="accessory-list-wrap">
                <div class="accessory-list col-4">
                <ul>
                    <li v-for="(item,index) in goodsList" :key="item.productId">
                        <div class="pic">
                            <a href="#"><img v-lazy="'/static/images/'+item.productImage" alt=""></a>
                        </div>
                        <div class="main">
                            <div class="name">{{item.productName}}</div>
                            <div class="price">{{item.salePrice}}</div>
                            <div class="btn-area">
                            <a href="javascript:;" class="btn btn--m" @click="addCart(item.productId)">加入购物车</a>
                            </div>
                        </div>
                    </li>
                </ul>
                <div v-infinite-scroll="loadMore" infinite-scroll-disabled="busy" infinite-scroll-distance="10">
                    <img src="./../assets/loading-cylon.svg" alt="" v-show="loading">
                </div>
                </div>
            </div>
            </div>
        </div>
    </div>
    <div class="md-overlay" v-show="overLayFlag" @click="closePop"></div>
    <modal v-bind:mdShow="mdShow" v-on:close="closeModal">
        <p slot="massage">请先登录，否则无法加入到购物车！</p>
        <div slot="btnGroup">
            <a href="javascript:;" class="btn btn--m" @click="mdShow=false">关闭</a>
        </div>
    </modal>
    <modal v-bind:mdShow="mdShowCart" v-on:close="closeModal">
        <p slot="message">
            <svg class="navbar-cart-logo">
                <use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#icon-cart"></use>
            </svg>
            <span>加入购物车成功！</span>
        </p>
        <div slot="btnGroup">
            <a href="javascript:;" class="btn btn--m" @click="mdShowCart=false">继续购物</a>
            <router-link class="btn btn--m" to='/cart'>查看购物车</router-link>
        </div>
    </modal>
    <nav-footer></nav-footer>
  </div>
</template>

<script>
import "./../assets/css/base.css"
import "./../assets/css/product.css"
import NavHeader from '@/components/NavHeader'
import NavFooter from '@/components/NavFooter'
import NavBread from '@/components/NavBread'
import Modal from '@/components/Modal'
import axios from 'axios'
export default {
  data () {
      return {
        goodsList:[],
        loading:false,
        priceFilter:[
            {
                startPrice:0.00,
                endPrice:100.00
            },
            {
                startPrice:100.00,
                endPrice:500.00
            },
            {
                startPrice:500.00,
                endPrice:1000.00
            },
            {
                startPrice:1000.00,
                endPrice:5000.00
            }            
        ],
        priceChecked:'all',
        fifterBy:false,
        overLayFlag:false,
        sortFlag:true,
        page:1,
        pageSize:5,
        busy: false,
        mdShow:false,
        mdShowCart:false
      }
  },
  components:{
      NavHeader,
      NavFooter,
      NavBread,
      Modal
  },
  mounted:function(){
    this.getGoodsList();
  },
  methods:{
      getGoodsList (flag) {
          var param = {
              sort:this.sortFlag?1:-1,
              page:this.page,
              pageSize:this.pageSize,
              priceLevel:this.priceChecked
          };
          this.loading=true;
          axios.get('/goods/list',{params:param}).then(result => {
            var res = result.data;
            this.loading=false;
            if(res.status=="0"){                
                if(flag){
                  this.goodsList=this.goodsList.concat(res.result.list);  
                  if(res.result.count==0){
                    this.busy=true;
                  }else{
                      this.busy=false;
                  }
                }else{
                    this.goodsList=res.result.list;
                    this.busy=false;
                }                
            }else{
              this.goodsList=[];  
            }            
          });
      },
      sortGoods () {
        this.sortFlag = !this.sortFlag;
        this.page=1;
        this.getGoodsList();
      },
      showFilterPop () {
        this.fifterBy=true,
        this.overLayFlag=true
      },
      setPriceFilter (index) {
        this.priceChecked=index;
        this.page=1;
        this.getGoodsList();
        this.closePop();
      },
      closePop () {
        this.fifterBy=false,
        this.overLayFlag=false
      },
      loadMore () {
          this.busy = true;
          setTimeout(() => {
              this.page++;
              this.getGoodsList(true);
          },500)
      },
      addCart (productId) {
          axios.post('/goods/addCart',{productId:productId}).then(result=>{
              var res = result.data;
              if(res.status=='0'){
                  this.mdShowCart=true;
                  this.$store.commit('updateCartCount',1);
              }else{
                  this.mdShow=true;
              }
          });
      },
      closeModal () {
          this.mdShow=false;
          this.mdShowCart=false;
      }
  }
}
</script>


<template>
  <div class="app-container">
    <Header></Header>
    <Goods 
    v-for="item in list" 
    :key="item.id" 
    :id="item.id"
    :title="item.goods_name" 
    :pic="item.goods_img" 
    :price="item.goods_price" 
    :state="item.goods_state" 
    :count="item.goods_count"
    @state-change="getNewState">
    </Goods>
    <Footer :isfull="fullState" :amount="amt" :all="total" @full-change="getFullState"></Footer>
  </div>
</template>

<script>
// 导入组件
import Header from '@/components/Header/Header.vue' 
import Goods from '@/components/Goods/Goods.vue'
import Footer from '@/components/Footer/Footer.vue'
import axios from 'axios'
import bus from '@/components/eventBus.js'
import { log } from 'console'
export default {
  // 生命周期函数
  created() {
    // 调用请求数据的方法
    this.initCartList()

    bus.$on('share',val => {
      this.list.some(item => {
        if(item.id === val.id) {
          item.goods_count = val.value
          return true
        }
      })
    })
  },
  // 数据
  data(){
    return {
      list:[]
    }
  },
  // 计算属性
  computed: {
    // 动态计算出全选的状态是 true 还是 false
    fullState() {
      return this.list.every(item => item.goods_state)
    },
    // 已勾选商品的总价格
    amt() {
      return this.list.filter(item => item.goods_state).reduce((total, item) => (total += item.goods_price * item.goods_count), 0)
    },
    // 已勾选商品的总数量
    total() {
      return this.list.filter(item => item.goods_state).reduce((t,item) => (t += item.goods_count), 0 )
    }
  },
  // 方法
  methods:{
    // 封装数据请求的方法
    async initCartList() {
      // 调用 axios 的 get 方法，请求列表数据
      const{ data:res } = await axios.get('https://www.escook.cn/api/cart')
      if(res.status === 200){
        this.list = res.list
      }
    },
    // 接收子组件传递过来的数据
    // e 的格式为 { id, value }
    getNewState(e) {
      this.list.some(item => {
        if(item.id === e.id){
          item.goods_state = e.value
        return true
        }   
      })
    },
    getFullState( val ) {
      this.list.forEach(item => item.goods_state = val )
    }
  },
  // 注册组件
  components:{
    Header,
    Goods,
    Footer
  }
}
</script>

<style lang="less" scoped>
.app-container {
  padding-top: 45px;
  padding-bottom: 50px;
}
</style>

<template>
  <view-box ref="viewBox" class="orderDetail">
    <x-header 
      slot="header" 
      :left-options="{showBack: true,backText:'返回'}" 
      style="width:100%;position:absolute;left:0;top:0;z-index:100;" 
      title="订单详情"
    ></x-header>
    <div style="padding-top:46px;padding-bottom:50px;">

      <div v-if = "OrderDetail.order">
        <group v-if="OrderDetail.order.order_status == 0" title="取消原因">
          <cell title="取消方" :value="OrderDetail.order.cancelled_type | cancelled_type"></cell>
          <cell title="取消原因" :value="OrderDetail.order.cancelled_reason"></cell>
        </group>

        <group title="收货信息">
          <cell title="收货人" :value="OrderDetail.order.receiver_name"></cell>
          <cell title="电话" >
            <a :href="'tel:'+OrderDetail.order.receiver_phone" slot="default">{{OrderDetail.order.receiver_phone}}</a>
          </cell>
          <cell title="收货地址："></cell>
          <p style="padding:0 15px 10px;color:#888">{{OrderDetail.order.receiver_address_detail}}</p>
          <cell  title="期望发货时间" :value="OrderDetail.order_detail[0].delivery_at" ></cell>
        </group>
        
        <group title="订单明细">
            <div class="weui_cell"  v-for="order in OrderDetail.order_detail"> 
              <div class="weui_cell_bd weui_cell_primary">
                <flexbox>
                  <flexbox-item :span="7">
                    <p>{{order.product_name}}<b v-if="order.npk_scale">({{order.npk_scale}})</b></p>
                  </flexbox-item>
                  <flexbox-item :span="2"><p style="text-align:right;padding:0 .5rem">×{{order.buy_amount}}</p></flexbox-item>
                  <flexbox-item :span="3"><p style="text-align:right;padding:0 .5rem">￥{{order.product_price}}</p></flexbox-item>
                </flexbox>
                <span class="vux-label-desc">{{order.product_specification}}</span>
              </div>
            </div>
          <cell :value="'总计:￥'+OrderDetail.order.total_deal_price"></cell>
        </group>

        <group title="商家信息">
          <cell title="商家名称" :value="OrderDetail.order.shop_name"></cell>
          <cell title="商家地址" :value="OrderDetail.order.shop_address_detail"></cell>
          <cell title="服务地区" :value="OrderDetail.order.service_region"></cell>
          <cell title="服务电话" >
            <a :href="'tel:'+OrderDetail.order.shop_phone" slot="default">{{OrderDetail.order.shop_phone}}</a>
          </cell>
        </group>

        <group title="其他信息">
          <cell title="订单号" :value="OrderDetail.order.order_no"></cell>
          <cell title="订单状态" :value="OrderDetail.order.order_status | order_status"></cell>
          <cell title="下单时间" :value="OrderDetail.order.created_at"></cell>
          <cell title="支付方式" :value="OrderDetail.order.pay_type | pay_type"></cell>
          <cell v-if="OrderDetail.order.send_at" title="交货时间" :value="OrderDetail.order.send_at"></cell>
        </group>
      </div>

      <infinite-loading :on-infinite="_onInfinite"  ref="infiniteLoading" spinner="default">
        <div slot="no-results"><p style="padding:1rem;text-align:center;" @click="_reset">加载失败,请点我重试</p></div>
        <div slot="no-more"></div>
      </infinite-loading>

    </div>

  </view-box>
</template>

<script>
import InfiniteLoading from 'vue-infinite-loading'
import { Group, Cell ,XInput,XButton,XHeader,Tabbar, TabbarItem,Flexbox,FlexboxItem,ViewBox } from 'vux'
import { mapActions,mapGetters } from 'vuex'

export default {
  components: {
    Group,XInput ,XButton,XHeader,Tabbar, TabbarItem,Flexbox,FlexboxItem,Cell,InfiniteLoading,ViewBox
  },
  beforeRouteEnter(to, from, next){
    next(vm => {
      vm.orderDetailClear()
      vm.$refs.infiniteLoading.$emit('$InfiniteLoading:reset')
    })
  },
  filters: {
    order_status(state){
      switch(state){
        case 0 : return '已取消';
        case 1 : return '待付款';
        case 2 : return '待发货';
        case 3 : return '待发货';
        case 4 : return '已取消';
      }
    },
    pay_type(state){
      switch(state){
        case 1 : return '易支付';
        case 2 : return '微信支付';
        case 3 : return '线下支付';
      }
    },
    cancelled_type(state){
      switch(state){
        case 1 : return '买方';
        case 2 : return '卖方';
      }
    }
  },
  computed:{
    ...mapGetters(['OrderDetail'])
  },
  methods:{
    ...mapActions(['getOrderDetail','orderDetailClear']),
    _onInfinite(){
      this.getOrderDetail()
      .then(()=>{
        this.$refs.infiniteLoading.$emit('$InfiniteLoading:loaded');
        this.$refs.infiniteLoading.$emit('$InfiniteLoading:complete');
      })
      .catch((error)=>{
        this.$refs.infiniteLoading.$emit('$InfiniteLoading:complete');
      })
    },
    _reset(e){
      this.$refs.infiniteLoading.$emit('$InfiniteLoading:reset');
    }
  }
}
</script>
<template>
  <div>
    <x-header
      slot="header"
      :left-options="{showBack: true,backText:'返回'}"
      style="width:100%;position:absolute;left:0;top:0;z-index:100;"
      title="修改地址"
    >
    </x-header>
    <div style="padding-top: 46px;background: #fff;overflow: hidden;">
      <x-input title="  姓名" placeholder="请输入收货人的姓名" v-model="name" is-type="china-name"></x-input>
      <x-input title="联系电话" placeholder="请输入收货人的手机号码" v-model="phone" keyboard="number" is-type="china-mobile"></x-input>
      <group>
        <cell title="请选择收货地址" link="/userinfo/amap/" :inline-desc="address"></cell>
      </group>
    </div>
    <x-button
      type="primary"
      style="width:90%;margin-top: 10px;"
      @click.native="submit"
      :disabled="disableSubmit"
    >确认</x-button>
  </div>
</template>

<script>
  import {XButton, XHeader, Group, XInput, Cell, XTextarea} from 'vux'
  import { mapActions,mapGetters } from 'vuex'
  export default {
    components: {
      XButton, XHeader, Group, XInput, Cell, XTextarea
    },
    methods: {
      ...mapActions(['setUserLnglat','setUserAddress','setUserName','setUserPhone','setAddressId','editAddress']),
      submit(){
        const that=this;
//        提交数据
        this.editAddress({
          address:{
          	"receiver_address_id":this.InfoLnglat.address_id,
            "address_detail":this.InfoLnglat.address_detail,
            "latitude":this.InfoLnglat.latitude,
            "longitude":this.InfoLnglat.longitude,
            "receiver_name":this.InfoLnglat.name,
            "receiver_phone":this.InfoLnglat.phone,
          }
        })
          .then((res)=>{
            if(res.data.code != 20000){
              this.$vux.toast.show({
                type:'warn',
                text: res.data.message
              });
              return;
            }
            this.$vux.toast.show({
              text: '修改地址成功',
              time:1000
            });
//    			清空state中的数据
            that.setUserLnglat({
              "latitude":'',
              "longitude":''
            });
            that.setUserName('');
            that.setUserPhone('');
            that.setUserAddress('');
            that.setAddressId('')
            setTimeout(()=>{
              history.back()
            },1000)
          })
      },
      //      检测提交按钮是否可以点击
      checkSubmit(){
        if(
          (this.InfoLnglat.name != '')
          &&
          (this.InfoLnglat.phone != '')
          &&
          (this.InfoLnglat.address_detail != '')
        ){
          this.disableSubmit=false;
        }else {
          this.disableSubmit=true;
        }
      },
      //左边的返回
      
    },
    computed:{
      ...mapGetters(['InfoLnglat']),
    },
    data () {
      return {
        name: '',
        phone: '',
        address: '',
        disableSubmit:false,
      }
    },
    watch: {
      name: function (val) {
        this.setUserName(val);
        this.checkSubmit();
      },
      phone: function (val) {
        this.setUserPhone(val);
        this.checkSubmit();
      },
      address: function () {
        this.checkSubmit();
      }
    },
    mounted(){
    	if(this.InfoLnglat.name == ''){
        this.$router.replace('/userinfo/address_list/')
        return;
      }
      this.address=this.InfoLnglat.address_detail;
      this.name=this.InfoLnglat.name;
      this.phone=this.InfoLnglat.phone;
    }

  }
</script>
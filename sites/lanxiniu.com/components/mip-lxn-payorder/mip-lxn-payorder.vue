<template>
  <div
    id="payorder"
    class="wrapper">
    <div class="pay-head">
      <div class="content">
        <div
          class="time"
          v-text="head.time"/>
        <div class="pay-address">
          <div class="item left">
            <span
              class="title"
              v-text="head.moveOut"/>
            <p v-text="head.moveOutfloor"/>
          </div>
          <div class="item center">
            <p
              class="title"
              v-text="head.carType"/>
            <p class="img arrow"/>
          </div>
          <div class="item right">
            <span
              class="title"
              v-text="head.moveIn"/>
            <p v-text="head.moveInfloor"/>
          </div>
        </div>
      </div>
    </div>
    <div class="pay-content">
      <!-- <div class="head">
        <span class="arrow-down"/>
        <span class="">费用明细</span>
      </div> -->
      <div class="pay-list">
        <ul>
          <li
            v-for="item in pillList"
            :class="{title:item.billName==='合计'}"
            :key="item.billName">
            <span v-text="item.billName"/>
            <span v-text="item.billMount"/>
          </li>
        </ul>
      </div>
      <!-- <div
        :class="{'fontmoney-show':useFontMoney}"
        class="prepay fontmoney">
        <ul>
          <li>
            <span>备注</span>
            <span v-text="frontMoney">40元</span>
          </li>
        </ul>
      </div> -->
      <div class="remark">
        <ul>
          <li>
            <div>
              备注 :
            </div>
            <div>
              <input
                v-model="remark"
                type="text"
                placeholder="请输入备注信息">
            </div>
          </li>
        </ul>
      </div>
      <div class="desc">
        <ul>
          <li>付款须知:</li>
          <li>1、线上预付费用,支付资金由平台监管,未服务100%退款。</li>
          <li>2、待确定的额外费用服务结束后按收费标准支付。</li>
          <li>3、如因额外服务产生费用,请通过在线支付,杜绝乱收费。</li>
        </ul>
      </div>
    </div>
    <div class="footer">
      <div
        :class="{'fontmoney-show':useFontMoney}"
        class=" fontmoney">
        <div class="baoxian">
          <p class="tips"/>
          <p>支付方式</p>
          <p class="yfdj">预付定金</p>
          <p>  预付定金即可预约,服务完成付尾款</p>
        </div>
      </div>

      <p
        class="btn"
        @click="payOrders">
        {{ btnText }}
      </p>
    </div>
  </div>
</template>

<script>
import base from '../../common/utils/base'
import '../../common/utils/base.less'
base.setHtmlRem()
export default {
  props: {
    globaldata: {
      type: Object,
      default: function () { return {} }
    },
    payConfig: {
      type: Object,
      default: function () { return {} }
    },
    userlogin: {
      type: Object,
      default: function () { return {} }
    }
  },
  data () {
    return {
      head: {
        time: '-',
        moveOut: '-',
        moveOutfloor: '-',
        carType: '-',
        moveIn: '-',
        moveInfloor: '-'
      },
      pillList: [
        {
          billName: '-',
          billMount: '-'
        }
      ],
      interval: '',
      frontMoney: '-',
      useFontMoney: true,
      beizhu: '备注',
      remark: '',
      btnText: '确认支付'
    }
  },
  watch: {
    globaldata (val, oldval) {
      console.log('数据改变了+重置数据=======')
    }
  },
  created () {
    console.log('创建数据')

    console.log(this.globaldata)
    // this.setList()
  },
  mounted () {
    window.addEventListener('hide-page', (e) => {
      this.interval && clearInterval(this.interval)
    })
    this.setList()
    console.log('这里是支付页面 !')
    this.clickRipple()
    this.$element.customElement.addEventAction('login', (event, str) => {
      this.interval = setInterval(() => {
        if (this.userlogin.sessionId !== '') {
          this.initData()
          clearInterval(this.interval)
        }
      }, 200)
    })

    if (this.userlogin.sessionId !== '') {
      this.initData()
    }

    // $emit
  },
  methods: {
    //   初始化数据
    initData () {
      let orderNum = base.getRequest().OrderNum
      let sessionid = this.userlogin.sessionId
      let urlsParam = base.setUrlParam({
        orderNum: orderNum,
        sessionId: sessionid
      })
      let urls = base.url + '/Order/detail?' + urlsParam
      fetch(urls, {
        method: 'get'
      })
        .then(response => response.json())
        .catch(error => console.error('Error:', error))
        .then(response => {
          let data = response.data
          console.log(data)
          let poiList = data.poiList
          // 列表
          let billinfos = data.billinfo
          let floorData = [
            '有电梯',
            '无电梯1楼',
            '无电梯2楼',
            '无电梯3楼',
            '无电梯4楼',
            '无电梯5楼',
            '无电梯6楼',
            '无电梯7楼',
            '无电梯8楼'
          ]
          // 头部
          this.head = {
            time: data.TransTime,
            moveOut: poiList[0].deliverAddress,
            moveIn: poiList[1].deliverAddress,
            carType: data.CarTypeName,
            moveOutfloor: floorData[data.start_stairs_num],
            moveInfloor: floorData[data.end_stairs_num]
          }

          billinfos.push({
            billName: '合计',
            billMount: data.billTotal
          })
          this.pillList = billinfos
          if (data.frontMoney) {
            let money = data.frontMoney
            if (+money !== 0) {
              this.useFontMoney = true
              this.frontMoney = money + '元'
            }
          }
        })
    },

    // 请求订单数据
    setList () {
      let _this = this
      let data = base.getSession()
      console.log('查看订单信息============')
      console.log(data)
      if (data !== null) {
        if (data.order && data.order !== '') {
          let order = data.order
          let poiList = order.poiList
          let floorData = [
            '有电梯',
            '无电梯1楼',
            '无电梯2楼',
            '无电梯3楼',
            '无电梯4楼',
            '无电梯5楼',
            '无电梯6楼',
            '无电梯7楼',
            '无电梯8楼'
          ]
          // 头部
          _this.head = {
            time: order.TransTime,
            moveOut: poiList[0].deliverAddress,
            moveIn: poiList[1].deliverAddress,
            carType: order.CarTypeName,
            moveOutfloor: floorData[data.moveOutNum],
            moveInfloor: floorData[data.moveInNum]
          }
          // 列表
          let billinfos = order.billinfo
          billinfos.push({
            billName: '合计',
            billMount: order.billTotal
          })
          _this.pillList = billinfos

          if (order.frontMoney) {
            let money = order.frontMoney
            if (+money !== 0) {
              this.useFontMoney = true
              this.frontMoney = money + '元'
              this.btnText = '确认支付订金' + this.frontMoney
            }
          }
        }
      }
    },
    // 点击波纹效果
    clickRipple () {
      let util = MIP.util
      util.event.delegate(
        this.$element,
        '.btn',
        'click',
        function (e) {
          let target = e.target
          console.log(target)
          if (target.className.indexOf('btn') > -1) {
            let rect = target.getBoundingClientRect()
            let ripple = target.querySelector('.ripple')
            if (!ripple) {
              ripple = document.createElement('span')
              ripple.className = 'ripple'
              ripple.style.height = ripple.style.width =
                Math.max(rect.width, rect.height) + 'px'
              target.appendChild(ripple)
            }
            ripple.classList.remove('show')
            let top =
              e.pageY -
              rect.top -
              ripple.offsetHeight / 2 -
              document.body.scrollTop
            let left =
              e.pageX -
              rect.left -
              ripple.offsetWidth / 2 -
              document.body.scrollLeft
            ripple.style.top = top + 'px'
            ripple.style.left = left + 'px'
            ripple.classList.add('show')
            return false
          }
        }
      )
    },
    //   支付
    payOrders () {
      let lxndata = base.getSession()
      let orderNum = lxndata.order.OrderNum
      let price = ''
      let billTotal = lxndata.order.billTotal
      if (lxndata.order.frontMoney) {
        let money = lxndata.order.frontMoney
        if (+money !== 0) {
          price = money + '元'
        } else {
          price = billTotal
        }
      } else {
        price = billTotal
      }
      //   let price = lxndata.order.billTotal
      //   let price = lxndata.order.frontMoney
      //   let sessionid = base.getbaiduLogMsg()
      let sessionid = this.userlogin.sessionId
      console.log('token:' + sessionid + '======' + 'orderNum:' + orderNum)
      let urlsParam = base.setUrlParam({
        orderNum: orderNum,
        sessionId: sessionid,
        remark: this.remark,
        total_fee: price,
        ver: 2.0
      })
      let obj = {
        sessionId: sessionid,
        redirectUrl: 'https://www.lanxiniu.com/Pay/success?' + urlsParam,
        fee: price,
        postData: {
          orderNum: orderNum,
          token: sessionid,
          remark: this.remark,
          ver: 2.0
        }
      }
      MIP.setData({
        payConfig: MIP.util.fn.extend({}, this.payConfig, obj)
      })
      this.$emit('actionPay', {})
    }
  }
}
</script>

<style scoped lang="less">
.pay-head .content {
  background: #36a0e9;
  box-shadow: 0 2px 4px 0 rgba(0, 0, 0, 0.2);
  text-align: center;
  color: #ffffff;
  padding: 0.32rem 0.3975rem;
}
.pay-address {
  margin-top: 0.1rem;
  display: flex;
}
.pay-address > div.item {
  flex: 1;
  height: 100%;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
.pay-address > div span.title {
  font-size: 0.28rem;
}
.pay-address > div p:last-child {
  font-size: 0.24rem;
  margin-top: 0.05rem;
}
.pay-address .left {
  text-align: left;
}
.pay-address .center {
  align-items: center;
  padding-top: 0.1rem;
  font-size: 0.24rem;
}
.pay-address .right {
  text-align: right;
}
.pay-content {
  padding: 0 0.2rem;
  margin-top: 0.2rem;
}

.pay-list {
  padding: 0 0.3975rem;
  background: #ffffff;
  box-shadow: 0 2px 2px 0 rgba(0, 0, 0, 0.1);
  border-radius: 2px;
}
.pay-list li {
  font-size: 0.28rem;
  color: #555555;
  letter-spacing: 0.06px;
  padding: 0.24rem 0;
  border-bottom: 0.02rem solid #f1f1f1;
}
.pay-list li span:last-child {
  float: right;
}
.pay-list li.title span:first-child{
   font-weight: 700;
}
.pay-list li.title span:last-child {
  font-size: 0.32rem;
  color: #ff6666;
  letter-spacing: 0.07px;
}
.pay-list li:last-child{
    border-bottom: none;
}
/* .prepay {
  margin-top: .2rem;
  padding: 0 0.3975rem;
  background: #ffffff;
  box-shadow: 0 2px 2px 0 rgba(0, 0, 0, 0.1);
  border-radius: 2px;
}
.prepay li {
  font-size: .28rem;
  color: #555555;
  letter-spacing: 0.06px;
  padding: 0.24rem 0;
}
.prepay li span:first-child{
    font-weight: 700;
}
.prepay li span:last-child {
  float: right;
  font-size: .32rem;
  color: #ff6666;
  letter-spacing: .07px;
} */
.remark{
  margin-top: .2rem;
  padding: 0 0.3975rem;
  background: #ffffff;
  box-shadow: 0 2px 2px 0 rgba(0, 0, 0, 0.1);
  border-radius: 2px;
}
.remark li{
  color: #555555;
  letter-spacing: 0.06px;
  height: .8rem;
  display: flex;
  align-items: center;
}

.remark li div:last-child{
    flex: 1;
    height: 100%;
    padding-left: .1rem;
    input{
        height: 100%;
    }
}

.desc {
  margin-top: 0.3rem;
  padding: 0 0.1975rem;
  font-size: 0.2rem;
  color: #999999;
  letter-spacing: 0.04px;
}
.desc li {
  margin-top: 0.1rem;
}
.desc li span {
  color: #36a0e9;
}
.footer {
  margin: 0.4rem 0;
  padding: 0 0.2rem;
}
.footer .baoxian {
  background: #fcf0ae;
  display: flex;
  font-size: 0.2rem;
  color: #666666;
  align-items: center;
  padding: .05rem 0;
  padding-left: .2rem;
}
.footer .tips{
    margin-right: .1rem;
    display: inline-block;
    width: .26rem;
    height: .26rem;
    background-size: 100% 100%;
    background-image: url(https://www.lanxiniu.com/Public/baidumip/tip.png)
}

.yfdj{
    line-height: unset;
    padding: .03rem .1rem;
    background: #FFD552;
    border-radius: 3px;
    margin-right: .1rem;
}
.footer .safe {
  position: relative;
  top: 0.05rem;
  right: 0.05rem;
}
.footer p.btn {
  margin-top: 0.1rem;
  height: 0.76rem;
  line-height: 0.76rem;
  background: #36a0e9;
  box-shadow: 0 0.02rem 0.04rem 0 rgba(0, 0, 0, 0.3);
  border-radius: 0.02rem;
  color: #ffffff;
  text-align: center;
}

.fontmoney{
    display: none;
}
.fontmoney-show{
    display: block;
}
</style>

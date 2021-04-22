<!-- home -->
<template>
  <div class="index-container">
    <div class="go-to-miniprograme-button">
        <wx-open-launch-weapp
                id="launch-btn"
                username="gh_f2989b6a66d5"
                path="/pages/home/home.html">
            <script type="text/wxtag-template">
                <style>
                    .btn {
                      width:140px;
                      display:flex;
                      height: 140px;
                      z-index: 1000;
                      background-color: #FE9B14;
                      color: white;
                      align-items: center;
                      justify-content: center;
                      font-size: 20px;
                      border-radius: 50%;
                      font-weight: bold;
                      box-shadow: 0 0 20px rgba(0,0,0,0.15);
                      }
                      .btn-box{
                        padding: 30px;
                      }
                </style>
                <div class="btn-box">
                  <button class="btn">打开<br/>小程序</button>
                </div>
            </script>
        </wx-open-launch-weapp>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      appid: 'wx6b91e0d714723725'
    }
  },

  computed: {},

  mounted() {
    document.title = '我是保管家'
    const code = this.$route.query.code
    if (code) {
      this.getOpenIdByCode(code)
    } else {
      this.loginHandler()
    }
  },

  methods: {
	  sha1Sign(ticket, nstr , time, url){
	    console.log('签名参数', ticket, nstr , time, url)
	    return sha1(
	      'jsapi_ticket='+ ticket +'&' +
	      'noncestr='+nstr+'&' +
	      'timestamp='+time+'&' +
	      'url=' + url
	    )

	  },
	  createRandomStr(num) {
	    let e = num || 32,
	      t = "ABCDEFGHJKMNPQRSTWXYZabcdefhijkmnprstwxyz2345678",
	      a = t.length,
	      n = "";
	    for (let i = 0; i < e; i++) n += t.charAt(Math.floor(Math.random() * a));
	    return n
	  },
    loginHandler() {
      const redirect = 'https%3A%2F%2Fbaoguanjia.ltd%2Fhome'
      // let redirect = 'https%3A%2F%2Flocalhost:9020'
      const appid = this.appid
      window.location.href = `https://open.weixin.qq.com/connect/oauth2/authorize?appid=${appid}&redirect_uri=${redirect}&response_type=code&scope=snsapi_userinfo&state=wsy#wechat_redirect`
    },
    getOpenIdByCode(code) {
      const queryData = {
        code: code
      }
      // services.requestGetOpenId(queryData, {}).then((res) => {
      //   //  唤起小程序
      //   this.goToMiniPrograme()
      // })
    },
    goToMiniPrograme() {
      services.requestGetAccessForJsTicket().then((res) => {
        const returnJson = JSON.parse(res.data.data)

        const params = {
          accessToken: returnJson.access_token
        }
        services.requestGetJsTicket(params, {}).then((result) => {
          const returnChildJSON = JSON.parse(result.data.data)
          const time = new Date().getTime()
          const nonceStr = this.createRandomStr(16)
          const ticket = returnChildJSON.ticket
          const url = window.location.href
          // config信息验证后会执行ready方法，所有接口调用都必须在config接口获得结果之后，config是一个客户端的异步操作，所以如果需要在页面加载时就调用相关接口，则须把相关接口放在ready函数中调用来确保正确执行。对于用户触发时才调用的接口，则可以直接调用，不需要放在ready函数中
          window.wx.config({
            debug: false, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印
            appId: this.appid, // 必填，公众号的唯一标识
            timestamp: time, // 必填，生成签名的时间戳
            nonceStr: nonceStr, // 必填，生成签名的随机串
            signature: this.sha1Sign(ticket, nonceStr, time, url), // 必填，签名
            jsApiList: ['onMenuShareTimeline', 'checkJsApi', 'hideAllNonBaseMenuItem', 'showAllNonBaseMenuItem'], // 必填，需要使用的JS接口列表
            openTagList: ['wx-open-launch-app', 'wx-open-launch-weapp'] // 可选，需要使用的开放标签列表，例如['wx-open-launch-app']
          })

          window.wx.ready(function() {
            // config信息验证后会执行ready方法，所有接口调用都必须在config接口获得结果之后，config是一个客户端的异步操作，所以如果需要在页面加载时就调用相关接口，则须把相关接口放在ready函数中调用来确保正确执行。对于用户触发时才调用的接口，则可以直接调用，不需要放在ready函数中
          })

          window.wx.error(function(res) {
            // config信息验证失败会执行error函数，如签名过期导致验证失败，具体错误信息可以打开config的debug模式查看，也可以在返回的res参数中查看，对于SPA可以在这里更新签名
            console.log('异常信息： ', res)
          })

          var btn = document.getElementById('launch-btn')
          btn.addEventListener('launch', function(e) {
            console.log('success')
          })
          btn.addEventListener('error', function(e) {
            console.log('fail', e.detail)
          })
        })
      })
    }
  }
}
</script>
<style lang="scss" scoped>
    .index-container{
        width: 100%;
        height: 100vh;
        background-size: 100% 100%;
        position: relative;
    }
    .go-to-miniprograme-button{
        width: 100%;
        text-align: center;
        position: absolute;
        left: 0;
        bottom: 160px;
    }
</style>

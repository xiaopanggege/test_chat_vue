<template>
  <div class="app-container" >
    <JwChat-index :config="config" :showRightBox='true' scrollType="scroll"
:taleList="taleList" @enter="bindEnter" @clickTalk="talkEvent" 
 v-model="inputMsg" :toolConfig="tool" >
    <JwChat-rightbox class="rightSlot" :config="rightConfig" @click="rightClick" />
    
    </JwChat-index>
        

       
  </div>
</template>

<script>
// import { getSaltKeyList, flushSaltKeyList, acceptSaltKey, deleteSaltKey, rejectSaltKey, deleteDeniedSaltKey, testSaltKey } from '@/api/saltstack'
import waves from '@/directive/waves' // Waves directive
import { parseTime } from '@/utils'
import Pagination from '@/components/Pagination' // Secondary package based on el-pagination
import { Message } from 'element-ui'
import { mapGetters } from 'vuex'


export default {
  name: 'Chat',
  components: { 
    Pagination,
  },
  computed: {
    ...mapGetters([
      'avatar',
      'name',
      'token'
    ])
  },
  data() {
    return {
      inputMsg: '',
      taleList: [],
      tool: {
        // show: ['file', 'history', 'img', ['文件1', '', '美图']],
        // showEmoji: false,
        callback: this.toolEvent
      },
      config: {
        img: require('@/assets/sidebar-logo/sidebar-log.png'),
        name: '小胖运维聊天',
        dept: '是朋友就一起来PY',
        callback: this.bindCover,
        historyConfig:{
          show: true,
          tip: '加载更多',
          callback: this.bindLoadHistory,
        },
        quickList: [
          {text: '这里是jwchat，您想了解什么问题。'},
          {text: 'jwchat是最好的聊天组件'},
          {text: '谁将烟焚散，散了纵横的牵绊；听弦断，断那三千痴缠。'},
          {text: '长夏逝去。山野间的初秋悄然涉足。'},
          {text: '江南风骨，天水成碧，天教心愿与身违。'},
          {text: '总在不经意的年生。回首彼岸。纵然发现光景绵长。'},
          {text: '外面的烟花奋力的燃着，屋里的人激情的说着情话'},
          {text: '假如你是云，我就是雨，一生相伴，风风雨雨；'},
          {text: '即使泪水在眼中打转，我依旧可以笑的很美，这是你学不来的坚强。'},
          {text: ' 因为不知来生来世会不会遇到你，所以今生今世我会加倍爱你。'},
        ]
      },
      rightConfig: {
        listTip: '当前在线',
        notice: '【公告】这是一款高度自由的聊天组件，基于AVue、Vue、Element-ui开发。点个赞再走吧 ',
        list: [
        ]
      },

      wpath: "ws://192.168.1.114:8000/ws/chat/xiaopgg/",   // 我固定了发送的IP地址(可以换域名的)和聊天房间所以每个人登录都会进入这个房间，因为只是玩一下，真要做肯定要动态
      ws: null,


      
    }
  },
  created() {
    this.initWebSocket();
    
  },
  
  beforeDestroy() {
      this.websocketclose() //离开路由之后断开websocket连接
  },

  methods: {

    initWebSocket(){
      // 实例化socket，这里的实例化直接赋值给this.ws是为了后面可以在其它的函数中也能调用websocket方法，例如：this.ws.close(); 完成通信后关闭WebSocket连接
      this.ws = new WebSocket(this.wpath + `?${this.token}`)  //在ws的链接最后用?xxxx来加的内容会被后端scope['query_string']捕获用来做用户认证的因为jwt的认证无法被channels识别
      // 定义接收数据后的操作
      this.ws.onmessage = this.websocketonmessage;
      // 定义建立连接后的操作
      this.ws.onopen = this.websocketonopen;
      // 定义连接建立失败后的操作
      this.ws.onerror = this.websocketonerror;
      // 定义关闭连接操作
      this.ws.onclose = this.websocketclose;

      // //监听是否连接成功
      // this.ws.onopen = ()=> {
      //   console.log('ws连接状态：' +this.ws.readyState);
      //   //连接成功则发送一个数据
      // this.ws.send('连接成功');
      // }
      // //接听服务器发回的信息并处理展示
      // this.ws.onmessage = (data)=> {
      //     console.log('接收到来自服务器的消息：');
      //     console.log(data)       
      // }
      
      // //监听连接关闭事件
      // this.ws.onclose = ()=>{
      //     //监听整个过程中websocket的状态
      //     console.log('ws连接状态：' + this.ws.readyState);
      // }
      
      // //监听并处理error事件
      // this.ws.onerror = function(error) {
      //     console.log(error);
      // }
    },
    websocketonopen(){ //连接建立之后执行send方法发送数据，然后后端会判断message内容返回房间用户字典回来
      let actions = {"code": 200, "message":"connect"};
      this.websocketsend(actions);
    },
    websocketonerror(){//连接建立失败重连
      setTimeout(function () {
          this.initWebSocket;  // 还有问题要测试一下
      }, 300);
      
    },
    websocketonmessage(e){ //数据接收
      const redata = JSON.parse(e.data);
      const message = redata['message']
      // 这里没有做异常判断，有可能message不是一个字典哈
      // 如果返回码是666说明返回的是聊天内容
      if(message['code'] === '666'){
        const data = message['message']
        if(data['name']===this.name){
          data['mine'] = true  // 如果接收到的数据里用户名等于当前页面用户名则把mine改成true，即显示聊天页面右边
        }
        // console.log(data);
        this.taleList.push(data)
      }
      // 如果返回码是200，则说明是返回的房间信息
      else if(message['code'] === '200'){
        const data_dict = message['message']
        this.rightConfig.list = []
        for(var key in data_dict){
          const data = {}
          data['name'] = data_dict[key].username
          // 右边栏头像获取，我写死的方法，实际环境看情况再调整吧，这里不纠结了，能用就行啊哈
          data['img'] = process.env.VUE_APP_BASE_API + '/' + data_dict[key].avatar
          this.rightConfig.list.push(data)
        }
        
      }
      else{
        return false
      }
      
    },
    websocketsend(seData){//数据发送
      this.ws.send(JSON.stringify(seData));
    },
    websocketclose(){  //关闭
      console.log('断开连接');
      setTimeout(() => {
          this.ws.close()
        }, 0.5 * 1000)
      
    },


    bindEnter () {
      const msg = this.inputMsg
      if (!msg) return;
      const msgObj = {
        "date":  parseTime(new Date(),'{y}-{m}-{d} {h}:{i}'),
        "text": { "text": msg },
        "mine": false,
        "name": this.name,
        "img": this.avatar
      }
      const seData = {"code": 666, "message":msgObj}
      this.websocketsend(seData)
      // this.taleList.push(msgObj)
    },

     /**
     * @description: 
     * @param {*} type 当前点击的按钮
     * @param {*} plyload 附加文件或者需要处理的数据
     * @return {*}
     */
    toolEvent (type, plyload) {
      console.log('tools', type, plyload)
    },
     /**
     * @description: 点击加载更多的回调函数
     * @param {*}
     * @return {*}
     */
    bindLoadHistory () {
      const history = new Array(3).fill().map((i, j) => {
        return {
          "date": "2020/05/20 23:19:07",
          "text": { "text": j + new Date() },
          "mine": false,
          "name": "JwChat",
          "img": "image/three.jpeg"
        }
      })
      let list = history.concat(this.list)
      this.list = list
      console.log('加载历史', list, history)
    },
    bindCover (type) {
      console.log('header', type)
    },
    bindWinBar (play = {}) {
      const {type, data={}} = play
      console.log(data);
      if(type==='winBar'){
        const { id, dept, name, img } = data
        this.winBarConfig.active = id
      }
    },

    talkEvent(){
      return true
    },

    rightClick (type) {
      console.log('rigth', type)
    },
   
  }
}
</script>


<style rel="stylesheet/scss" lang="scss" scoped>
.app-container{
  .rightSlot {
    width: 100%;
    height: 100%;
    overflow: hidden;
    flex-direction: column;
  }
}


</style>
<template>
  <div class="dashboard-editor-container">
    <!-- <github-corner class="github-corner" /> -->

    <!-- <panel-group @handleSetLineChartData="handleSetLineChartData" />

    <el-row style="background:#fff;padding:16px 16px 0;margin-bottom:32px;">
      <line-chart :chart-data="lineChartData" />
    </el-row> -->
    聊天系统
  </div>
</template>

<script>
import MinionStatusChart from './components/MinionStatusChart'
import MinionInfoChart from './components/MinionInfoChart'
import ReleaseCountChart from './components/ReleaseCountChart'
import TaskLogTable from './components/TaskLogTable'
import ReleaseLogTable from './components/ReleaseLogTable'
import ServerTable from './components/ServerTable'
import { dashboard } from '@/api/dashboard'



export default {
  name: 'DashboardAdmin',
  components: {
    MinionStatusChart,
    MinionInfoChart,
    ReleaseCountChart,
    TaskLogTable,
    ReleaseLogTable,
    ServerTable,
    // TodoList,
    // BoxCard
  },
  watch: {
    autoUpdate(value) {
      if(!value){
        clearInterval(this.myInterval)
      }else{
        clearInterval(this.myInterval)
        this.myInterval = setInterval(()=>{
          // 听说这么写轮询，不会卡死
          setTimeout(this.fetchData(),0)
        }, 60000)
      }
    }
  },
  created() {
    // this.fetchData()
  },
  data() {
    return {
      autoUpdate: false,
      list: null,
      now_time: "",
      old_list: null,
      myInterval: '',
      task_log: null,
      release_log: null,
      saltkey_certification_count: null,
      minion_status_count: null,
      release_log_count: null,
      sys_status: null,
    }
  },
  methods: {
    fetchData() {
      let data = { opt_cmd: "dashboard_search" }
      dashboard(data).then(response => {
        // 切割获取前6条数据
          let new_list = response.data.results
          if(this.old_list != new_list){
             this.release_log = response.data.results.release_log
             this.task_log = response.data.results.task_log
             this.saltkey_certification_count = response.data.results.saltkey_certification_count
             this.minion_status_count = response.data.results.minion_status_count
             this.release_log_count = response.data.results.release_log_count
             this.sys_status = response.data.results.sys_status
          }
          this.now_time = new Date()
      })
    }
  }
}
</script>

<style lang="scss" scoped>
.dashboard-editor-container {
  padding: 32px;
  background-color: rgb(240, 242, 245);
  position: relative;

  .github-corner {
    position: absolute;
    top: 0px;
    border: 0;
    right: 0;
  }

  .chart-wrapper {
    background: #fff;
    padding: 16px 16px 0;
    margin-bottom: 32px;
  }
}

@media (max-width:1024px) {
  .chart-wrapper {
    padding: 8px;
  }
}
</style>

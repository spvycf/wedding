<template>
  <div class="executions">
    <!--执行命令-->
    <p class="code" v-for="(execution, index) in executions" :key="index" v-show="execution.visible">
      <span class="addon"></span>
      [<span class="time">{{ execution.time }}</span>]
      <br/>
      <span class="task white-space">{{ execution.name }}</span>
      <span class="durationUnit" >{{ execution.duration }} {{ execution.unit }}</span>
    </p>
    <!--进度条-->
    <p class="code" v-show="isProcessed">
      <span class="addon"></span>
      {{ progressBarText }}
      <span class="percentage">{{ percentage }}%</span>
    </p>
    <!--执行命令-->
    <p class="code" v-show="endExecution.visible">
      <span class="addon"></span>
      [<span class="time">{{ endExecution.time }}</span>]
      <span class="task">{{ endExecution.name }}</span>
    </p>
  </div>
</template>

<script>
  import data from '../mock/data'
  export default {
    name: 'Executions',
    props: ['canExecute'],
    data () {
      return {
        executions: data.executions,
        isProcessed: false,
        isFinished: false,
        progressBarText: '---------------------------------',
        endExecution: {
          name:'请柬打开中...',
          time: '',
          visible: false
        }
      }
    },
    computed: {
      percentage () {
        const hashL = this.progressBarText.split('').filter(c => c==='#').length
        const l = this.progressBarText.length
        return Math.floor(hashL*100/l)
      }
    },
    watch: {
      canExecute: function (val) {
        if (val===true) {
          this.runExecutions()
        }
      }
    },
    updated() {
      this.$emit('onUpdating')
    },
    methods: {
      // 逐条执行命令
      runExecutions: async function () {
        for (const execution of this.executions) {
          await this.progressivelyRun(execution)
        }
        // 执行完命令，开始显示进度条
        await this.successProcessing(2000)
        // 执行最后一条命令
        await this.progressivelyRun(this.endExecution)
          .then(() => {
            setTimeout(() => {
              this.isFinished = true
              this.$emit('onFinish')
            }, 1500)
          })
      },
      // 执行一条命令
      progressivelyRun(execution, customDuration) {
        return new Promise((resolve) => {
          let now = new Date(),
              duration = customDuration ? customDuration : Math.random()*20+200,
              showCode = () => {
                execution.time = now.toLocaleTimeString('chinese', { hour12: false })
                execution.duration = execution.duration == 0 ? duration.toFixed(2) : execution.duration
                execution.visible = true
                resolve()
              }
          setTimeout(showCode, 1500)
          //2s延迟

        })
      },
      // 显示进度条
      successProcessing(duration){
        return new Promise((resolve) => {
          let start = null, progressing, progressingCount = 0
          this.isProcessed = true
          let step = (timestamp) => {
              let timeGap
              if (start === null) {
                start = timestamp
              }
              timeGap = timestamp - start
              if (progressingCount%3 === 0) {
                this.progressBarText = this.progressBarText.replace('-', '#')
              }
              progressingCount++
              if (timeGap < duration) {
                progressing = requestAnimationFrame(step)
              } else {
                resolve()
                this.progressBarText = this.progressBarText.replace(/-/g, '#')
                cancelAnimationFrame(progressing)
              }
            }
          progressing = requestAnimationFrame(step)
        });
      }
    }
  }
</script>

<style lang="less">
  .executions{
    .addon{
      color: #68FCFB;
    }
    .time{ color: #666; }
    .task{ color: #009AB2; }
    .duration{
      margin-left: 10px;
      color: #BF36B7;
    }
    .durationUnit{
      margin-right: 10px;
      float: right;
      color: #BF36B7;
    }
    .white-space {
      white-space:pre
    }
  }
</style>
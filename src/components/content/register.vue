<template>
  <div>
  <el-button :plain="true" @click="open2">成功</el-button>
  <el-button :plain="true" @click="open3">警告</el-button>
  <el-button :plain="true" @click="open">消息</el-button>
  <el-button :plain="true" @click="open4">错误</el-button>
  </div>
</template>

<script>
  export default {
    methods: {
      open () {
        this.$notify.success({
          title: '成功',
          message: '这是一条成功的提示消息',
          offset: 100
        })
      },
      open2 () {
        this.$message({
          message: '恭喜你，这是一条成功消息',
          type: 'success'
        })
      },

      open3 () {
        this.$message({
          message: '警告哦，这是一条警告消息',
          type: 'warning'
        })
      },

      open4 () {
        this.$message.error('错了哦，这是一条错误消息')
      }
    }
  }
</script>

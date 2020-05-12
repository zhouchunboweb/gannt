<template>
  <div class="app-container">
    <div ref="gantt" class="left-container"/>
    <input value="Export to PDF" type="button" onclick='gantt.exportToPDF()'/>
    <input value="Export to Excel" type="button" onclick='gantt.exportToExcel()'/>
    <input value="Export to xml" type="button" onclick='gantt.exportToMSProject()'/>
    <input value="show Tasks" type="button" @click="cons"/>
  </div>
</template>
<script>
import gantt from 'dhtmlx-gantt' // 引入模块
import 'dhtmlx-gantt/codebase/sources/ganttApi' // 导出调用接口
import 'dhtmlx-gantt/codebase/dhtmlxgantt.css'
import 'dhtmlx-gantt/codebase/sources/locale/locale_cn' // 中文设置
export default {
  name: 'DhtmlxGantt',
  data () {
    return {
      tasks: {
        data: [
          { id: '1', text: '建设电厂施工', type: 'project', open: true },
          { id: '2', text: '工程签约', start_date: '15-04-2020', personName: '林志款', duration: 1, progress: 1, parent: '1' },
          { id: '3', text: '施工', start_date: '18-04-2020', personName: '林志款', duration: 60, progress: 0.4, parent: '1' },
          { id: '4', text: '挖掘机开工', start_date: '20-04-2020', personName: '林志款', duration: 10, progress: 0.4, parent: '3' },
          { id: '5', text: '里程碑', start_date: '01-05-2020', personName: '林志款', duration: 0, type: 'milestone', parent: '1' }
        ],
        links: [
          { id: '1', source: '2', target: '3', type: '0' }
        ]
      }
    }
  },
  methods: {
    cons () {
      console.log(this.tasks)
    },
    dateToString (val) {
      let day = val.getDate()
      let month = val.getMonth() + 1
      let year = val.getFullYear()
      if (day < 10) {
        day = '0' + day
      }
      if (month < 10) {
        month = '0' + month
      }
      return day + '-' + month + '-' + year
    }
  },
  mounted () {
    // 初始化
    gantt.config.autofit = true // 表格列宽自适应
    gantt.config.date_scale = '%M %d日' // 设置右侧标题栏
    gantt.config.grid_width = 450 // 左侧宽度
    gantt.config.columns = [
      {
        name: 'wbs',
        label: '序号',
        width: 40,
        template: gantt.getWBSCode,
        align: 'width'
      },
      {name: 'text', label: '任务名', tree: true, width: '*'}, // tree设置该列为树形控件
      {name: 'start_date', label: '开始时间', align: 'center'},
      {name: 'duration', label: '持续时间', align: 'center'},
      {name: 'add', label: ''} // add效果是成为'+'
    ] // 左侧列设置
    gantt.templates.timeline_cell_class = function (item, date) {
      if (date.getDay() === 0 || date.getDay() === 6) {
        return 'weekend'
      }
    } // 设置周末不同颜色
    // gantt.config.order_branch = true
    // gantt.config.order_branch_free = true // 实现任务拖拽功能 使用需要增加拖拽监听数据
    gantt.templates.tooltip_text = function (start, end, task) {
      return '<b>任务名:</b> ' + task.text + '<br/><b>开始时间:</b>' + gantt.templates.tooltip_date_format(start) + '<br/><b>结束时间:</b>' + gantt.templates.tooltip_date_format(end)
    } // tooltip内容设置
    gantt.plugins({
      tooltip: true
    }) // 启用tooltip
    gantt.config.lightbox.sections = [ // 新增盒子属性
      {name: 'description', height: 70, map_to: 'text', type: 'textarea'},
      {name: 'personName', height: 70, map_to: 'personName', type: 'textarea'},
      {name: 'type', type: 'typeselect', map_to: 'type'},
      {name: 'time', height: 72, type: 'duration', map_to: 'auto'}
    ]
    gantt.templates.rightside_text = function (start, end, task) {
      if (task.type === gantt.config.types.milestone) {
        return task.text
      }
      return ''
    }
    // gantt.config.readonly = true // 只读模式
    gantt.init(this.$refs.gantt)
    // 数据解析
    gantt.parse(this.$data.tasks)
    let that = this
    gantt.attachEvent('onAfterTaskAdd', (id, item) => { // 添加监听
      that.tasks.data.push({
        id: item.id,
        text: item.text,
        start_date: that.dateToString(item.start_date),
        personName: item.personName,
        duration: item.duration,
        progress: item.progress,
        parent: item.parent,
        type: item.type
      })
      gantt.render()
    })
    gantt.attachEvent('onAfterTaskUpdate', (id, item) => { // 修改监听
      that.tasks.data = that.tasks.data.map(obj => {
        if (obj.id === id) {
          obj.text = item.text
          obj.start_date = that.dateToString(item.start_date)
          obj.personName = item.personName
          obj.duration = item.duration
          obj.progress = item.progress
          obj.parent = item.parent
        }
        return obj
      })
      gantt.render()
    })
    gantt.attachEvent('onAfterTaskDelete', (id, item) => { // 删除监听
      that.tasks.data = that.tasks.data.filter(obj => {
        if (obj.id !== id) {
          return obj
        }
      })
      gantt.render()
    })
    gantt.attachEvent('onAfterLinkAdd', (id, item) => { // 连线添加监听
      that.tasks.links.push(item)
    })
    gantt.attachEvent('onAfterLinkUpdate', (id, item) => { // 连线修改监听
      that.tasks.links = that.tasks.links.map(obj => {
        if (obj.id === id) {
          obj.source = item.source
          obj.target = item.target
          obj.type = item.type
        }
      })
    })
    gantt.attachEvent('onAfterLinkDelete', (id, item) => { // 连线删除监听
      that.tasks.links = that.tasks.links.filter(obj => {
        if (obj.id !== id) {
          return obj
        }
      })
    })
  }
}
</script>
<style>
.left-container {
  height: 500px;
}
.weekend {
  background: #f4f7f4;
}
.gantt_selected .weekend {
  background: #f7eb91;
}
</style>

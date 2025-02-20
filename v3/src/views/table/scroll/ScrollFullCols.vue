<template>
  <div>
    <p class="tip">
      虚拟滚动渲染，左右固定列<br>
      大数据不建议使用双向绑定的 <table-api-link name="data"/> 属性（vue 监听会大数据会短暂的卡顿），建议使用 <table-api-link prop="loadData"/>/<table-api-link prop="loadColumn"/> 函数<br>
      对于多选 type=<table-column-api-link prop="checkbox"/> 当数据量海量时应该绑定 <table-api-link prop="checkField"/> 属性渲染速度更快<br>
      <span class="green">（性能优化：横向虚拟滚动由列宽决定性能，每一列的列宽越大就越流畅；纵向虚拟滚动由行高决定性能，每一行的高度越高就越流畅）</span>
    </p>

    <vxe-grid
      border
      resizable
      show-overflow
      show-header-overflow
      ref="xGrid"
      height="600"
      :row-config="{isCurrent: true, isHover: true}"
      :loading="loading"
      :toolbar-config="{slots: {buttons: 'toolbar_buttons'}}"
      :checkbox-config="{checkField: 'checked', labelField: 'name'}"
      :scroll-y="{enabled: true}">
      <template #toolbar_buttons>
        <vxe-button @click="loadColumnAndData(50, 20)">50列20条</vxe-button>
        <vxe-button @click="loadColumnAndData(100, 50)">100列50条</vxe-button>
        <vxe-button @click="loadColumnAndData(200, 100)">200列100条</vxe-button>
        <vxe-button @click="loadColumnAndData(5000, 10000)">5k列1w条</vxe-button>
        <vxe-button @click="loadColumnAndData(10000, 50000)">1w列5w条</vxe-button>
        <vxe-button @click="loadColumnAndData(10000, 100000)">1w列10w条</vxe-button>
        <vxe-button @click="$refs.xGrid.setAllCheckboxRow(true)">所有选中</vxe-button>
        <vxe-button @click="$refs.xGrid.clearCheckboxRow()">清除选中</vxe-button>
        <vxe-button @click="getSelectEvent">获取选中</vxe-button>
      </template>
    </vxe-grid>

    <pre>
      <pre-code>
        | Arrow Up ↑ | 匀速向上滚动数据 |
        | Arrow Down ↓ | 匀速向下滚动数据 |
        | Arrow Left ← | 匀速向左滚动数据 |
        | Arrow Right → | 匀速向右滚动数据 |
        | Page Up | 向上翻页滚动 |
        | Page Down | 向下翻页滚动 |
        | Spacebar | 翻页滚动 |
        | Home | 滚动到顶部 |
        | End | 滚动到底部 |
      </pre-code>
    </pre>

    <p class="demo-code">{{ $t('app.body.button.showCode') }}</p>

    <pre>
      <pre-code class="xml">{{ demoCodes[0] }}</pre-code>
      <pre-code class="javascript">{{ demoCodes[1] }}</pre-code>
    </pre>
  </div>
</template>

<script>
import VXETable from 'vxe-table'

export default {
  data () {
    return {
      loading: false,
      demoCodes: [
        `
        <vxe-grid
          border
          resizable
          show-overflow
          show-header-overflow
          ref="xGrid"
          height="600"
          :row-config="{isCurrent: true, isHover: true}"
          :loading="loading"
          :toolbar-config="{slots: {buttons: 'toolbar_buttons'}}"
          :checkbox-config="{checkField: 'checked', labelField: 'name'}"
          :scroll-y="{enabled: true}">
          <template #toolbar_buttons>
            <vxe-button @click="loadColumnAndData(50, 20)">50列20条</vxe-button>
            <vxe-button @click="loadColumnAndData(100, 50)">100列50条</vxe-button>
            <vxe-button @click="loadColumnAndData(200, 100)">200列100条</vxe-button>
            <vxe-button @click="loadColumnAndData(5000, 10000)">5k列1w条</vxe-button>
            <vxe-button @click="loadColumnAndData(10000, 50000)">1w列5w条</vxe-button>
            <vxe-button @click="loadColumnAndData(10000, 100000)">1w列10w条</vxe-button>
            <vxe-button @click="$refs.xGrid.setAllCheckboxRow(true)">所有选中</vxe-button>
            <vxe-button @click="$refs.xGrid.clearCheckboxRow()">清除选中</vxe-button>
            <vxe-button @click="getSelectEvent">获取选中</vxe-button>
          </template>
        </vxe-grid>
        `,
        `
        import VXETable from 'vxe-table'
        
        export default {
          data () {
            return {
              loading: false
            }
          },
          created () {
            this.loadColumnAndData(20, 20)
          },
          methods: {
            loadColumnAndData (colSize, rowSize) {
              this.loading = true
              Promise.all([
                this.mockColumns(colSize),
                this.mockList(rowSize)
              ]).then(rest => {
                const columns = rest[0]
                const data = rest[1]
                const startTime = Date.now()
                const xGrid = this.$refs.xGrid
                // 使用函数式加载，阻断 vue 对大数组的双向绑定
                if (xGrid) {
                  Promise.all([
                    xGrid.reloadColumn(columns),
                    xGrid.reloadData(data)
                  ]).then(() => {
                    VXETable.modal.message({ content: \`渲染 \${colSize} 列 \${rowSize} 行，用时 \${Date.now() - startTime}毫秒\`, status: 'info' })
                    this.loading = false
                  })
                }
              })
            },
            mockColumns (size) {
              return new Promise(resolve => {
                const cols = []
                for (let index = 0; index < size; index++) {
                  if (index === 0) {
                    cols.push({
                      title: \`Col_\${index}\`,
                      type: 'seq',
                      fixed: 'left',
                      width: 100
                    })
                  } else if (index === 1) {
                    cols.push({
                      title: \`Col_\${index}\`,
                      type: 'checkbox',
                      fixed: 'left',
                      width: 200
                    })
                  } else if (index === size - 1) {
                    cols.push({
                      title: \`Col_\${index}\`,
                      field: \`col\${index}\`,
                      fixed: 'right',
                      width: 140
                    })
                  } else {
                    cols.push({
                      title: \`Col_\${index}\`,
                      field: \`col\${index}\`,
                      width: 160
                    })
                  }
                }
                resolve(cols)
              })
            },
            mockList (size) {
              return new Promise(resolve => {
                const list = []
                for (let index = 0; index < size; index++) {
                  list.push({
                    name: \`名称\${index}\`,
                    checked: false,
                    col1: '0',
                    col2: 123,
                    col3: 18,
                    col4: 234,
                    col5: 3,
                    col6: 'shenzhen',
                    col599: \`600_\${index}\`,
                    col999: \`1k_\${index}\`,
                    col4999: \`5k_\${index}\`,
                    col9999: \`1w_\${index}\`,
                    col19999: \`2w_\${index}\`
                  })
                }
                resolve(list)
              })
            },
            getSelectEvent () {
              let selectRecords = this.$refs.xGrid.getCheckboxRecords()
              VXETable.modal.alert(selectRecords.length)
            }
          }
        }
        `
      ]
    }
  },
  created () {
    this.loadColumnAndData(20, 20)
  },
  methods: {
    loadColumnAndData (colSize, rowSize) {
      this.loading = true
      Promise.all([
        this.mockColumns(colSize),
        this.mockList(rowSize)
      ]).then(rest => {
        const columns = rest[0]
        const data = rest[1]
        const startTime = Date.now()
        const xGrid = this.$refs.xGrid
        // 使用函数式加载，阻断 vue 对大数组的双向绑定
        if (xGrid) {
          Promise.all([
            xGrid.reloadColumn(columns),
            xGrid.reloadData(data)
          ]).then(() => {
            VXETable.modal.message({ content: `渲染 ${colSize} 列 ${rowSize} 行，用时 ${Date.now() - startTime}毫秒`, status: 'info' })
            this.loading = false
          })
        }
      })
    },
    mockColumns (size) {
      return new Promise(resolve => {
        const cols = []
        for (let index = 0; index < size; index++) {
          if (index === 0) {
            cols.push({
              title: `Col_${index}`,
              type: 'seq',
              fixed: 'left',
              width: 100
            })
          } else if (index === 1) {
            cols.push({
              title: `Col_${index}`,
              type: 'checkbox',
              fixed: 'left',
              width: 200
            })
          } else if (index === size - 1) {
            cols.push({
              title: `Col_${index}`,
              field: `col${index}`,
              fixed: 'right',
              width: 140
            })
          } else {
            cols.push({
              title: `Col_${index}`,
              field: `col${index}`,
              width: 160
            })
          }
        }
        resolve(cols)
      })
    },
    mockList (size) {
      return new Promise(resolve => {
        const list = []
        for (let index = 0; index < size; index++) {
          list.push({
            name: `名称${index}`,
            checked: false,
            col1: '0',
            col2: 123,
            col3: 18,
            col4: 234,
            col5: 3,
            col6: 'shenzhen',
            col599: `600_${index}`,
            col999: `1k_${index}`,
            col4999: `5k_${index}`,
            col9999: `1w_${index}`,
            col19999: `2w_${index}`
          })
        }
        resolve(list)
      })
    },
    getSelectEvent () {
      const selectRecords = this.$refs.xGrid.getCheckboxRecords()
      VXETable.modal.alert(selectRecords.length)
    }
  }
}
</script>

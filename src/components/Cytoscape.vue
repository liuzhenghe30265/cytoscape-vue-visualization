<template>
  <div
    class="cy-container">
    <div
      class="info">
      {{ info }}
    </div>
    <div
      class="btns">
      <span
        @click="zoomFun('zoom-in')">
        +
      </span>
      <span
        @click="zoomFun('zoom-out')">
        -
      </span>
      <span
        @click="zoomFun('resize')">
        reset
      </span>
      <span
        @click="refreshFun()">
        刷新布局
      </span>
    </div>
    <div
      id="cy"
      class="cy">
    </div>
  </div>
</template>


<script>
import cytoscape from 'cytoscape'
import cxtmenu from 'cytoscape-cxtmenu' // 圆形菜单
export default {
  name: 'Cytoscape',
  data() {
    return {
      info: ''
    }
  },
  mounted() {
    this.initCytoscape()
  },
  methods: {
    /**
     * @name: 添加元素
     * @param {*} eles 元素集合
     */
    addElements() {
      let eles = [
        { group: 'nodes', data: { id: 'yzjptu', name: '腰椎间盘突出症', type: 'type1' } },
        { group: 'nodes', data: { id: 'zjptu', name: '椎间盘突出', type: 'type1' } },
        { group: 'nodes', data: { id: 'yzgxz', name: '腰椎管狭窄', type: 'type1' } },
        { group: 'nodes', data: { id: 'yzgxz', name: '腰椎滑脱', type: 'type1' } },
        { group: 'nodes', data: { id: 'ybtt', name: '腰部疼痛', type: 'type2' } },
        { group: 'nodes', data: { id: 'xztt', name: '下肢疼痛', type: 'type2' } },
        { group: 'nodes', data: { id: 'ybdt', name: '腰部钝痛', type: 'type2' } },
        { group: 'nodes', data: { id: 'xzttptshj', name: '腰部钝痛，平躺时缓解', type: 'type2' } },
        { group: 'nodes', data: { id: 'man', name: '男性', type: 'type3' } },
        { group: 'nodes', data: { id: 'woman', name: '女性', type: 'type3' } },
        { group: 'nodes', data: { id: 'sixty', name: '60岁及以上', type: 'type4' } },
        { group: 'nodes', data: { id: 'fts', name: '45岁到60岁', type: 'type4' } },
        { group: 'nodes', data: { id: 'ztf', name: '0岁到14岁', type: 'type4' } },
        { group: 'edges', data: { id: 'yy', name: 'symptom-...', source: 'yzjptu', target: 'ybtt' } },
        { group: 'edges', data: { id: 'ym', name: 'symptom-...', source: 'yzjptu', target: 'man' } },
        { group: 'edges', data: { id: 'yw', name: 'symptom-...', source: 'yzjptu', target: 'woman' } },
        { group: 'edges', data: { id: 'ys', name: 'symptom-...', source: 'yzjptu', target: 'sixty' } },
        { group: 'edges', data: { id: 'yf', name: 'symptom-...', source: 'yzjptu', target: 'fts' } },
        { group: 'edges', data: { id: 'yztf', name: 'symptom-...', source: 'yzjptu', target: 'ztf' } },
        { group: 'edges', data: { id: 'ybdtxzttptshj', name: 'symptom-...', source: 'ybdt', target: 'xzttptshj' } },
        { group: 'edges', data: { id: 'ybdtyzjptu', name: 'symptom-...', source: 'ybdt', target: 'yzjptu' } },
        { group: 'edges', data: { id: 'xzttptshjybdt', name: 'symptom-...', source: 'xzttptshj', target: 'ybdt' } },
      ]
      for (let i = 0; i < eles.length; i++) {
        eles[i].position = this.usablePosition[i].position
      }
      this.usablePosition.splice(0, eles.length)

      // 开始批处理
      this.$cy.startBatch()
      this.$cy.batch(() => {
        let elements = ((Array.isArray ? Array.isArray(eles) : null != eles && eles instanceof Array) ? eles : [eles])
        let filterElements = elements.filter(__ => !this.$cy.getElementById(__.data.id).length)
        this.$cy.add(filterElements)
        // this.$cy.layout({ name: 'grid', randomize: false, animate: true }).run()
      })
      this.$cy.endBatch()
    },
    /**
     * @name: 删除元素及关联
     * @param {*} ID 元素 ID
     */
    delElement(ID) {
      if (ID) {
        this.$cy.startBatch()
        this.$cy.batch(() => {
          let selectedEles = this.$cy.$('#' + ID) // 根据 ID 删除
          // 未选择不进行操作
          if (!selectedEles || 1 > selectedEles.length) {
            return false
          }
          selectedEles.remove()
        })
        this.$cy.endBatch()
      } else {
        this.$cy.startBatch()
        this.$cy.batch(() => {
          let selectedEles = this.$cy.elements(':selected') // 根据当前选中的删除
          // 未选择不进行操作
          if (!selectedEles || 1 > selectedEles.length) {
            return false
          }
          selectedEles.remove()
        })
        this.$cy.endBatch()
      }
    },
    /**
     * @name: 刷新布局
     */
    refreshFun(type) {
      if (type) {
        this.$cy.layout({ name: type, randomize: false, animate: true }).run()
      } else {
        let arr = ['grid', 'breadthfirst', 'cose', 'preset', 'circle']
        let name = arr[Math.floor((Math.random() * arr.length))]
        this.$cy.layout({ name: name, randomize: false, animate: true }).run()
      }

    },
    /**
     * @name: 缩放
     * @param {*} type
     * @return {*}
     */
    zoomFun(type) {
      let selectedEles = this.$cy.elements('node:selected')
      let selectedEle = selectedEles && selectedEles.length ? selectedEles[0] : null
      let pan = this.$cy.pan() // 画布偏移位置
      let [x, y] = selectedEle ? [selectedEle.position('x'), selectedEle.position('y')] : [pan.x, pan.y] // 原点坐标
      let level = 0
      if (type === 'zoom-in') {
        level = this.$cy.zoom() + 0.1
        this.$cy.zoom({ level: level, renderedPosition: { x: x, y: y } })
      } else if (type === 'zoom-out') {
        level = this.$cy.zoom() - 0.1
        this.$cy.zoom({ level: level, renderedPosition: { x: x, y: y } })
      } else if (type === 'resize') {
        this.$cy.fit()
      }

    },
    initCytoscape() {
      let _this = this
      if (!cytoscape().cxtmenu) {
        cytoscape.use(cxtmenu)
      }
      cytoscape.warnings(false)
      this.$cy = cytoscape({
        container: document.getElementById('cy'),
        style: [
          {
            // 节点样式
            selector: 'node',
            style: {
              // 'content': 'data(id)',
              "content": function (ele) {
                return ele.data('name') || ele.data('id')
              },
              'text-opacity': 0.5,
              'height': 40,
              'width': 40,
              // "width": function (ele) {
              //   return ele.data('type') == 'type1' ? 70 : ele.data('type') == 'type2' ? 60 : ele.data('type') == 'type3' ? 40 : 30
              // },
              // "height": function (ele) {
              //   return ele.data('type') == 'type1' ? 70 : ele.data('type') == 'type2' ? 60 : ele.data('type') == 'type3' ? 40 : 30
              // },
              'background-color': function (ele) {
                return ele.data('type') == 'type1' ? '#47D2AC' : ele.data('type') == 'type2' ? '#2196F4' : ele.data('type') == 'type3' ? '#EAA829' : '#EAA830'
              },
              'pie-size': '100%',
              'text-valign': 'center',
              'text-halign': 'bottom',
            }
          },
          {
            // 路径样式
            selector: 'edge',
            // style: {
            //   'curve-style': 'bezier',
            //   'target-arrow-shape': 'triangle'
            // }
            style: {
              // 'width': 4,
              // // 'label': 'data(id)',
              // "label": function (ele) {
              //   return ele.data('name') || ele.data('id')
              // },
              // 'curve-style': 'bezier', // bezier 贝塞尔曲线
              // // 'line-color': '#666',
              // 'target-arrow-shape': 'triangle',
              // // 'target-arrow-fill': 'hollow',
              // 'target-arrow-color': '#333',

              'curve-style': 'bezier',
              'target-arrow-shape': 'triangle',
              'width': 1,
              'line-color': '#ddd',
              'target-arrow-color': '#ddd',
              'content': 'data(relationship)'
            }

          },
          {
            // 点击选中样式
            selector: ':selected',
            style: {
              // "border-width": 3,
              "border-color": '#333',
              // "background-color": 'black',
              // "line-color": 'black',
              // "target-arrow-color": 'black',
              // "source-arrow-color": 'black'
            }
          },
          {
            selector: ".nodeHover",
            style: {
              "shape": "ellipse",
              "background-opacity": .8
            }
          },
          {
            selector: ".nodeLock",
            style: {
              "shape": "ellipse",
              "background-opacity": .5
            }
          },
          {
            selector: ".nodeActive",
            style: {
              "border-color": '#4EA2F0',
              "border-width": 10,
              "border-opacity": .8
            }
          },
          {
            selector: ".edgeShow",
            style: {
              "color": "#999",
              "text-opacity": 1,
              "font-weight": 400,
              "label": function (ele) {
                return ele.data("name")
              },
              "font-size": 10,
              "arrow-scale": .8,
              "width": 1.5,
              "source-text-margin-y": 20,
              "target-text-margin-y": 20,
            },
          },
          {
            selector: ".edgeActive",
            style: {
              "arrow-scale": .8,
              "width": 1.5,
              "color": "#330",
              "text-opacity": 1,
              "font-size": 10,
              "text-background-color": "#fff",
              "text-background-opacity": .8,
              "text-background-padding": 0,
              "source-text-margin-y": 20,
              "target-text-margin-y": 20,
              "z-index-compare": "manual",
              "z-index": 1,
              "line-color": function () {
                return "#4EA2F0"
              },
              "target-arrow-color": function () {
                return "#4EA2F0"
              },
              label: function (a) {
                return a.data("label")
              }
            }
          },
          {
            selector: ".dull",
            style: {
              "z-index": 1,
              "opacity": .5
            }
          },
          {
            selector: ':parent',
            css: {
              'text-valign': 'top',
              'text-halign': 'center',
              // 'text-halign': 'right',
              // 'text-rotation': '90deg',
            }
          },
        ],
        // id 必须为字母
        elements: {
          nodes: [
            { data: { id: 'yzjptu', name: '腰椎间盘突出症', type: 'type1' } },
            { data: { id: 'zjptu', name: '椎间盘突出', type: 'type1' } },
            { data: { id: 'yzgxz', name: '腰椎管狭窄', type: 'type1' } },
            { data: { id: 'yzgxz', name: '腰椎滑脱', type: 'type1' } },
            { data: { id: 'ybtt', name: '腰部疼痛', type: 'type2' } },
            { data: { id: 'xztt', name: '下肢疼痛', type: 'type2' } },
            { data: { id: 'ybdt', name: '腰部钝痛', type: 'type2' } },
            { data: { id: 'xzttptshj', name: '腰部钝痛，平躺时缓解', type: 'type2' } },
            { data: { id: 'man', name: '男性', type: 'type3' } },
            { data: { id: 'woman', name: '女性', type: 'type3' } },
            { data: { id: 'sixty', name: '60岁及以上', type: 'type4' } },
            { data: { id: 'fts', name: '45岁到60岁', type: 'type4' } },
            { data: { id: 'ztf', name: '0岁到14岁', type: 'type4' } },
          ],
          edges: [
            { data: { id: 'yy', name: 'symptom-...', source: 'yzjptu', target: 'ybtt' } },
            { data: { id: 'ym', name: 'symptom-...', source: 'yzjptu', target: 'man' } },
            { data: { id: 'yw', name: 'symptom-...', source: 'yzjptu', target: 'woman' } },
            { data: { id: 'ys', name: 'symptom-...', source: 'yzjptu', target: 'sixty' } },
            { data: { id: 'yf', name: 'symptom-...', source: 'yzjptu', target: 'fts' } },
            { data: { id: 'yztf', name: 'symptom-...', source: 'yzjptu', target: 'ztf' } },
            { data: { id: 'ybdtxzttptshj', name: 'symptom-...', source: 'ybdt', target: 'xzttptshj' } },
            { data: { id: 'ybdtyzjptu', name: 'symptom-...', source: 'ybdt', target: 'yzjptu' } },
            { data: { id: 'xzttptshjybdt', name: 'symptom-...', source: 'xzttptshj', target: 'ybdt' } },
          ]
        },
        layout: {
          name: 'grid', // 用哪种方式排列，可选：breadthfirst(广度优先)、cose(缝制)、preset(预设)、circle(圆形)、grid(矩形)
          idealEdgeLength: 100,
          nodeOverlap: 20,
          refresh: 20,
          fit: true,
          padding: 30,
          randomize: false,
          componentSpacing: 20,
          nodeRepulsion: 400,
          edgeElasticity: 10,
          nestingFactor: 5,
          animate: true,
          gravity: 80,
          numIter: 1000,
          initialTemp: 200,
          coolingFactor: 0.95,
          minTemp: 1.0,
        }
      })

      // 添加节点、关系线
      // this.$cy.add({
      //   group: 'nodes', data: { id: 'add1' }, position: { x: 500, y: 500 }
      // })
      // this.$cy.add([
      //   { group: 'nodes', data: { id: 'add2' }, position: { x: 650, y: 650 } },
      //   { group: 'nodes', data: { id: 'add3' }, position: { x: 650, y: 500 } },
      //   { group: 'edges', data: { id: 'add2-add3', source: 'add2', target: 'add3' } },
      //   { group: 'edges', data: { id: 'add1-add2', source: 'add1', target: 'add2' } },
      //   { group: 'edges', data: { id: 'add3-add1', source: 'add3', target: 'add1' } },
      // ])

      // 移除元素
      // cy.remove(cy.$('#add2'))
      // cy.remove(cy.$('#BA'))
      this.$cy.remove(this.$cy.$(this.$cy.elements('node[weight > 50]')))

      // cy.collection()
      // cy.getElementById()
      // cy.$() et al
      // cy.batch() et al
      // cy.mount()
      // cy.unmount()
      // cy.destroy()
      // cy.destroyed()

      this.$cy.collection('edge').addClass('edgeShow')
      this.$cy.on("mouseover", "node", function (ele) {
        let node = document.getElementById('cy')
        node.style.cursor = 'move'
        let c = ele.target
        c.addClass("nodeHover")
        _this.$cy.collection('edge').removeClass('edgeActive')
        c.neighborhood('edge').addClass('edgeActive')
      })
      this.$cy.on("mouseout", "node", function (a) {
        let node = document.getElementById('cy')
        node.style.cursor = 'default'
        let c = a.target
        c.removeClass("nodeHover")
        _this.$cy.collection("edge").removeClass("edgeActive")
      })
      this.$cy.on("click", "node", function (ele) {
        let c = ele.target
        c.removeClass("nodeActive")
        _this.$cy.collection("edge").removeClass("edgeActive")
        _this.info = 'id：' + c.data('id') + '；name：' + c.data('name') + '；type：' + c.data('type')
      })
      // 监听鼠标按下左键
      this.$cy.on("vmousedown", function () {
        // 取消高亮
        _this.$cy.collection("edge").removeClass('dull')
        _this.$cy.collection("node").removeClass('dull')
      })
      this.$cy.on("vmousedown", "node", function (a) {
        // 高亮有关联的节点
        let c = a.target
        _this.$cy.collection("edge").addClass('dull')
        _this.$cy.collection("node").addClass('dull')
        c.removeClass("dull")
        c.neighborhood("edge").removeClass("dull")
        c.neighborhood("edge").addClass("edgeActive")
        c.neighborhood("edge").connectedNodes().removeClass("dull")
        c.neighborhood("node").removeClass("dull")
      })
      //监听鼠标左键释放
      // eslint-disable-next-line no-unused-vars
      this.$cy.on("tapend", "node", function (a) {
        // 取消高亮
        // let c = a.target
        // cy.collection("edge").removeClass('dull')
        // cy.collection("node").removeClass('dull')
        // c.neighborhood("edge").removeClass("edgeActive")
        // c.neighborhood("node").removeClass("nodeActive")
      })
      this.$cy.on("mouseover", "edge", function (a) {
        let c = a.target
        _this.$cy.collection("edge").removeClass("edgeActive")
        c.addClass("edgeActive")
      })
      this.$cy.on("mouseout", "edge", function (a) {
        let c = a.target
        c.removeClass("edgeActive")
      })

      this.$cy.autolock(false) // 锁定所有节点

      // 圆形菜单
      this.$cy.cxtmenu({
        menuRadius: 80, // 菜单大小
        selector: 'node',
        commands: () => {
          return [
            {
              fillColor: 'rgba(200, 200, 200, 0.75)',
              content: '收起',
              contentStyle: {},
              select: () => {
                let currentEle = _this.$cy.elements('node') // 当前所有节点
                console.log(currentEle)
              },
              enabled: true, // 启用/禁用
            },
            // {
            //   fillColor: 'rgba(200, 200, 200, 0.75)',
            //   content: _this.$cy.$('#' + element.id()).locked() ? '解锁' : '锁定',
            //   contentStyle: {},
            //   select: (ele) => {
            //     _this.$cy.$('#' + ele.id()).locked() ? (() => {
            //       _this.$cy.$('#' + ele.id()).unlock()
            //       ele.removeClass("nodeLock")
            //     })() : (() => {
            //       ele.addClass("nodeLock")
            //       _this.$cy.$('#' + ele.id()).lock()
            //     })()
            //   },
            //   enabled: true, // 启用/禁用
            // },
            {
              // fillColor: 'rgba(200, 200, 200, 0.75)', 
              content: '<span>隐藏</span>',
              // contentStyle: {},
              select: (ele) => {
                this.delElement(ele.id())
              },
              enabled: true,
            },
            {
              // fillColor: 'rgba(200, 200, 200, 0.75)',
              content: '展开',
              // contentStyle: {},
              select: (ele) => {
                // 与目标节点相关联的节点和关系
                let elesData = [
                  { group: 'nodes', data: { id: 'yzjptun', name: '腰椎间盘突出症', type: 'type1', class: 'test' } },
                  { group: 'nodes', data: { id: 'zjptun', name: '椎间盘突出', type: 'type1', class: 'test' } },
                  { group: 'nodes', data: { id: 'yzgxzn', name: '腰椎管狭窄', type: 'type1', class: 'test' } },
                  { group: 'nodes', data: { id: 'yzgxzn', name: '腰椎滑脱', type: 'type1' } },
                  { group: 'nodes', data: { id: 'ybttn', name: '腰部疼痛', type: 'type2' } },
                  { group: 'nodes', data: { id: 'xzttn', name: '下肢疼痛', type: 'type2' } },
                  { group: 'nodes', data: { id: 'ybdtn', name: '腰部钝痛', type: 'type2' } },
                  { group: 'nodes', data: { id: 'xzttptshjn', name: '腰部钝痛，平躺时缓解', type: 'type2' } },
                  { group: 'nodes', data: { id: 'mann', name: '男性', type: 'type3' } },
                  { group: 'nodes', data: { id: 'womann', name: '女性', type: 'type3' } },
                  { group: 'nodes', data: { id: 'sixtyn', name: '60岁及以上', type: 'type4' } },
                  { group: 'nodes', data: { id: 'ftsn', name: '45岁到60岁', type: 'type4' } },
                  { group: 'nodes', data: { id: 'ztfn', name: '0岁到14岁', type: 'type4' } },
                  { group: 'edges', data: { id: 'yy', name: 'symptom-...', source: ele.id(), target: 'ybtt' } },
                  { group: 'edges', data: { id: 'ym', name: 'symptom-...', source: ele.id(), target: 'man' } },
                  { group: 'edges', data: { id: 'yw', name: 'symptom-...', source: ele.id(), target: 'woman' } },
                  { group: 'edges', data: { id: 'ys', name: 'symptom-...', source: ele.id(), target: 'sixty' } },
                  { group: 'edges', data: { id: 'yf', name: 'symptom-...', source: ele.id(), target: 'fts' } },
                  { group: 'edges', data: { id: 'yztf', name: 'symptom-...', source: ele.id(), target: 'ztf' } },
                  { group: 'edges', data: { id: 'ybdtxzttptshj', name: 'symptom-...', source: ele.id(), target: 'xzttptshj' } },
                  { group: 'edges', data: { id: 'ybdtyzjptu', name: 'symptom-...', source: ele.id(), target: 'yzjptu' } },
                  { group: 'edges', data: { id: 'xzttptshjybdt', name: 'symptom-...', source: ele.id(), target: 'ybdt' } },
                ]
                // 模拟数据，设定不重复 id
                // for (let i = 0; i < elesData.length; i++) {
                //   elesData[i].data.id = ele.id() + elesData[i].data.id
                // }
                // // 如果当前要添加的节点数大于预生成的位置数，就从屏幕随机取位置
                // if (elesData.length > this.usablePosition.length) {
                //   for (let i = 0; i < elesData.length; i++) {
                //     elesData[i].position = {
                //       x: Math.floor(Math.random() * (window.innerWidth - 0) + 0),
                //       y: Math.floor(Math.random() * (window.innerHeight - 0) + 0)
                //     }
                //   }
                // } else {
                //   for (let i = 0; i < elesData.length; i++) {
                //     elesData[i].position = this.usablePosition[i].position
                //   }
                // }
                // this.usablePosition.splice(0, elesData.length)
                // 开始批处理
                this.$cy.startBatch()
                this.$cy.batch(() => {
                  let elements = ((Array.isArray ? Array.isArray(elesData) : null != elesData && elesData instanceof Array) ? elesData : [elesData])
                  let filterElements = elements.filter(__ => !this.$cy.getElementById(__.data.id).length)
                  this.$cy.add(filterElements)
                })
                this.$cy.endBatch()
                console.log(this.usablePosition)
              },
              enabled: true,
            }
            // {
            //   // fillColor: 'rgba(200, 200, 200, 0.75)',
            //   content: '禁用',
            //   // contentStyle: {},
            //   select: (ele) => {
            //     console.log('禁用', ele.id())
            //   },
            //   enabled: false,
            // }
          ]
        },
        fillColor: 'rgba(0, 0, 0, 0.75)', // 菜单背景
        activeFillColor: 'rgba(1, 105, 217, 0.75)', // 选中背景
        activePadding: 10, // 选中的菜单大小
        indicatorSize: 14, // 指针大小
        separatorWidth: 4, // 相邻菜单间距
        spotlightPadding: 10, // 焦点填充
        minSpotlightRadius: 10, // 焦点最小半径
        maxSpotlightRadius: 14, // 焦点最大半径
        openMenuEvents: 'cxttapstart taphold', // cxttapstart（鼠标右键触发）、taphold（鼠标左键加空格键触发）
        itemColor: 'white', // 指令元素内字体颜色
        itemTextShadowColor: 'red', // 各指令元素内字体阴影颜色
        zIndex: 9999,
        atMouse: true, // 在鼠标位置绘制菜单
      })
    }
  }
}
</script>

<style>
.cy-container {
  width: 100%;
  height: 100%;
  position: absolute;
  top: 0;
  left: 0;
  bottom: 0;
}
.cy-container .cy {
  width: 100%;
  height: 100%;
}
.cy-container .info {
  background: #fff;
  position: absolute;
  left: 50px;
  bottom: 50px;
  z-index: 9;
}
.cy-container .btns {
  background: #fff;
  position: absolute;
  right: 50px;
  bottom: 200px;
  z-index: 99;
}
.cy-container .btns span {
  cursor: pointer;
}
.cy-container .types {
  position: absolute;
  left: 50px;
  top: 50px;
  z-index: 9;
}
.cy-container .types span {
  display: inline-block;
  background: #a5abb6;
  border-radius: 4px;
  margin-right: 10px;
  padding: 5px 6px;
  cursor: pointer;
  color: #2e0f00;
  font-size: 12px;
}
</style>
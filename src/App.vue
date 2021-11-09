<template>
  <div class="parent">
    <div>
      <button @click="show">显示</button>
      <button @click="hide">隐藏</button>
    </div>
    <div id="container"></div>
    <div id="map"></div>
  </div>
</template>

<script>
// import HelloWorld from './components/HelloWorld.vue'
import { Graph } from '@antv/x6'
import '@antv/x6-vue-shape'
// const data = [];
import Node from './components/node.vue'
import data from "./data"
// 使用 Mock
export default {
  name: 'App',
  data() {
    return {
      graph: null,
      nodes: []
    }
  },
  mounted() {
    // var Mock = require('mockjs')
    // var data = Mock.mock({
    //   // 属性 list 的值是一个数组，其中含有 1 到 10 个元素
    //   'list|1-10': [
    //     {
    //       // 属性 id 是一个自增数，起始值为 1，每次增 1
    //       'id|+1': 1,
    //       'x|0-1000': 0,
    //       'y|0-1000': 0,
    //       name: '@name',
    //       'relations|1-3': [
    //         {
    //           'id|+1': 100,
    //           name: '@name',
    //           'targetId|+2': 1
    //         }
    //       ]
    //     }
    //   ]
    // })
    this.nodes = data.list;
    // 输出结果
    console.log(JSON.stringify(data, null, 4))
    console.log(data)
    Graph.registerVueComponent(
      'node-component',
      {
        template: `<node></node>`,
        components: {
          Node
        }
      },
      true
    )
    Graph.registerPortLayout("in", (portsPositionArgs) => {
      return portsPositionArgs.map((arg) => {
        return {
          position: {
            x: arg.x,
            y: arg.y,
          },
          angle: 0,
        };
      });
    });
    Graph.registerPortLayout("header", (portsPositionArgs) => {
      return portsPositionArgs.map(() => {
        return {
          position: {
            x: -4,
            y: 16,
          },
          // angle: 0,
        };
      });
    });
    const container = document.getElementById("container")
    this.graph = new Graph({
      // async: true,
      panning: {
        enabled: true, // 画布可以被拖动
        // modifiers: "Space",
        eventTypes: ["leftMouseDown", "rightMouseDown", "mouseWheel"],
      },
      highlighting: {
        // 当链接桩可以被链接时，在链接桩外围渲染一个 2px 宽的红色矩形框
        magnetAvailable: {
          name: "stroke",
          args: {
            padding: 4,
            attrs: {
              "stroke-width": 2,
              stroke: "red",
            },
          },
        },
      },
      selecting: {
        enabled: false,
      },
      // 挂载dom
      container: container,
      width: container.clientWidth,
      height: container.clientHeight,
      grid: true,
      autoResize: true, // 窗口改变自动重置画布大小
      minimap: {
        enabled: true,
        width: 200,
        height: 200,
        container: document.getElementById("map"),
      },
      scroller: {
        // 设置滚动平移
        enabled: true,
      },
      // history: true, // 撤销/重做
      keyboard: {
        enabled: true, // 快捷键
      },
      // 定义默认连接线
      connecting: {
        allowBlank: false,
        router: {
          name: "metro",
          // name: "manhattan",
          args: {
            startDirections: ["right"],
            endDirections: ["left"],
          },
        },
      },
      interacting(cellView) {
        if (cellView.cell.getProp("customLinkInteractions")) {
          // return { vertexAdd: false };
        }
        return true;
      },
    });
    const nodes = []
    this.nodes.forEach((item) => {
      const isE = nodes.find((i) => i.id === item.id);
      if (isE) return;
      nodes.push({
        shape: "vue-shape",
        x: item.x,
        y: item.y,
        width: 220,
        height: 100,
        id: item.id,
        data: item,
        zIndex: 10,
        ports: {
          groups: {
            header: {
              position: "header",
              attrs: {
                circle: {
                  r: 4,
                  magnet: true,
                  stroke: "#31d0c6",
                  strokeWidth: 0,
                  fill: "#fff",
                  fillOpacity: 0,
                },
              },
            },
            relation: {
              position: "in",
              attrs: {
                circle: {
                  r: 0,
                  magnet: true,
                  stroke: "#eee",
                  strokeWidth: 0,
                  fill: "#fff",
                },
              },
            },
          },
          items: [
            {
              id: "port" + item.id,
              group: "header",
            },
          ],
        },
        component: "node-component",
      });
    });
    this.graph.addNodes(nodes);
    setTimeout(() => {
      this.connectedNodes();
    }, 200)
  },
  methods: {
    show() {
      // const nodes = this.graph.getNodes();
      this.graph.getNodes().forEach(node => {
        node.setVisible(true);
      })
    },
    hide() {
      this.graph.getNodes().forEach(node => {
        node.setVisible(false);
      })
    },
    connectedNodes() {
      this.nodes.forEach((i) => {
        if (i?.relations) {
          i.relations.forEach((relation) => {
            const target = this.nodes.find((i) => i.id === relation?.targetId);
            if (!target) return;
            this.addRelationEdge(i, relation, target);
          });
        }
      });
    },
    /**
     * 添加连线
     * @param source 源节点
     * @param field 字段
     * @param target 目标
     */
    addRelationEdge(source, field, target) {
      const portId = `port${source.id}-${field.id}`;
      const sourceNode = this.graph.getCellById(source.id);
      const curPort = sourceNode.getPort(portId);
      if (curPort) return;
      // 1. 计算字段位置
      const position = this.getRelationFieldPosition(source, field.id, target);
      if (!position) return;
      // 2. 创建连接桩
      const port = {
        id: `port${source.id}-${field.id}`,
        group: "relation",
        zIndex: 2,
        attrs: {
          circle: {
            r: 4,
            magnet: true,
            // stroke: "#ccc",
            strokeWidth: 0,
            fill: "#fff",
            fillOpacity: 0,
          },
        },
        args: {
          x: position.x,
          y: position.y,
        },
      };
      // 3. 添加连接桩,并设置连线.
      const targetNode = this.graph.getCellById(target.id);
      sourceNode.addPort(port);
      this.graph.addEdge({
        source: {
          cell: sourceNode,
          port: `port${source.id}-${field.id}`,
        },
        target: {
          cell: targetNode,
          port: `port${target.id}`,
        },
        attrs: {
          line: {
            // sourceMarker: marker,
            targetMarker: {
              name: "circle",
              fill: "none",
              stroke: "#ccc", // 使用自定义边框色
              strokeWidth: 2,
            },
          },
        },
        zIndex: 1,
        // connector: 'smooth',
      })
    },
    // 获取关联字段位置
    getRelationFieldPosition(source, fieldId, target) {
      const sourceDom = document.querySelector(`.node[data-id="${source.id}"]`);
      if (!sourceDom) return null;
      const f = sourceDom.querySelector(`.node-data-item[data-id="${fieldId}"]`);
      if (!f) return null;

      const targetDom = document.querySelector(`.node[data-id="${target.id}"]`);
      if (!targetDom) return null;
      const sourceRect = sourceDom?.getBoundingClientRect();
      const targetRect = targetDom?.getBoundingClientRect();
      return {
        x: sourceRect.x < targetRect.x ? sourceRect.width : 0,
        y: f.getBoundingClientRect().y - sourceRect.y + f.clientHeight / 2,
      };
    }
  }
  // components: {
  //   HelloWorld
  // }
}
</script>

<style>
* {
  padding: 0;
  margin: 0;
  box-sizing: border-box;
}
html {
  height: 100%;
  width: 100%;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  width: 100%;
  height: 100%;
}
.parent {
  width: 100vw;
  height: 100vh;
  position: relative;
  display: flex;
  flex-direction: column;
  box-sizing: border-box;
}
#container {
  height: 100%;
  flex: 1;
}
#map {
  position: absolute;
  right: 40px;
  bottom: 40px;
  width: 200px;
  height: 200px;
}
</style>

<template>
  <div id="app">
    <div class='container'>
      <div class='treeSelf'>
        <tree ref="tree_1" @node-click="nodeClick" @operator-click="opeClick" :root='data_1' :drag-option='dragOption' :default-text='"新生成接点"' :hover-color='"lightblue"' :operator-offset='20' :highlight-color='"green"'>
          <div slot="left">
            <div v-if="node">
              <span>{{node.name}}</span>
              <li v-for="(item, idx) in node.binded" :key="idx">{{ item }}<span>解绑</span></li>
            </div>
          </div>
          <div slot="right">
            <ul>
              <li><span>Add</span></li>
              <li><span>Del</span></li>
            </ul>
          </div>
        </tree>
      </div>
      <div class='showSec'>
        <pre>{{formatData}}</pre>
      </div>
    </div>
  </div>
</template>

<script>
  import Vue from 'Vue'
  import Tree from './components/index'
  export default {
    name: 'Hello',
    data() {
      return {
        node: null,
        dragOption: {
          draggable: true,
          highlight: true,
          highlightColor: '#ccc'
        },
        en: false,
        data_1: {
          name: '华东区',
          id: 0,
          binded: [{
            name: '苏州',
            id: 1111111
          }, {
            name: '苏州',
            id: 2222222
          }],
          children: []
        },
      }
    },
    computed: {
      formatData() {
        return JSON.stringify(this.data_1, null, 2)
      }
    },
    methods: {
      nodeDragDrop(ev, addFunc, removeFunc, model) {
        addFunc({
          name: 'Drag',
          id: '哈哈哈'
        })
      },
      nodeClick(ev, node) {
        this.node = node;
        let type = ev.button ? 'right' : 'left'
        this.$refs.tree_1.doShow(type)
      },
      opeClick(ev) {
        console.log(arguments)
      },
      assignData(data) {
        this.data = data
      }
    },
    components: {
      tree: Tree
    },
    mounted() {}
  }
</script>

<style scoped>
  .main {
    display: flex;
    flex-direction: column;
    align-items: center;
  }
  .sentence {
    background: #D3DCE6;
    margin-top: 2rem;
    width: 50%;
    padding: 1rem;
    font-size: 25px;
  }
  .senMain {
    background: white;
    padding: 1rem;
    color: #8492A6;
  }
  .lang {
    font-size: 20px;
    color: #8492A6;
    font-weight: bold;
  }
  ul {
    margin-left: 1rem;
  }
  .container {
    width: 60%;
    display: flex;
    margin-top: 2rem;
    justify-content: space-between;
    padding: 0 5rem;
  }
  .treeSelf {
    border: 3px solid #E5E9F2;
    width: 40%;
    text-align: left;
    padding: 1rem;
  }
  .showSec {
    border: 3px solid #E5E9F2;
    width: 40%;
    text-align: left;
    padding: 1rem;
    font-size: 15px;
  }
  .howtouse {
    text-align: center;
    width: 30%;
    background: #E5E9F2;
    margin-top: 1rem;
    padding: 1rem;
    font-size: 50px;
    font-weight: bold;
  }
  .howtouse>div {
    font-family: 'Tangerine';
    font-size: 5rem;
  }
  .getstart>div {
    background: #D3DCE6;
    padding: 0.5rem 1rem;
  }
  .getstart>p {
    margin: 0.5rem 0;
  }
  a {
    color: black;
  }
</style>

<style>
  .tree-view-item-key {
    font-size: 20px;
    color: black;
  }
  .tree-view-item-value {
    font-size: 20px;
    font-weight: bold;
    color: white;
  }
</style>
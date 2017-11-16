<template>
  <div>
    <div class='treeNodeText' :id='model.id' :draggable='draggable' @dragstart='dragStart' @keyup.enter="editSubmit" @dragover='dragOver' @dragenter='dragEnter' @dragleave='dragLeave' @drop='drop' @dragend='dragEnd' :style='styleObj'>
      <span v-show="!isLoading" :class='[isOpen ? "node-icon-expanded" : "", "node-icon"]' @click='toggle'></span>
      <span v-show="isLoading" class="el-icon-loading"></span>
      <span v-show='!isEditName' @dblclick='dblclick' @mousedown='handNodeMouseDown'>{{model.name}}</span>
      <input v-show='isEditName' type="text" v-model='model.name'>
    </div>
    <div class='treeMargin' v-show='isOpen' v-if='isFolder'>
      <tree-node v-for='model in model.children' :model='model' :key='model.id' :draggable="draggable">
      </tree-node>
    </div>
  </div>
</template>

<script>
  import Vue from 'Vue';
  let id = 1000
  export default {
    name: 'treeNode',
    data() {
      return {
        isOpen: false,
        isEditName: false,
        form: {
          name: ''
        },
        isLoading: false,
        styleObj: {
          background: 'white',
        },
        rules: {
          name: [{
              required: true,
              message: '请输入名称',
              trigger: 'blur'
            }
          ]
        }
      }
    },
    props: {
      model: Object,
      // 当前节点高亮
      draggable: {
        type: Boolean,
        default: false
      }
    },
    computed: {
      /**
       * is Children exist 
       */
      isFolder() {
        return this.model.children &&
          this.model.children.length
      },
    },
    methods: {
      handNodeClick() {
        console.log('click')
      },
      handNodeMouseDown(ev) {
        console.log('mousedown')
        // ev.preventDefault();
        let tree = this.tree,
          posi = tree.operator.position;
        /**
         * currentNode 被点击节点
         * 
         */
        tree.currentNode = this.model
        tree.getArguement = this.getArguement.bind(this)
        posi.top = ev.clientY;
        posi.left = ev.clientX;
        tree.$emit('node-click', ev, ...this.getArguement(), this.model)
        // TODO 手动计算
        // layerX -- 非标准属性
        // 找到相对于包含框的距离
        // let tar = ev.target
        // let parent = tar.parentNode;
        // let totalPosi = {
        //   top: tar.offsetTop,
        //   left: tar.offsetLeft
        // };
        // while ( parent !== tar.offsetParent) {
        //   totalPosi.top += parent.offsetTop;
        //   totalPosi.left += parent.offsetLeft;
        //   console.log(parent,totalPosi)
        //   parent = parent.parentElement
        // }
        // debugger
      },
      toggle() {
        const tree = this.tree
        tree.doHide()
        if (tree.lazy && tree.loadFn && !this.load) {
          /**
           * 设置加载图标
           * 传递函数设置子节点
           * 考虑增加标志，记录其子节点是否已经加载
           */
          if (!this.model.children) {
            this.$set(this.model, 'children', [])
          }
          this.isLoading = true;
          tree.loadFn(this.model, (data) => {
            if (!data instanceof Array) {
              return console.error('data must be Array');
            }
            this.model.children.push(...data)
            this.isLoading = false;
            this.isOpen = !this.isOpen
          }, () => {
            this.isLoading = this.loading = false;
          })
        } else {
          if (this.isFolder) {
            this.isOpen = !this.isOpen
          }
          /**
           * 高亮功能
           * 即仅突出当前被点击的节点同时关闭其他所有节点
           * 涉及树结构深层遍历
           * 树节点数量较大不建议使用
           */
          if (tree.currentHighlight) {
            // 防止已高亮节点被再次点击
            // 遍历重置所有树组件的高亮样式
            // let treeParent = rootTree.$parent
            // let nodeStack = [treeParent.$children[0]]
            // while (nodeStack.length != 0) {
            //   let item = nodeStack.shift()
            //   item.styleObj.background = 'white'
            //   if (item.$children && item.$children.length > 0) {
            //     nodeStack = nodeStack.concat(item.$children)
            //   }
            // }
            // 然后把当前节点的样式设置为高亮
            this.styleObj.background = tree.highlightColor ? tree.highlightColor : '#99A9BF'
            // 设置为当前节点
          }
        }
      },
      exchangeData(rootCom, from, to) {
        //如果drag的目的地是 + - 符号的话，退出。
        if (!to || !from || typeof to == 'string' || from.id == to.id) {
          return
        }
        from = JSON.parse(JSON.stringify(from))
        to = JSON.parse(JSON.stringify(to))
        // copy一个,最后再赋值给state.treeData.这样能保证值的变化都会触发视图刷新(因为JS判断引用类型是否相同是根据内存地址.)
        let treeData = JSON.parse(JSON.stringify(this.model))
        let nodeStack = [treeData]
        let status = 0
        // 如果from或者to节点存在父子/祖辈关系，会报id of undefined的错。这种情况不考虑拖拽功能，所以catch住，返回/return就行
        try {
          // 广度优先遍历,找到涉及数据交换的两个对象.然后交换数据.
          while (!(status === 2)) {
            let item = nodeStack.shift()
            if (item.id == from.id) {
              item.id = to.id
              item.name = to.name
              if (to.children && to.children.length > 0) {
                item['children'] = to.children
              } else {
                item.children = []
              }
              status++
              //找到后,跳出当前循环.
              continue;
            }
            if (item.id == to.id) {
              item.id = from.id
              item.name = from.name
              if (from.children && from.children.length > 0) {
                item['children'] = from.children
              } else {
                item.children = []
              }
              status++
              //找到后,跳出当前循环.
              continue;
            }
            if (item.children && item.children.length > 0) {
              nodeStack = nodeStack.concat(item.children)
            }
          }
        } catch (e) {
          return
        }
        //API: 对外开放交换后的数据的赋值操作
        rootCom.assignData(treeData)
      },
      dblclick() {
        const tree = this.tree;
        if (tree.edit) {
          this.isEditName = !this.isEditName
        }
        this.tree.doHide()
      },
      editSubmit(ev) {
        ev.preventDefault()
        this.$refs.form
            .validate((isValid) => {
              this.isEditName = !isValid
            })
        // return false
      },
      addChild(node) {
        let def = {
          name: this.tree.defaultText ? this.tree.defaultText : 'New Node',
          id: id++
        }
        if (!this.model.children) {
          this.$set(this.model, 'children', [])
        }
        this.model.children.push(node ? node : def)
        // open the folder
        this.isOpen = true;
      },
      removeChild(id) {
        // 获取父组件的model.children
        // 在父组件model.children里删除
        if (this === this.tree) {
          return console.error('Root can\'t not delete!')
        }
        let parentModelChildren = this.$parent.model.children
        for (let index in parentModelChildren) {
          // 找到该删的id
          if (parentModelChildren[index].id == id) {
            parentModelChildren.splice(index, 1)
            break
          }
        }
      },
      dragStart(ev) {
        this.tree.doHide()
        ev.dataTransfer.effectAllowed = 'move';
        // TODO
        // IE 浏览器 setData Key 需要修改
        ev.dataTransfer.setData('text/plain', this.model.id);
        this.tree.$emit('node-drag-start', ev, ...this.getArguement(), this.model)
        return true
      },
      dragEnd(ev) {
        const tree = this.tree;
        tree.dragEnd()
      },
      dragOver(ev) {
        ev.preventDefault()
        return true
      },
      dragEnter(ev) {
        const target = ev.target,
          className = target.className;
        if (className.indexOf('treeNodeText') !== -1) {
          let tree = this.tree,
            style = ev.target.style;
          Object.assign(style, tree.dragOption.style)
        }
        /**
         * 无需拖拽交换数据
         * 暂时关闭该功能
         */
        // rootTree.exchangeData(rootTree.$parent, fromData, toData)
      },
      dragLeave(ev) {
        if (ev.target.tagName !== 'SPAN') {} else {
          this.clearStyle(ev.target, this.tree.dragOption.style)
          console.log(ev.target)
        }
      },
      drop(ev) {
        this.clearStyle(ev.target, this.tree.dragOption.style)
        this.tree.$emit('node-drag-drop', ev, ...this.getArguement(), this.model)
      },
      getArguement() {
        return [this.addChild.bind(this), this.removeChild.bind(this, this.model.id)]
      },
      clearStyle(ele, style) {
        if (!style)
          return
        while (ele.className.indexOf('treeNodeText') === -1) {
          ele = ele.parentElement;
        }
        const keys = Object.keys(style) || []
        keys.forEach(key => ele.style[key] = '')
      }
    },
    /**
     * 将根节点的Vue实例挂载在tree上
     * 方便子节点与父节点进行交互
     */
    created() {
      const parent = this.$parent;
      if (parent.isTree) {
        this.tree = parent;
      } else {
        this.tree = parent.tree;
      }
      const tree = this.tree;
      if (!tree) {
        console.warn('Can not find node\'s tree.');
      }
      // console.log('this.hig', this.highlightColor, '|', this.hoverColor)
    },
  }
</script>

<style>
  .treeNodeText {
    padding: 0.4rem 0.5rem;
    width: fit-content;
    font-size: 14px;
  }
  .treeMargin {
    margin-left: 2rem;
  }
  .changeTree {
    width: 1rem;
    color: #324057;
  }
  .node-icon {
    display: inline-block;
    width: 0;
    height: 0;
    padding-right: 3px;
    border-left: 6px solid black;
    border-top: 6px solid transparent;
    border-bottom: 6px solid transparent;
    border-right: 0 solid yellow;
    transition: transform .3s ease-in-out;
  }
  .node-icon-expanded {
    transform: rotate(90deg);
  }
</style>

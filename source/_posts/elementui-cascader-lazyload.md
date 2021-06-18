---
title: elementui-cascader-lazyload
date: 2020-09-14 15:39:16
tags: elementUI vue
categories: 
- web前端
---
el-cascader动态加载数据, 父子节点不关联单选的情况下, 直接点击radio图标不自动加载下一级的解决办法

## 业务背景及问题
  **业务背景**: 后端无法在初始化的时候返回树状结构的数据(数据是历史遗留问题), 只能根据节点id动态加载子节点数据
  **需求**: 可任意选择任意级的选项, 但必须是单选
## 演示地址
  [codepen地址](https://codepen.io/honorgo/pen/zYqGLey)
  **问题重现**: 先点击选项1前面的radio框, 下一级无法加载; 只有点击 选项1 的时候下一级才能正常显示;
## 解决
  **解决思路**: 选中radio框的时候, 同样执行类似点击 选项1 的js方法
  ```vue.js
  <template>
    <el-cascader :props="props" ref="demo" @change="change"></el-cascader>
  </template>
  <script>
    export default {
      data() {
        return {
          props: {
            lazy: true,
            checkStrictly: true,
            lazyLoad(node, resolve) {
              const { level } = node;
              setTimeout(() => {
                const nodes = Array.from({ length: level + 1 }).map((item) => ({
                  value: ++id,
                  label: `选项${id}`,
                  leaf: level >= 2
                }));
                resolve(nodes);
              }, 1000);
            }
          }
        }
      },
      methods: {
        change() {
          let node = this.$refs.demo.getCheckedNodes()[0];
          // 防止已加载数据重复加载; 叶子节点不加载下一级
          if (node.children && node.children.length == 0 && !node.isLeaf)
            this.$refs.demo.$refs.panel.lazyLoad(node);
        }
      }
    }
  </script>
  ```
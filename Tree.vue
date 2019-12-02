<template>
	<div class="Tree" :class="{isTop:isTop}">
        <div  class="node_row" :style="{
            height: clacHeight(item)*nodeHeight + 'px',
            transition:`height ${atf} ${attime}ms`,
        }" v-for="(item, key) in currentOptions" :key="key">
            <div class="node" :style="{
            paddingLeft:paddingLeft,
            height:nodeHeight+'px',
            lineHeight:nodeHeight+'px',
            }" :class="{
                node_parent:!!item[childrenField],
                node_child:!item[childrenField],
                node_top:(isTop && key === 0),
            }" @click="toggleClick(item, key,{
                item:item,
                key:key,
                keyMap:(keyMap)?`${keyMap},${key}`:`${key}`,
                node_parent:!!item[childrenField],
                node_child:!item[childrenField],
                node_top:(isTop && key === 0),
                node_open:!item.node_open
            })">
                <slot :node="item">{{item[showNameField]}}</slot>
            </div>
            <z-tree v-if="item[childrenField]"
                    :options="item[childrenField]"
                    :isTop="false"
                    :indentIndex="indentIndex+1"
                    :showNameField="showNameField"
                    :keyMap="(keyMap)?`${keyMap},${key}`:`${key}`"
                    :parentNode="item"
                    :atf="atf"
                    :attime="attime"
                    @parent="emitParent"
                    @child="emitChild"
                    @node="emitNode"
            >
                <template slot-scope="{node}">
                    <slot :node="node">{{node[showNameField]}}</slot>
                </template>
            </z-tree>
        </div>
	</div>
</template>

<script>
export default {
    name: "z-tree",
    props:{
        // 树结构数据
        options: {
            type: Array,
            default: Array
        },
        // 是否为顶级组件
        isTop: {
            type: Boolean,
            default: true
        },
        // 是否为顶级组件
        indent: {
            type: Number,
            default: 15
        },
        // 是否为顶级组件
        indentIndex: {
            type: Number,
            default: 0
        },
        // 是否为顶级组件
        showNameField: {
            type: String,
            default: "name"
        },
        // 子节点父级字段
        childrenField: {
            type: String,
            default: "children"
        },
        // 节点路径
        keyMap: {
            type: String,
            default: null
        },
        // 节点高度
        nodeHeight: {
            type: Number,
            default: 40
        },
        // 节点高度
        parentNode: {
            type: Object,
            default: Object
        },
        // 动画曲线
        atf: {
            type: String,
            default: "ease"
        },
        // 动画时间
        attime: {
            type: Number,
            default: 300
        }
    },
    data() {
        return {
            currentOptions:[],
        }
    },
    computed:{
        // 缩进计算
        paddingLeft() {
            return this.indent*this.indentIndex+"px";
        },
    },
    watch: {
        options() {
            this.initOptions();
        }
    },
    mounted() {
        this.initOptions();
    },
    methods: {
        // 获取节点路径的总个数，包含本身
        getNodeIndex(item){
            let index =  1;
            if(item[this.childrenField] && item.node_open){
                item[this.childrenField].forEach((e)=>{
                    index += this.clacHeight(e);
                });
            }
            return  index;
        },
        // 高度计算
        clacHeight(item){
            return this.getNodeIndex(item);
        },
        // 初始化选项
        initOptions() {
            // 拷贝数据
            this.currentOptions = this.options;
        },
        // 点击切换
        toggleClick(item, key, node) {
            item.node_open = !item.node_open;
            this.$set(this.currentOptions, key, item);
            if(node.node_parent){
                // 父节点切换
                this.emitParent(node);
            }else {
                // 子节点切换
                this.emitChild(node);
            }
            // 节点切换
            this.emitNode(node);
        },
        // 父节点切换
        emitParent(node){
            this.$emit('parent',node);
        },
        // 子节点切换
        emitChild(node){
            this.$emit('child',node);
        },
        // 节点切换
        emitNode(node){
            this.$emit('node',node);
        }
    },
}
</script>

<style scoped lang="less">
	.Tree {
        width: 500px;
        margin: auto;
        &.isTop{
            /*border:1px solid #e5e5e5;*/
            margin-top: 10px;
        }
        .nodeStyles(){
            cursor: pointer;
        }
        .node_row{
            overflow: hidden;
            .node{
                .nodeStyles();
                &.node_top{
                    margin-top: 0;
                }
            }
        }

	}
</style>

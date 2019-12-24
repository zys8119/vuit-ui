<template>
    <div class="HorizontalTreeList">
        <div class="HorizontalTree">
            <div class="HorizontalTreeItem header">
                <el-checkbox class="el-checkbox" :indeterminate="isIndeterminate" v-model="checkedAll"  @input="checkboxInputAll"></el-checkbox>
                <span class="HorizontalTreeItem-label">全选</span>
            </div>
            <div class="HorizontalTreeItem" :class="{check:check(item)}" v-for="item,key in options" :key="key">
                <el-checkbox class="el-checkbox" :indeterminate="indeterminateItem(item)"  :value="checkboxItem(checkbox.indexOf(item[valueKeyName]) > -1,item[valueKeyName])" @input="checkboxInput($event,item[valueKeyName],null,item)"></el-checkbox>
                <span class="HorizontalTreeItem-label" @click="TreeItemClick(item,key)">{{item.name}}</span>
                <i class="el-icon-arrow-right" v-if="item[childKeyName] && item[childKeyName].length > 0" @click="TreeItemClick(item,key)"></i>
            </div>
        </div>
        <horizontal-tree
            v-if="list"
            :options="list"
            :value="value"
            :childKeyName="childKeyName"
            :expand="currentExpand"
            :expandCopy="currentExpandCopy"
            :level="level+1"
            :isTop="false"
            :isExpandDefault="isExpandDefault"
            :valueKeyName="valueKeyName"
            :checks="checkbox"
            :parent="options[checkId]"
            @on-currentIsExpandDefault="onCurrentIsExpandDefault"
        ></horizontal-tree>
    </div>
</template>

<script>
    export default {
        name: "horizontal-tree",
        props:{
          options:{
              type:Array,
              default:Array
          },
          value:{
              type:Array,
              default: Array
          },
          childKeyName:{
              type:String,
              default: "list"
          },
          expand:{
              type:Array,
              default: Array
          },
          expandCopy:{
              type:Array,
              default: Array
          },
          level:{
              type:Number,
              default:0,
          },
          isTop:{
              type:Boolean,
              default:true,
          },
          isExpandDefault:{
              type:Boolean,
              default:false,
          },
          valueKeyName:{
              type:String,
              default:"id"
          },
          checks:{
              type:Array,
              default:Array
          },
          parent:{
              type:Object,
              default:undefined
          }
        },
        computed:{
          list(){
              this.currentCheckbox = [];
              this.isIndeterminate = false;
              let findObj = this.options[this.checkId];
              if(findObj && findObj[this.childKeyName] && findObj[this.childKeyName].length){
                  if(this.currentIsExpandDefault && this.currentExpand[this.level] !== null){
                      this.TreeItemClick(findObj,0,this.currentIsExpandDefault);
                  }
                  return findObj[this.childKeyName];
              }
          }
        },
        data(){
            return {
                checkedAll:false,
                checkId:0,
                currentExpand:[],
                currentExpandCopy:[],
                checkbox:[],
                currentCheckbox:[],
                currentIsExpandDefault:false,
                isIndeterminate:false,
            }
        },
        watch:{
            expand(){
                this.initExpand();
            },
            expandCopy(){
                this.initExpandCopy();
            },
            checks(){
                this.initChecks();
            },
            checkbox(value){
                this.$emit('check',value);
            },
            currentCheckbox(val){
                this.initCheckedAll(val);
            },
            list(){
                this.initCheckedAll(this.currentCheckbox);
            }
        },
        mounted(){
            this.initExpand();
            this.initExpandCopy();
            this.initChecks();
            this.initCheckedAll(this.currentCheckbox);
        },
        methods:{
            getChild(item, result){
                result = result || [];
                if(item[this.childKeyName] && item[this.childKeyName].length > 0){
                    item[this.childKeyName].forEach(e=>{
                        result.push(e[this.valueKeyName]);
                        this.getChild(e,result);
                    })
                };
                return result;
            },
            indeterminateItem(item, val){
                let resultBool = false;
                if(this.checkbox.indexOf(item[this.valueKeyName]) === -1){
                    this.getChild(item).forEach(e=>{
                        if(this.checkbox.indexOf(e) > -1){
                            resultBool = true;
                        }
                    });
                }
                return resultBool;
            },
            checkboxInputAll(val){
                this.options.forEach(item=>{
                    this.checkboxInput(val, item[this.valueKeyName], null, item);
                });
            },
            initCheckedAll(val){
                this.checkedAll = false;
                let itemArr = [];
                this.options.forEach(item=>{
                    try {
                        if(val.some(e=>e === item[this.valueKeyName])){
                            itemArr.push(item)
                        }
                    }catch (e) {}
                });
                if(itemArr.length === this.options.length){
                    this.checkedAll = true;
                }else {
                    this.checkedAll = false;
                }
                if(!this.checkedAll && val && val.length > 0){
                    this.isIndeterminate = true;
                }else {
                    this.isIndeterminate = false;
                }
            },
            checkboxItem(val,id){
              this.checkboxInput(val,id,"currentCheckbox");
              return val;
            },
            initChecks(){
                this.checkbox = this.checks;
            },
            checkboxInput(val,id, field, item){
                field = field || "checkbox";
                let index = this[field].indexOf(id);
                let existId = this.options.some(e=>e[this.valueKeyName] === id);
                if(existId && val && index === -1){
                    this[field].push(id);
                }
                if(existId && !val && index !== -1){
                    this[field].splice(index,1);
                };
                if(item && this.parent){
                    let isArr = true;
                    this.getChild(this.parent).forEach(e=>{
                        if(this[field].indexOf(e) === -1){
                            isArr = false;
                        }
                    });
                    if(isArr){
                        this[field].push(this.parent[this.valueKeyName]);
                    }else {
                        let index2 = this[field].indexOf(this.parent[this.valueKeyName]);
                        if(index2 !== -1){
                            this[field].splice(index2,1);
                        }
                    }
                }
            },
            onCurrentIsExpandDefault(val){
                this.currentIsExpandDefault = val;
            },
            initExpandCopy(){
                this.currentExpandCopy = this.expandCopy;
                if(this.isTop){
                    this.currentExpandCopy = JSON.parse(JSON.stringify(this.expand));
                }
                if(this.currentExpandCopy.length === 0){
                    this.currentIsExpandDefault = true;
                }else {
                    this.currentIsExpandDefault = false;
                }
            },
            initExpand(){
                this.currentExpand = this.expand;
                if(this.currentExpand[this.level]){
                    let checkId = this.options.findIndex(e=>e[this.valueKeyName] === this.currentExpand[this.level]);
                    if(checkId)this.checkId = checkId;
                }
            },
            check(item){
                let result = this.currentExpand.indexOf(item[this.valueKeyName]) > -1;
                return result;
            },
            TreeItemClick(item,key,bool){
                if(!bool){
                    this.currentIsExpandDefault = false;
                }
                this.checkId = key;
                this.$emit('on-currentIsExpandDefault',this.currentIsExpandDefault);
                if(this.currentExpand[this.level] !== item[this.valueKeyName]){
                    this.currentExpand.forEach((e,i)=>{
                        if(i > this.level){
                            this.$set(this.currentExpand,i,null);
                        }
                    });
                    this.$set(this.currentExpand,this.level,item[this.valueKeyName]);
                }
            }
        },
    }
</script>

<style lang="less">
    .HorizontalTreeList{
        overflow: hidden;
        .HorizontalTree {
            border-right: 1px solid #EEEEEE;
            width: 274px;
            float: left;
            .HorizontalTreeItem{
                overflow: hidden;
                @h:46px;
                line-height: @h;
                padding: 0 18px;
                position: relative;
                cursor: pointer;
                .el-checkbox{
                    float: left;
                    margin-right: 15px;
                }
                .el-icon-arrow-right{
                    position: absolute;
                    top: 0;
                    right:0;
                    height: @h;
                    line-height: @h;
                    color: #C2C6CC;
                    padding-right: 18px;
                }
                .HorizontalTreeItem-label{
                    float: left;
                    color: #262A30;
                    font-size: 14px;
                    width: 274px - 18px*2 - 30px;
                }
                &:hover{
                    background-color: #F6F6F6;
                }
                &.header{
                    border-bottom: 1px solid #EEEEEE;
                    .HorizontalTreeItem-label{
                        color: #232A32;
                    }
                    &:hover{
                        background-color: transparent;
                    }
                }
                &.check{
                    .HorizontalTreeItem-label{
                        color: #09AF39;
                        font-weight: bold;
                    }
                }
            }
        }
    }
</style>

<template>
    <div class="modify">
        <el-form :model="form" :rules="rules" ref="form" label-width="100px" :status-icon="true">
            <el-form-item label="菜单类型">
                <el-radio-group v-model="form.menuType" @change = "menuTypeChange">
                    <template v-for="item in menuType">
                        <el-radio :label="item.itemValue">{{item.itemText}}</el-radio>
                    </template>
                </el-radio-group>
            </el-form-item>
            <template v-if="form.menuType === '2'">
                <el-form-item label="按钮/权限" prop="name">
                    <el-input v-model="form.name" placeholder="请输入按钮名称" clearable></el-input>
                </el-form-item>
                <el-form-item label="父级菜单" prop="parentId" :rules = "{required : true, message : '必填'}">
                    <select-tree
                            ref = "selectTree"
                            :value = "form.parentId"
                            :props = "tree.props"
                            :options = "menus"
                            @getValue = "getValue($event)"
                    >

                    </select-tree>
                </el-form-item>
                <el-form-item label="菜单路径" prop="url">
                    <el-input v-model="form.url" placeholder="请输入菜单路径" clearable></el-input>
                </el-form-item>
                <el-form-item label="授权标识">
                    <el-input v-model="form.perms" placeholder="多个用逗号分隔, 如: user:list,user:create" clearable></el-input>
                </el-form-item>
                <el-form-item label="授权策略">
                    <el-radio-group v-model="form.permsType">
                        <template v-for="item in permissionType">
                            <el-radio :label="item.itemValue">{{item.itemText}}</el-radio>
                        </template>
                    </el-radio-group>
                </el-form-item>
                <el-form-item label="状态">
                    <el-radio-group v-model="form.status">
                        <template v-for="item in validStatus">
                            <el-radio :label="item.itemValue">{{item.itemText}}</el-radio>
                        </template>
                    </el-radio-group>
                </el-form-item>
            </template>
            <template v-else>
                <el-form-item label="菜单名称" prop="name">
                    <el-input v-model="form.name" placeholder="请输入菜单名称" clearable></el-input>
                </el-form-item>
                <template v-if="form.menuType === '1'">
                    <el-form-item label="父级菜单" prop="parentId" :rules = "{required : true, message : '必填'}">
                        <select-tree
                            ref = "selectTree"
                            :value = "form.parentId"
                            :props = "tree.props"
                            :options = "menus"
                            @getValue = "getValue($event)"
                        >

                        </select-tree>
                    </el-form-item>
                </template>
                <el-form-item label="菜单路径" v-if="form.menuType === '2'">
                    <el-input v-model="form.url" placeholder="请输入菜单路径" clearable></el-input>
                </el-form-item>
                <el-form-item
                        v-else
                        label="菜单路径"
                        prop="url"
                        :rules="[{required : true, message : '必填',trigger :'change'}]">
                    <el-input v-model="form.url" placeholder="请输入菜单路径" clearable></el-input>
                </el-form-item>
                <el-form-item label="前端组件" prop="component">
                    <el-input v-model="form.component" placeholder="请输入菜单组件" clearable></el-input>
                </el-form-item>
                <template v-if="form.menuType === '0'">
                    <el-form-item label="默认跳转地址">
                        <el-input v-model="form.redirect" placeholder="请输入路由参数 redirect" clearable></el-input>
                    </el-form-item>
                </template>
                <el-form-item label="菜单图标">
                    <el-input v-model="form.icon" placeholder="请输入菜单图标" clearable>
                        <i slot="suffix" class="el-input__icon el-icon-setting cursor-pointer" @click="checkIcons"></i>
                    </el-input>
                </el-form-item>
                <el-form-item label="排序">
                    <el-input-number v-model="form.sortNo" :min="0" class="w-full"></el-input-number>
                </el-form-item>
                <el-form-item label="是否路由">
                    <el-radio-group v-model = "form.route">
                        <el-radio-button :label = "true">是</el-radio-button>
                        <el-radio-button :label = "false">否</el-radio-button>
                    </el-radio-group>
                </el-form-item>
                <el-form-item label="隐藏路由">
                    <el-radio-group v-model = "form.hidden">
                        <el-radio-button :label = "true">是</el-radio-button>
                        <el-radio-button :label = "false">否</el-radio-button>
                    </el-radio-group>
                </el-form-item>
                <el-form-item label="是否缓存路由">
                    <el-radio-group v-model = "form.keepAlive">
                        <el-radio-button :label = "true">是</el-radio-button>
                        <el-radio-button :label = "false">否</el-radio-button>
                    </el-radio-group>
                </el-form-item>
                <el-form-item label="聚合路由">
                    <el-radio-group v-model = "form.alwaysShow">
                        <el-radio-button :label = "true">是</el-radio-button>
                        <el-radio-button :label = "false">否</el-radio-button>
                    </el-radio-group>
                </el-form-item>
                <el-form-item label="打开方式">
                    <el-radio-group v-model = "form.internalOrExternal">
                        <el-radio-button :label = "true">外部</el-radio-button>
                        <el-radio-button :label = "false">内部</el-radio-button>
                    </el-radio-group>
                </el-form-item>
            </template>
        </el-form>
        <drag-dialog :drag-dialog="dialog" @confirm="iconChooseConfirm">
            <icons ref="icons" :icons-checked="icons"></icons>
        </drag-dialog>
    </div>
</template>

<script>
    import {mapState, mapActions} from 'vuex'
    import {phoneCheck} from '@/utils/modules/validate'
    import {http, apiList, constant, sweetAlert} from '@/utils'
    import Icons from './icon/Icons'
    import DragDialog from '@/components/dragDialog'
    import SelectTree from '@/components/treeSelect/TreeSelect'
    import {isEmpty} from '30-seconds-of-code'

    let customParams = {}

    export default {
        name: "Modify",
        components: {
            Icons,
            DragDialog,
            SelectTree
        },
        props: {
            data: {
                type: Object
            }
        },
        data() {
            return {
                tree : {
                    value : '',
                    props : {
                        value : 'id',
                        label : 'name',
                        children : 'children'
                    }
                },
                form: {
                    menuType: '0',                  //菜单类型
                    name: '',                      //菜单名称
                    parentId: '',                  //父级菜单
                    url: '',                       //菜单路径
                    component: '',                 //前端组件
                    redirect: '',                  //默认跳转地址
                    icon: '',                      //菜单图标
                    sortNo: '',                    //排序
                    route: true,                    //是否路由
                    hidden: false,                    //隐藏路由
                    keepAlive: false,              //是否缓存路由
                    alwaysShow: false,              //聚合路由
                    permsType: '1',                  //授权策略
                    perms: '',                      //授权标识
                    status: '1',                      //状态
                    internalOrExternal : false,        //打开方式
                },
                rules: {
                    name: [
                        {required: true, message: '必填', trigger: 'change'}
                    ],
                    // url : [
                    //     {required : true, message : '必填',trigger :'change'}
                    // ],
                    component: [
                        {required: true, message: '必填', trigger: 'change'}
                    ]
                },
                dialog: {
                    width: '50',
                    height: '60',
                    name: 'iconSelect',
                    title: '图标选择',
                    showFooter: true,
                },
                icons: {}
            }
        },
        computed: {
            ...mapState({
                menus: ({system}) => system.menus,
                menuType: ({dict}) => dict.menuType,
                permissionType: ({dict}) => dict.permissionType,
                validStatus: ({dict}) => dict.validStatus
            })
        },
        watch: {
            data: {
                handler(props) {
                    if (!isEmpty(props)) {
                        let {menuType} = props
                        this.form = {
                            ...this.form,
                            ...props,
                            menuType: menuType.toString()
                        }
                    }else{
                        this.$nextTick(()=>{
                            this.resetFields()
                            this.$refs.form.resetFields()
                        })
                    }
                },
                immediate: true
            },
        },
        methods: {
            ...mapActions({
                getPermissionList: 'GET_PERMISSION_LIST'
            }),
            resetFields(){
                this.form = {
                    menuType: '0',                  //菜单类型
                    name: '',                      //菜单名称
                    parentId: '',                  //父级菜单
                    url: '',                       //菜单路径
                    component: '',                 //前端组件
                    redirect: '',                  //默认跳转地址
                    icon: '',                      //菜单图标
                    sortNo: '',                    //排序
                    route: true,                    //是否路由
                    hidden: false,                    //隐藏路由
                    keepAlive: false,              //是否缓存路由
                    alwaysShow: false,              //聚合路由
                    permsType: '1',                  //授权策略
                    perms: '',                      //授权标识
                    status: '1',                      //状态
                    internalOrExternal : false,        //打开方式
                }
            },
            getValue(value){
                this.form = {
                    ...this.form,
                    parentId : value
                }
                this.tree = {
                    ...this.tree,
                    value
                }
            },
            menuTypeChange(){
               this.$refs.form.clearValidate()
            },
            checkIcons() {
                let {name} = this.dialog
                let {icon} = this.form
                if (icon) {
                    this.icons = {
                        ...this.icons,
                        name: icon,
                    }
                } else {
                    this.icons = {
                        ...this.icons,
                        name: '',
                        paneName: 'direct'
                    }
                }
                this.$modal.show(name)
            },
            iconChooseConfirm() {
                let {checked: {icon}, pane: {name: paneName}} = this.$refs.icons
                let {name} = this.dialog
                if (icon) {
                    this.form = {
                        ...this.form,
                        icon
                    }
                    this.icons = {
                        ...this.icons,
                        name : icon,
                        paneName
                    }
                    this.$modal.hide(name)
                } else {
                    sweetAlert.errorWithTimer('请选择图标')
                }
            },
            async saveData() {
                let {id} = this.data
                let params = {
                    ...this.form,
                }
                let res
                if (id) {                     //编辑
                    res = await http.put(apiList.sys_menu_edit, params)
                } else {
                    res = await http.post(apiList.sys_menu_add, params)
                }
                let {success, message} = res
                if (success) {
                    this.getPermissionList()                        //刷新菜单
                    sweetAlert.successWithTimer(message)
                    this.$emit('successClose')
                } else {
                    sweetAlert.error(message)
                }
            }
        },
        mounted() {

        }
    }
</script>

<style scoped>

</style>
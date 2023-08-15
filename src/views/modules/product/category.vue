<template>
    <div>
        <el-tree :data="menu" :props="defaultProps" :expand-on-click-node="false" show-checkbox node-key="catId"
            :default-expanded-keys="expandedKeys" draggable="true">
            <span class="custom-tree-node" slot-scope="{ node, data }">
                <span>{{ node.label }}</span>
                <span>
                    <el-button v-if="node.level <= 2" type="text" size="mini" @click="() => append(data)">
                        Append
                    </el-button>
                    <el-button v-if="node.childNodes.length == 0" type="text" size="mini" @click="() => remove(node, data)">
                        Delete
                    </el-button>
                    <el-button type="text" size="mini" @click="() => edit(data)">
                        Edit
                    </el-button>
                </span>
            </span>
        </el-tree>

        <el-dialog :title="title" :visible.sync="dialogVisible" width="30%" :close-on-click-modal="false">
            <el-form :model="category">
                <el-form-item label="category name">
                    <el-input v-model="category.name" autocomplete="off"></el-input>
                </el-form-item>
                <el-form-item label="category icon" v-if="dialogType == 'edit'">
                    <el-input v-model="category.icon" autocomplete="off"></el-input>
                </el-form-item>
                <el-form-item label="product unit" v-if="dialogType == 'edit'">
                    <el-input v-model="category.productUnit" autocomplete="off"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="dialogVisible = false">Cancel</el-button>
                <el-button type="primary" @click="submitData">Confirm</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
//这里可以导入其他文件（比如：组件，工具 js，第三方插件 js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》';

export default {
    //import 引入的组件需要注入到对象中才能使用
    components: {},
    props: {},
    data() {
        return {
            title: "",
            category: {
                name: "",
                parentCid: 0,
                catLevel: 0,
                showStatus: 1,
                sort: 0,
                catId: null,
                icon: "",
                productUnit: "",
            },
            dialogVisible: false,
            menu: [],
            defaultProps: {
                children: 'children',
                label: 'name'
            },
            expandedKeys: [],
            dialogType: ""
        };
    },
    methods: {

        getMenus() {
            this.$http({
                url: this.$http.adornUrl('/product/category/list/tree'),
                method: 'get',
            }).then(({ data }) => {
                console.log("成功获取到菜单数据...", data.data)
                this.menu = data.data
            })
        },

        append(data) {
            this.dialogVisible = true;
            this.category.parentCid = data.catId;
            this.category.catLevel = data.catLevel * 1 + 1;
            this.dialogType = "append";
            this.title = "add category";

            this.category.name = null;
            this.category.catId = null;
            this.category.icon = null;
            this.category.productUnit = null;
            this.category.sort = 0;
            this.category.showStatus = 1;
            console.log(this.category)
        },

        remove(node, data) {
            let ids = [data.catId]
            this.$confirm(`This will delete the menu: ${data.name}. Continue?`, 'Warning', {
                confirmButtonText: 'OK',
                cancelButtonText: 'Cancel',
                type: 'warning'
            }).then(() => {
                this.$http({
                    url: this.$http.adornUrl('/product/category/delete'),
                    method: 'post',
                    data: this.$http.adornData(ids, false)
                }).then(({ data }) => {
                    this.$message({
                        message: 'Menu delete successfully.',
                        type: 'success'
                    });
                    this.getMenus()
                    this.expandedKeys = [node.parent.data.catId]
                })
            }).catch(() => {

            });
        },

        addCategory() {
            this.$http({
                url: this.$http.adornUrl('/product/category/save'),
                method: 'post',
                data: this.$http.adornData(this.category, false)
            }).then(({ data }) => {
                this.$message({
                    message: 'Menu append successfully.',
                    type: 'success'
                });
                this.dialogVisible = false;
                this.getMenus();
                this.expandedKeys = [this.category.parentCid];
            });
        },

        edit(data) {
            this.dialogVisible = true;
            this.category.name = data.name;
            this.category.catId = data.catId;
            this.dialogType = "edit";
            this.title = "edit category"

            this.$http({
                url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
                method: 'post',
                data: this.$http.adornData(data, false)
            }).then(({ data }) => { 
                console.log('edit data:', data)
                this.category.name = data.category.name;
                this.category.catId = data.category.catId;
                this.category.icon = data.category.icon;
                this.category.productUnit = data.category.productUnit;
                this.category.catLevel = data.category.catLevel;
                this.category.sort = data.category.sort;
                this.category.showStatus = data.category.showStatus;
            });
        },

        submitData() {
            if (this.dialogType == "append") {
                this.addCategory();
            } else if (this.dialogType == "edit") {
                this.editCategory()
            }
        },

        editCategory() {
            let { catId, name, icon, productUnit } = this.category
            this.$http({
                url: this.$http.adornUrl('/product/category/update'),
                method: 'post',
                data: this.$http.adornData({ catId, name, icon, productUnit }, false)
            }).then(({ data }) => {
                this.$message({
                    message: 'Menu edit successfully.',
                    type: 'success'
                });
                this.dialogVisible = false;
                this.getMenus();
                this.expandedKeys = [this.category.parentCid];
            });
        }
    },
    //计算属性 类似于 data 概念
    computed: {},
    //监控 data 中的数据变化
    watch: {},
    //生命周期 - 创建完成（可以访问当前 this 实例）
    created() {
        this.getMenus();
    },
    //生命周期 - 挂载完成（可以访问 DOM 元素）
    mounted() {

    },
    beforeCreate() { }, //生命周期 - 创建之前
    beforeMount() { }, //生命周期 - 挂载之前
    beforeUpdate() { }, //生命周期 - 更新之前
    updated() { }, //生命周期 - 更新之后
    beforeDestroy() { }, //生命周期 - 销毁之前
    destroyed() { }, //生命周期 - 销毁完成
    activated() { }, //如果页面有 keep-alive 缓存功能，这个函数会触发
}
</script>
<style scoped></style>
<template>
  <div>
    <ta-row>
      <ta-col :span="row.col.fullSpan">
        <span class="title">组织权限</span>
        <!--<div class="filter">-->
        <!--<ta-input placeholder="请输入资源名称" v-model="adminFilterText"/>-->

        <!--</div>-->
        <div class="modalTreeStyle">
          <ta-e-tree :data="treeData"
                     :load="loadData"
                     show-checkbox
                     highlight-current
                     check-strictly
                     node-key="orgId"
                     :props="defaultProps"
                     @check-change="handleCheckNodeChange"
                     ref="tree" lazy>
            <span class="custom-tree-node" slot-scope="{ node, data }">
              {{data.orgName}}
              <span v-if="data.isAuthority === '0'">
                <span style="float: right;color: #ccc;font-size: 12px;cursor: not-allowed">无操作权限</span>
              </span>
              <span v-if="data.disabled">
                <span style="float: right;color: #ccc;font-size: 12px">该组织已选择</span>
              </span>

            </span>
          </ta-e-tree>
        </div>
      </ta-col>
    </ta-row>
    <ta-row>
      <ta-col>
        <div class="ant-modal-footer" style="text-align: center;">
          <ta-button @click="$emit('modalCancel')">返回</ta-button>
          <ta-button type="primary" @click="fnSaveOrUpdate">保存</ta-button>
        </div>
      </ta-col>
    </ta-row>
  </div>

</template>
<script>
import $api from '../../api/index'

export default {
  name: 'adminAuthorityMg',
  props: ['item'], // 布局需要从上面获取,内容需要从上面获取,属性也要从上面获取
  data () {
    return {
      roleId: '', // 角色id
      adminFilterText: '', // 组织过滤框
      treeData: [], // 组织数据
      defaultProps: {
        label: 'orgName',
        isLeaf: 'isLeaf'
      },
      row: {
        col: {
          span: 12,
          fullSpan: 24
        }
      }
    }
  },
  watch: {
    // 过滤组织树
    adminFilterText (val) {
      console.log('val', val)
    }
  },
  created() {
    this.roleId = this.item.roleId
  },
  methods: {
    // 处理节点点击事件
    handleCheckNodeChange (data, checked, indeterminate) {
      // 如果已经选中,则移除所有的选项,然后进行选中,如果取消选择,
      let node = this.$refs.tree.getCheckedKeys()
      if (checked) { // 如果选中则进行提示
        if (data.isAuthority === '0') {
          this.$message.warning('您没有该组织的操作权限')
          this.$refs.tree.setChecked(data, false)
          return
        }
        if (node.length >= 2) {
          for (let i = 0; i < node.length; i++) {
            if (node[i] !== data.orgId) {
              this.$refs.tree.setChecked(node[i], false, false)
            }
          }
        }
      }
    },
    // 加载数据
    loadData (node, resolve) {
      if (node.level == 0) {
        let data = {
          roleId: this.roleId
        }
        // 请求第一级
        $api.selectPermissionOrgScope(this, data, dd => {
          // 数据成功后默认返回的数据
          return resolve(dd.data.adminOrgScope)
        })
      }
      if (node.level >= 1) {
        let orgId = node.data.orgId
        let data = {
          orgId: orgId,
          roleId: this.roleId
        }
        $api.selectPermissionOrgScope(this, data, dd => {
          // 数据成功后默认返回的数据
          let res = dd.data.adminOrgScope
          if (res[0].children) {
            let orgs = res[0].children
            if (node.data.disabled) {
              let disableOrgs = orgs.map(v => {
                v.disabled = true
                return v
              })
              return resolve(disableOrgs)
            } else {
              return resolve(orgs)
            }
          }
        })
      }
    },

    // 查询所有组织授权权限
    fnSelectPermissionOrgScope (roleId) {
      let self = this
      let data = {
        roleId: roleId
      }
      $api.selectPermissionOrgScope(this, data, (data) => {
        // 组织全部数据
        this.treeData = data.data.adminOrgScope
      })
    },
    // 新增或者更新
    fnSaveOrUpdate () {
      // 保存组织权限数据
      let nodes = this.$refs.tree.getCheckedNodes()
      if (nodes.length < 1) {
        this.$message.warning('请选择组织', 2.5)
        return
      }
      let node = nodes[0]
      let data = {
        roleId: this.roleId,
        orgId: node.orgId,
        idPath: node.idPath
      }
      $api.addOrgPermission(this, data, (data) => {
        this.$message.success('保存数据成功')
        // 通知关闭
        this.$emit('modalCancel')
      })
    }
  }
}
</script>
<style type="text/scss" scoped>
  .title {
    font-weight: bold;
  }

  .filter {
    display: flex;
    margin-top: 10px;
  }

  .modalTreeStyle {
    height: 380px;
    margin: 10px 0;
    /*overflow: scroll;*/
    overflow-y: scroll;
    border: 1px dashed lightgray;
  }

  .custom-tree-node {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: space-between;
    font-size: 14px;
    padding-right: 8px;
  }
</style>

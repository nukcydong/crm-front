<template>
  <div>
    <ta-user-select title="新增组织人员"
                    width="800px"
                    v-model="userListInfo"
                    :is-show="isShow"
                    :is-multi="true"
                    with-tag
                    :load="loadOrgTreeNode"
                    :user-list-data="relationUserListData"
                    :user-select-data="userSelectData"
                    :default-user-list="userListResult"
                    :props="userDefaultProps"
                    @search="fnSearch"
                    @close="fnCloseUserModal"
                    @queryUserList="fnQueryUserList"
                    @getUserListResult="fnGetUserListResult"
                    id="relationUserTag">
    </ta-user-select>
  </div>
</template>

<script>
import $api from '../api/index'

export default {
  name: 'customOrgUserSelectTag',
  props: {
    visible: {
      type: Boolean
    }
  },
  data () {
    return {
      // userSelectId: 'relationUser',//默认人员选择框id
      isShow: false, // 是否显示
      userListResult: [], // 选择到的用户信息
      relationUserListData: [], // 点击左侧组织查询到的用户信息
      userSelectData: [], // 组件搜索的人员数据
      includeChild: false,
      userDefaultProps: {
        treeNodeKey: 'orgId',
        treeLabel: 'label',
        treeChildren: 'children',
        treeIsLeaf: 'isLeaf',
        listKey: 'userId', // 每条用户信息的唯一标识（默认：userId）
        listTitle: 'name', // 用户信息第一行显示内容
        listSubTitle: 'mobile', // 用户信息第二行显示内容
        listDescription: 'namePath' // 用户信息鼠标移上去显示描述信息
      },
      userListInfo: []
    }
  },
  watch: {
    visible (isShow) {
      if (isShow) {
        this.isShow = isShow
      }
    },
    getUserIds (val) {
      this.$emit('input', val)
    }
  },
  computed: {
    getUserIds () {
      let result = []
      this.userListInfo.map((item) => {
        result.push(item.userId)
      })
      return result.join(',')
    }
  },
  methods: {
    // 加载左侧组织树
    loadOrgTreeNode (node, resolve) {
      if (node.level === 0) {
        $api.queryAllTaOrg(this, null, data => {
          // 数据成功后默认返回的数据
          let orgVos = data.data.orgTreeData
          if (orgVos[0].children && (orgVos[0].children instanceof Array) && orgVos[0].children.length > 0) {
            let dd = orgVos[0].children.map(v => {
              let obj = v
              if (v.childNum > 0) {
                obj.children = []
              }
            })
            orgVos[0].children = dd
          }
          return resolve(orgVos)
        })
      }
      if (node.level >= 1) {
        let orgId = node.data.orgId
        let isLeaf = node.data.isLeaf
        let data = {
          orgId: orgId
        }
        $api.queryAllTaOrg(this, data, data => {
          // 数据成功后默认返回的数据
          let dd = data.data.orgTreeData
          if (isLeaf) {
            dd = dd.map(v => {
              let obj = v
              obj.children = []
            })
          }
          return resolve(dd)
        })
      }
    },
    // 人员选择框输入查询
    fnSearch (val) {
      let data = {
        name: val,
        loginId: val,
        idCardNo: val,
        showChildUser: true,
        searchType: 'normal'
      }
      $api.queryOrgUser(this, data, (dd) => {
        this.userSelectData = dd.data.pageBean.list
      })
    },
    // 关闭人员选择模态框
    fnCloseUserModal () {
      this.isShow = false
      // 将选中的数据进行清空
      this.userListResult = []
      this.relationUserListData = []
      this.userSelectData = []
    },
    fnQueryUserList (orgId, includeChildren, pageNum, searchType, searchParam) {
      this.includeChild = includeChildren
      // 查询用户信息
      let data = {
        orgId: orgId,
        showChildUser: includeChildren ? '1' : '0',
        pageNumber: pageNum,
        pageSize: 10
      }
      if (searchType) {
        data[searchType] = searchParam
      }
      let self = this
      $api.queryBatchUserByOrgId(self, data, (data) => {
        self.relationUserListData = data.data.userList.list
      })
    },
    // 确认选择的人员信息
    fnGetUserListResult (data) {
      this.userListResult = data
    }

  }
}
</script>

<style scoped type="text/scss" lang="scss">

</style>

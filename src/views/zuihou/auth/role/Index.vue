<template>
  <div class="app-container">
    <div class="filter-container">
      <el-input
        v-model="queryParams.code"
        :placeholder="$t(&quot;table.role.code&quot;)"
        class="filter-item search-item"
      />
      <el-input
        v-model="queryParams.name"
        :placeholder="$t(&quot;table.role.name&quot;)"
        class="filter-item search-item"
      />
      <el-date-picker
        v-model="queryParams.timeRange"
        :range-separator="null"
        start-placeholder="开始日期"
        end-placeholder="结束日期"
        value-format="yyyy-MM-dd HH:mm:ss"
        format="yyyy-MM-dd HH:mm:ss"
        class="filter-item search-item date-range-item"
        type="daterange"
      />
      <el-button
        class="filter-item"
        type="primary"
        plain
        @click="search"
      >
        {{ $t('table.search') }}
      </el-button>
      <el-button
        class="filter-item"
        type="warning"
        plain
        @click="reset"
      >
        {{ $t('table.reset') }}
      </el-button>
      <el-dropdown
        v-has-any-permission="[&quot;role:add&quot;,&quot;role:delete&quot;,&quot;role:export&quot;]"
        trigger="click"
        class="filter-item"
      >
        <el-button>
          {{ $t('table.more') }}
          <i class="el-icon-arrow-down el-icon--right" />
        </el-button>
        <el-dropdown-menu slot="dropdown">
          <el-dropdown-item
            v-has-permission="[&quot;role:add&quot;]"
            @click.native="add"
          >
            {{ $t('table.add') }}
          </el-dropdown-item>
          <el-dropdown-item
            v-has-permission="[&quot;role:delete&quot;]"
            @click.native="batchDelete"
          >
            {{ $t('table.delete') }}
          </el-dropdown-item>
          <el-dropdown-item
            v-has-permission="[&quot;role:export&quot;]"
            @click.native="exportExcel"
          >
            {{ $t('table.export') }}
          </el-dropdown-item>
        </el-dropdown-menu>
      </el-dropdown>
    </div>

    <el-table
      ref="table"
      :key="tableKey"
      v-loading="loading"
      :data="tableData.records"
      border
      fit
      style="width: 100%;"
      @selection-change="onSelectChange"
      @sort-change="sortChange"
    >
      <el-table-column
        type="selection"
        align="center"
        width="40px"
      />
      <el-table-column
        :label="$t(&quot;table.role.code&quot;)"
        prop="code"
        align="center"
        width="200px"
      >
        <template slot-scope="scope">
          <span>{{ scope.row.code }}</span>
        </template>
      </el-table-column>
      <el-table-column
        :label="$t(&quot;table.role.name&quot;)"
        prop="name"
        :show-overflow-tooltip="true"
        align="center"
      >
        <template slot-scope="scope">
          <span>{{ scope.row.name }}</span>
        </template>
      </el-table-column>
      <el-table-column
        :label="$t(&quot;table.role.describe&quot;)"
        prop="describe"
        :show-overflow-tooltip="true"
        align="center"
      >
        <template slot-scope="scope">
          <span>{{ scope.row.describe }}</span>
        </template>
      </el-table-column>
      <el-table-column
        :label="$t(&quot;table.role.dsType&quot;)"
        align="center"
        width="100px"
      >
        <template slot-scope="scope">
          <span>{{ scope.row.dsType.desc }}</span>
        </template>
      </el-table-column>
      <el-table-column
        :label="$t(&quot;table.role.readonly&quot;)"
        align="center"
        width="80px"
      >
        <template slot-scope="scope">
          <span>{{ scope.row.readonly ? '是' : '否' }}</span>
        </template>
      </el-table-column>
      <el-table-column
        :label="$t(&quot;table.role.status&quot;)"
        :filters="[{ text: $t(&quot;common.status.valid&quot;), value: true }, { text: $t(&quot;common.status.invalid&quot;), value: false }]"
        :filter-method="filterStatus"
        class-name="status-col"
        width="70px"
      >
        <template slot-scope="{row}">
          <el-tag
            :type="row.status | statusFilter"
          >
            {{ row.status ? $t('common.status.valid') : $t('common.status.invalid') }}
          </el-tag>
        </template>
      </el-table-column>
      <el-table-column
        :label="$t(&quot;table.createTime&quot;)"
        prop="createTime"
        align="center"
        width="160px"
        sortable="custom"
      >
        <template slot-scope="scope">
          <span>{{ scope.row.createTime }}</span>
        </template>
      </el-table-column>
      <el-table-column
        :label="$t(&quot;table.operation&quot;)"
        align="center"
        width="100px"
        class-name="small-padding fixed-width"
      >
        <template slot-scope="{row}">
          <i
            v-hasPermission="[&quot;role:update&quot;]"
            class="el-icon-edit table-operation"
            style="color: #2db7f5;"
            @click="edit(row)"
          />
          <el-dropdown
            v-has-any-permission="[&quot;role:update&quot;,&quot;role:delete&quot;,&quot;role:auth&quot;,&quot;role:config&quot;]"
          >
            <span class="el-dropdown-link">
              {{ $t('table.more') }}
              <i class="el-icon-arrow-down el-icon--right" />
            </span>
            <el-dropdown-menu slot="dropdown">
              <el-dropdown-item
                v-hasPermission="[&quot;role:delete&quot;]"
                icon="el-icon-delete"
                @click.native="singleDelete(row)"
              >
                删除
              </el-dropdown-item>
              <el-dropdown-item
                v-hasPermission="[&quot;role:auth&quot;]"
                icon="el-icon-user"
                @click.native="authUser(row)"
              >
                授权
              </el-dropdown-item>
              <el-dropdown-item
                v-hasPermission="[&quot;role:config&quot;]"
                icon="el-icon-setting"
                @click.native="authResource(row)"
              >
                配置
              </el-dropdown-item>
            </el-dropdown-menu>
          </el-dropdown>

          <el-link
            v-has-no-permission="[&quot;role:update&quot;,&quot;role:delete&quot;,&quot;role:auth&quot;,&quot;role:config&quot;]"
            class="no-perm"
          >
            {{ $t('tips.noPermission') }}
          </el-link>
        </template>
      </el-table-column>
    </el-table>
    <pagination
      v-show="tableData.total>0"
      :total="Number(tableData.total)"
      :page.sync="pagination.current"
      :limit.sync="pagination.size"
      @pagination="fetch"
    />
    <role-edit
      ref="edit"
      :dialog-visible="dialog.isVisible"
      :type="dialog.type"
      @success="editSuccess"
      @close="editClose"
    />
    <user-role
      ref="userRole"
      :dialog-visible="userRoleDialog.isVisible"
      @success="userRoleSuccess"
      @close="userRoleClose"
    />
    <role-authority
      ref="roleAuthority"
      :dialog-visible="roleAuthorityDialog.isVisible"
      @success="roleAuthoritySuccess"
      @close="roleAuthorityClose"
    />
  </div>
</template>

<script>
import Pagination from '@/components/Pagination'
import RoleEdit from './Edit'
import UserRole from './UserRole'
import RoleAuthority from './RoleAuthority'
import roleApi from '@/api/Role.js'

export default {
  name: 'RoleManage',
  components: { Pagination, RoleEdit, UserRole, RoleAuthority },
  filters: {
    statusFilter (status) {
      const map = {
        false: 'danger',
        true: 'success'
      }
      return map[status] || 'success'
    }
  },
  data () {
    return {
      dialog: {
        isVisible: false,
        type: 'add'
      },
      userRoleDialog: {
        isVisible: false
      },
      roleAuthorityDialog: {
        isVisible: false
      },
      tableKey: 0,
      // total: 0,
      queryParams: {},
      sort: {},
      selection: [],
      // 以下已修改
      loading: false,
      tableData: {
        total: 0
      },
      pagination: {
        size: 10,
        current: 1
      }
    }
  },
  computed: {

  },
  mounted () {
    this.fetch()
  },
  methods: {
    filterStatus (value, row) {
      return row.status === value
    },
    editClose () {
      this.dialog.isVisible = false
    },
    userRoleClose () {
      this.userRoleDialog.isVisible = false
    },
    roleAuthorityClose () {
      this.roleAuthorityDialog.isVisible = false
    },
    editSuccess () {
      this.search()
    },
    userRoleSuccess () {
      this.search()
    },
    roleAuthoritySuccess () {
      this.search()
    },
    onSelectChange (selection) {
      this.selection = selection
    },
    search () {
      this.fetch({
        ...this.queryParams,
        ...this.sort
      })
    },
    reset () {
      this.queryParams = {}
      this.sort = {}
      this.$refs.table.clearSort()
      this.$refs.table.clearFilter()
      this.search()
    },
    exportExcel () {
      this.$message({
        message: '待完善',
        type: 'warning'
      })
    },
    singleDelete (row) {
      this.$refs.table.toggleRowSelection(row, true)
      this.batchDelete()
    },
    batchDelete () {
      if (!this.selection.length) {
        this.$message({
          message: this.$t('tips.noDataSelected'),
          type: 'warning'
        })
        return
      }
      this.$confirm(this.$t('tips.confirmDelete'), this.$t('common.tips'), {
        confirmButtonText: this.$t('common.confirm'),
        cancelButtonText: this.$t('common.cancel'),
        type: 'warning'
      }).then(() => {
        const ids = []
        this.selection.forEach((u) => {
          ids.push(u.id)
        })
        this.delete(ids)
      }).catch(() => {
        this.clearSelections()
      })
    },
    clearSelections () {
      this.$refs.table.clearSelection()
    },
    delete (ids) {
      roleApi.delete({ ids: ids })
        .then((response) => {
          const res = response.data
          if (res.isSuccess) {
            this.$message({
              message: this.$t('tips.deleteSuccess'),
              type: 'success'
            })
          }
          this.search()
        })
    },
    add () {
      this.dialog.type = 'add'
      this.dialog.isVisible = true
      this.$refs.edit.setRole(false)
    },
    edit (row) {
      this.$refs.edit.setRole(row)
      this.dialog.type = 'edit'
      this.dialog.isVisible = true
    },
    fetch (params = {}) {
      this.loading = true
      params.size = this.pagination.size
      params.current = this.pagination.current
      if (this.queryParams.timeRange) {
        params.startCreateTime = this.queryParams.timeRange[0]
        params.endCreateTime = this.queryParams.timeRange[1]
      }
      roleApi.findPage(params)
        .then((response) => {
          const res = response.data
          this.loading = false
          this.tableData = res.data
        })
    },
    sortChange (val) {
      this.sort.field = val.prop
      this.sort.order = val.order
      this.search()
    },
    authResource (row) {
      this.roleAuthorityDialog.isVisible = true
      this.$refs.roleAuthority.setRoleAuthority(row)
    },
    authUser (row) {
      this.userRoleDialog.isVisible = true
      this.$refs.userRole.setUserRole(row)
    }
  }
}
</script>
<style lang="scss" scoped>
</style>

<style>
.demo-table-expand {
  font-size: 0;
}
.demo-table-expand label {
  width: 90px;
  color: #99a9bf;
}
.demo-table-expand .el-form-item {
  margin-right: 0;
  margin-bottom: 0;
  width: 50%;
}
.itemP {
  height: 20px;
  width: 50%;
  float: left;
}
.itemP .el-button--text {
  font-size: 14px;
}
.itemP span {
  font-size: 14px;
  display: inline-block;
  margin: 0px 5px;
}
.not-select {
  /* -moz-user-select: none; 
  -webkit-user-select: none;
  -ms-user-select: none; 
  -khtml-user-select: none; 
  user-select: none; */
}
.el-table__expanded-cell {
  padding: 0 !important;
}
</style>

<template>
  <div>
    <el-form :inline="true" :model="formInline" class="demo-form-inline">
      <el-form-item label="渠道组">
        <el-select v-model="formInline.channelGroup" clearable filterable>
          <el-option
            v-for="item in channelList"
            :key="item.id"
            :label="item.name"
            :value="item.id"
          ></el-option>
        </el-select>
      </el-form-item>

      <el-form-item label="渠道">
        <el-input
          v-model="formInline.channelVal"
          placeholder="渠道value"
        ></el-input>
      </el-form-item>

      <el-form-item>
        <el-button @click="searchList">查询</el-button>
        <el-button type="primary" @click="batchTurnChannel" size="medium"
          >批量转移渠道</el-button
        >
        <el-button type="primary" @click="add" size="medium"
          >添加渠道</el-button
        >
      </el-form-item>
    </el-form>

    <!-- 表格报表 -->
    <!-- <p style="font-size: 14px; color: #999">
      转移渠道组模块功能介绍：渠道组展开渠道列表转移选中的渠道，否则转移当前选中渠道组下面的所有渠道。
    </p> -->
    <el-row>
      <el-table
        ref="multipleTable"
        :data="tableData"
        tooltip-effect="dark"
        style="width: 100%"
        v-loading="loading"
        :default-expand-all="expandAll"
        v-if="reloadTable"
        @expand-change="expandChange"
        @select="handleSelect"
        @select-all="handleSelectAll"
      >
        <el-table-column type="selection" width="55"> </el-table-column>

        <el-table-column type="expand">
          <template slot-scope="props">
            <div style="overflow: hidden">
              <channelSelect
                :ref="`multipleTableChild${props.row.id}`"
                :moreListNum="moreListNum"
                :oriDatas="props.row"
                :curDatas="temSubChannelList[props.row.id]"
                @syncDatas="syncDatas"
                @reurnChannel="reurnChannel"
                @editChannel="editChannel"
                v-if="temSubChannelList[props.row.id].flag"
              />
            </div>
          </template>
        </el-table-column>
        <el-table-column align="center" label="id" prop="id"> </el-table-column>
        <el-table-column align="center" label="系统" prop="type">
          <template slot-scope="scope">
            {{
              scope.row.type == 0
                ? 'ios & android'
                : scope.row.type == 1
                ? 'ios'
                : 'android'
            }}
          </template>
        </el-table-column>
        <el-table-column align="center" label="平台" prop="platformid">
          <template slot-scope="scope">
            {{ getVersionName(scope.row.platformid) }}
          </template>
        </el-table-column>
        <el-table-column align="center" label="应用" prop="apptypeid">
          <template slot-scope="scope">
            {{ getAppName(scope.row.apptypeid) }}
          </template>
        </el-table-column>
        <el-table-column align="center" label="渠道组名称" prop="name">
        </el-table-column>
        <el-table-column align="center" label="渠道组值" prop="value">
        </el-table-column>
        <el-table-column align="center" label="操作">
          <template slot-scope="scope">
            <el-button type="text" @click="addChild(scope.row, 'addChild')"
              >添加渠道</el-button
            >
          </template>
        </el-table-column>
      </el-table>
    </el-row>

    <!-- 转移渠道组 -->
    <el-dialog
      title="转移渠道组"
      :visible.sync="dialogVisibleReturn"
      :before-close="beforeCloseChannel"
      :close-on-click-modal="false"
      width="500px"
    >
      转到渠道组：<el-select v-model="channelId" placeholder="请选择">
        <el-option
          v-for="item in channelOriginalDatas"
          :key="item.id"
          :label="item.name"
          :value="item.id"
        >
        </el-option>
      </el-select>
      <el-button
        @click="toOtherChannel"
        type="primary"
        style="margin-left: 10px"
        >确定</el-button
      >
    </el-dialog>

    <!-- 添加 -->
    <el-dialog
      :title="channelTitle"
      :visible.sync="dialogVisible"
      :close-on-click-modal="false"
      width="60%"
      top="2%"
    >
      <el-form
        :model="dynamicValidateForm"
        ref="dynamicValidateForm"
        :rules="rules"
        label-width="100px"
        class="demo-dynamic"
        v-if="dialogVisible"
      >
        <el-form-item label="" label-width="30px">
          <el-checkbox
            v-model="isAliveGroup"
            :disabled="!!dynamicValidateForm.id"
            >是否向已有渠道组中添加渠道</el-checkbox
          >
        </el-form-item>

        <el-form-item label="渠道组：" v-if="isAliveGroup">
          <el-select v-model="channelId" filterable>
            <el-option
              v-for="item in channelList"
              :key="item.id"
              :label="item.name"
              :value="item.id"
            ></el-option>
          </el-select>
        </el-form-item>

        <el-form-item label="渠道组名：" prop="name" v-if="!isAliveGroup">
          <el-input
            v-model="dynamicValidateForm.name"
            :disabled="!!dynamicValidateForm.id"
          ></el-input>
        </el-form-item>
        <el-form-item label="渠道组值：" prop="value" v-if="!isAliveGroup">
          <el-input
            v-model="dynamicValidateForm.value"
            :disabled="!!dynamicValidateForm.id"
          ></el-input>
        </el-form-item>

        <el-form-item label="系统：" prop="type" v-if="!isAliveGroup">
          <el-radio-group
            v-model="dynamicValidateForm.type"
            :disabled="!!dynamicValidateForm.id"
          >
            <el-radio :label="1">ios</el-radio>
            <el-radio :label="2">android</el-radio>
          </el-radio-group>
        </el-form-item>

        <el-form-item label="应用：" prop="apptypeid" v-if="!isAliveGroup">
          <el-radio-group
            v-model="dynamicValidateForm.apptypeid"
            :disabled="!!dynamicValidateForm.id"
          >
            <el-radio :label="item.id" v-for="item in appList" :key="item.id">
              {{ item.name }}-{{ item.value }}</el-radio
            >
          </el-radio-group>
        </el-form-item>
        <el-form-item label="平台：" prop="platformid" v-if="!isAliveGroup">
          <el-radio-group
            v-model="dynamicValidateForm.platformid"
            :disabled="!!dynamicValidateForm.id"
          >
            <el-radio
              :label="item.id"
              v-for="item in versionList"
              :key="item.id"
            >
              {{ item.value }}</el-radio
            >
          </el-radio-group>
        </el-form-item>

        <el-form-item label="添加类型：">
          <el-radio-group v-model="addChannelType">
            <el-radio :label="0">按照key-value添加</el-radio>
            <el-radio :label="1">按照规则添加</el-radio>
          </el-radio-group>
        </el-form-item>

        <el-card
          v-for="(domain, index) in dynamicValidateForm.subchannel"
          :key="domain.key"
          style="position: relative; margin-bottom: 10px; padding-top: 20px"
        >
          <p>
            <span style="position: absolute; left: 10px; top: 10px"
              >NO.{{ index + 1 }}</span
            >
            <i
              v-if="dialogType != 'edit'"
              class="el-icon-remove"
              title="REMOVE/删除"
              style="
                font-size: 20px;
                position: absolute;
                right: 5px;
                top: 5px;
                cursor: pointer;
                color: red;
              "
              @click.prevent="removeDomain(domain)"
            ></i>
          </p>

          <el-form-item label="渠道名称：">
            <el-input
              v-model="domain.name"
              placeholder="请输入渠道名称"
            ></el-input>
          </el-form-item>

          <el-form-item label="渠道值：">
            <el-input
              v-model="domain.value"
              placeholder="请输入渠道值"
            ></el-input>
          </el-form-item>
          <el-form-item label="数量：" v-if="addChannelType == 1">
            <el-input-number v-model="domain.num" :min="1"></el-input-number>
          </el-form-item>
        </el-card>
        <el-form-item>
          <el-button type="primary" @click="submitForm('dynamicValidateForm')"
            >提交</el-button
          >
          <el-button @click="addDomain" v-if="dialogType != 'edit'"
            >新增渠道</el-button
          >
          <el-button @click="resetForm('dynamicValidateForm')">重置</el-button>
        </el-form-item>
      </el-form>
    </el-dialog>

    <!-- 修改渠道 -->
    <el-dialog
      title="修改渠道"
      :visible.sync="dialogChannel"
      :close-on-click-modal="false"
    >
      <el-form>
        <el-form-item label="渠道名称：">
          <el-input
            placeholder="渠道名称"
            v-model="channelForm.subchannel[0].name"
            disabled
          />
        </el-form-item>
        <el-form-item label="渠道值：">
          <el-input
            placeholder="渠道值"
            v-model="channelForm.subchannel[0].value"
          />
        </el-form-item>
      </el-form>

      <el-button @click="addChannelHandle" type="primary">提交</el-button>
    </el-dialog>
  </div>
</template>
<script>
import {
  getchannelgroup,
  addchannelgroup,
  putchannelgroup,
  getapptypeid,
  getversion,
} from '../../api/common'
import { getCookie } from '@/lib/util'

const userName = getCookie('USER-COOKIE-INFO') || '用户'

import channelSelect from './components/channelSelect'

export default {
  components: {
    channelSelect,
  },
  data() {
    const columnDatas = [
      { prop: 'id', label: 'id' },
      { prop: 'name', label: 'name' },
      { prop: 'value', label: 'value' },
      { prop: 'type', label: 'type' },
    ]
    return {
      childChannelIds: [],
      moreListNum: 500,
      temSubChannelList: {},
      expandAll: false,
      reloadTable: true,
      isAliveGroup: false,
      formInline: {
        channelGroup: '',
        channelVal: '',
      },
      domTable: '',
      loading: false,
      searchInput: '',
      channelForm: {
        id: 1,
        subchannel: [
          {
            id: 1,
            name: '测试子',
            value: 'test1',
          },
        ],
      },
      channelList: [],
      channelOriginalDatas: [],
      channelId: '',
      dialogVisibleReturn: false,
      dialogChannel: false,
      dialogVisible: false,
      channelTitle: '添加渠道',
      addChannelType: 0,
      systemList: [],
      appList: [],
      versionList: [],
      dialogType: '',
      rules: {
        name: [{ required: true, message: '请输入渠道组名', trigger: 'blur' }],
        value: [{ required: true, message: '请输入渠道组值', trigger: 'blur' }],
        type: [{ required: true, message: '请选择系统', trigger: 'change' }],
        apptypeid: [
          { required: true, message: '请选择应用', trigger: 'change' },
        ],
        platformid: [
          { required: true, message: '请选择平台', trigger: 'change' },
        ],
      },
      temNum: 10,
      dynamicValidateForm: {
        id: 0,
        type: 1,
        name: '',
        value: '',
        apptypevalue: 'DFTT',
        platformid: '',
        apptypeid: '',
        subchannel: [
          {
            name: '',
            value: '',
            num: 1,
          },
        ],
      },

      //表格数据
      tableData: [],
    }
  },
  created() {
    this.getList()
    getapptypeid().then((res) => {
      if (res.status != 0) {
        this.$message.error(res.msg || '获取应用信息失败')
        return
      }
      let temd = []
      for (let item in res.result) {
        temd.push(res.result[item])
      }
      this.appList = temd
    })

    getversion().then((res) => {
      if (res.status != 0) {
        this.$message.error(res.msg || '获取平台信息失败')
        return
      }
      let temd = []
      for (let item in res.result) {
        temd.push(res.result[item])
      }

      this.versionList = temd
    })
  },
  methods: {
    expandChange() {
      setTimeout(() => {
        this.tableData.forEach((x) => {
          if (typeof this.$refs[`multipleTableChild${x.id}`] != 'undefined') {
            this.$refs[`multipleTableChild${x.id}`].restList()
            this.$refs[`multipleTableChild${x.id}`].resetDomBox()
          }
        })
      }, 100)
    },
    syncDatas(d, id) {
      this.temSubChannelList[id] = d
    },

    beforeCloseChannel() {
      this.dialogVisibleReturn = false
      this.childChannelIds = []
      this.dynamicValidateForm.subchannel = [
        {
          name: '',
          value: '',
          num: 1,
        },
      ]
    },

    // 父级多选
    handleSelectAll(d) {
      if (!d.length) {
        const oriDatas = JSON.parse(
          window.sessionStorage.getItem('channelOriginalDatas')
        )
        oriDatas.forEach((x) => {
          this.temSubChannelList[x.id].slected = []
          this.temSubChannelList[x.id].flag = false
          this.$nextTick(() => {
            this.temSubChannelList[x.id].flag = true
          })
        })
        return
      }
      d.forEach((x) => {
        this.handleSelect(d, x)
      })
    },

    // 父级单选
    handleSelect(d, item) {
      let ids = this.$refs.multipleTable.selection.map((x) => x.id)

      if (ids.includes(item.id)) {
        let temD = item.subchannel.slice(
          0,
          this.temSubChannelList[item.id].ind * this.moreListNum
        )
        this.temSubChannelList[item.id].slected = temD.map((x) => x.id)
      } else {
        this.temSubChannelList[item.id].slected = []
      }

      this.temSubChannelList[item.id].flag = false
      this.$nextTick(() => {
        this.temSubChannelList[item.id].flag = true
      })
    },
    searchList() {
      this.loading = true
      this.expandAll = true
      if (!this.formInline.channelGroup && !this.formInline.channelVal) {
        this.expandAll = false
      }
      const oriDatas = JSON.parse(
        window.sessionStorage.getItem('channelOriginalDatas')
      )
      this.tableData = !this.formInline.channelGroup
        ? [...oriDatas]
        : oriDatas.filter((x) => x.id == this.formInline.channelGroup)

      const temD = [...this.tableData]

      temD.forEach((x) => {
        x.subchannel = x.subchannel.filter(
          (y) => y.value.indexOf(this.formInline.channelVal) > -1
        )
      })

      this.tableData = temD.filter((x) => x.subchannel.length)

      this.reloadTable = false
      this.$nextTick(() => {
        this.reloadTable = true
      })
      setTimeout(() => {
        this.loading = false
      }, 1000)
    },
    batchTurnChannel() {
      const { selection } = this.$refs.multipleTable
      let childIds = []
      for (let item in this.temSubChannelList) {
        childIds = [...childIds, ...this.temSubChannelList[item].slected]
      }

      if (!selection.length && !childIds.length) {
        this.$message.warning('请勾选要批量操作的信息')
        return
      }

      selection.forEach((x) => {
        if (typeof this.$refs[`multipleTableChild${x.id}`] == 'undefined') {
          // 未展开选中情况
          let temId = x.subchannel.map((y) => y.id)
          childIds = [...childIds, ...temId]
        }
      })

      this.childChannelIds = childIds

      this.dialogVisibleReturn = true
    },
    addChannelHandle() {
      putchannelgroup({
        username: userName,
        object: this.channelForm,
      }).then((res) => {
        if (res.status != 0) {
          this.$message.error(res.msg || '修改失败')
          return
        }
        this.$message.success('修改成功')
        this.dialogChannel = false
        this.getList()
      })
    },
    getVersionName(d) {
      const item = this.versionList.find((x) => x.id === d)
      if (!item) {
        return '--'
      }
      return item.value || '--'
    },
    getAppName(d) {
      const item = this.appList.find((x) => x.id === d)
      if (!item) {
        return '--'
      }
      return `${item.name}-${item.value}`
    },
    reurnChannel(d, item) {
      this.dialogVisibleReturn = true
      this.dynamicValidateForm = {
        id: d.id,
        type: d.type,
        name: d.name,
        value: d.value,
        apptypevalue: 'DFTT',
        subchannel: [item],
      }
    },
    toOtherChannel() {
      let temSubChannel = this.dynamicValidateForm.subchannel
      let temArr = []
      let temSubChannelChild = []

      if (this.childChannelIds.length > 0) {
        this.tableData.forEach((y) => {
          y.subchannel.length &&
            y.subchannel.forEach((z) => {
              temSubChannelChild.push(z)
            })
        })

        this.childChannelIds.forEach((x) => {
          const item = temSubChannelChild.find((y) => y.id == x)
          temArr.push(item)
        })

        temSubChannel = temArr
      }

      let temD0 = {
        username: userName,
        object: {
          id: this.channelId,
          subchannel: temSubChannel,
        },
      }

      putchannelgroup(temD0).then((res) => {
        if (res.status != 0) {
          this.$message.error(res.msg || '修改失败')
          return
        }
        this.$message.success('修改成功')
        this.dialogVisibleReturn = false
        this.childChannelIds = []
        this.formInline.channelGroup = ''
        this.formInline.channelVal = ''
        this.$refs.multipleTable.clearSelection()

        this.getList()
      })
    },
    editChannel(d, pid) {
      this.dialogChannel = true
      this.channelForm = {
        id: pid,
        subchannel: [
          {
            id: d.id,
            name: d.name,
            value: d.value,
          },
        ],
      }
    },
    getList() {
      this.loading = true
      this.expandAll = false
      this.temSubChannelList = {}
      this.channelOriginalDatas = []
      getchannelgroup().then((res) => {
        res.result.dftt.forEach((x) => {
          this.temSubChannelList = {
            ...this.temSubChannelList,
            [x.id]: {
              ind: 1,
              isall: false,
              slected: [],
              flag: true,
            },
          }
        })

        this.channelOriginalDatas = [...res.result.dftt]
        window.sessionStorage.setItem(
          'channelOriginalDatas',
          JSON.stringify(res.result.dftt)
        )
        this.tableData = res.result && res.result.dftt
        this.loading = false
        this.tableData.forEach((x) => {
          this.channelList.push({
            id: x.id,
            name: x.name,
            list: x.subchannel,
          })
        })
      })
    },
    handleSizeChange(val) {
      this.pageSize = val
      this.search()
    },
    handleCurrentChange(val) {
      this.currentPage = val
      this.search()
    },
    add() {
      this.addChannelType = 0
      this.isAliveGroup = false
      this.dialogVisible = true
      this.channelId = ''
      this.dynamicValidateForm = {
        id: 0,
        type: 1,
        name: '',
        value: '',
        apptypevalue: 'DFTT',
        platformid: '',
        apptypeid: '',
        subchannel: [
          {
            name: '',
            value: '',
            num: 1,
          },
        ],
      }
    },
    addChild(d, type) {
      this.isAliveGroup = false
      this.dialogVisible = true
      this.channelId = ''
      this.dynamicValidateForm = {
        id: d.id,
        type: parseInt(d.type),
        name: d.name,
        value: d.value,
        apptypevalue: 'DFTT',
        platformid: d.platformid,
        apptypeid: d.apptypeid,
        subchannel: d.subchannel,
      }

      this.dialogType = type
      this.dynamicValidateForm.name = d.name
      this.dynamicValidateForm.value = d.value
      this.dynamicValidateForm.id = d.id
      this.dialogVisible = true
      this.dynamicValidateForm = {
        id: d.id,
        type: parseInt(d.type),
        name: d.name,
        value: d.value,
        apptypevalue: 'DFTT',
        platformid: d.platformid,
        apptypeid: d.apptypeid,
        subchannel: [
          {
            name: '',
            value: '',
            num: 1,
          },
        ],
      }
    },
    submitForm(formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          let channelGroupObj = this.dynamicValidateForm
          this.channelId = this.dynamicValidateForm.id
            ? this.dynamicValidateForm.id
            : this.channelId

          if (this.addChannelType == 1) {
            let temsubChannel = []

            const temObj = this.channelOriginalDatas.find(
              (x) => x.id == this.channelId
            )

            this.dynamicValidateForm.subchannel.forEach((x) => {
              for (let i = 1; i <= x.num; i++) {
                temsubChannel.push({
                  name: `${x.name}${i}`,
                  value: `${x.value}${i}`,
                })
              }
            })

            channelGroupObj = {
              id: temObj.id,
              type: temObj.type,
              name: temObj.name,
              value: temObj.value,
              apptypevalue: 'DFTT',
              platformid: temObj.platformid,
              apptypeid: temObj.apptypeid,
              subchannel: temsubChannel,
            }
          }

          let temD = {
            username: userName,
            object: channelGroupObj,
          }

          addchannelgroup(temD).then((res) => {
            if (res.status != 0) {
              this.$message.error(res.msg || '添加失败')
              return
            }
            this.$message.success('添加成功')
            this.dialogVisible = false
            this.$refs.dynamicValidateForm.resetFields()
            this.resetForm(formName)
            this.getList()
          })
        } else {
          console.log('error submit!!')
          return false
        }
      })
    },
    resetForm(formName) {
      this.$refs[formName].resetFields()
      this.dynamicValidateForm.subchannel = [
        {
          name: '',
          value: '',
          num: 1,
        },
      ]
    },
    removeDomain(item) {
      var index = this.dynamicValidateForm.subchannel.indexOf(item)
      if (index !== -1) {
        this.dynamicValidateForm.subchannel.splice(index, 1)
      }
    },
    addDomain() {
      this.dynamicValidateForm.subchannel.push({
        id: 0,
        value: '',
        name: '',
        num: 1,
        key: Date.now(),
      })
    },
  },
}
</script>

<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="所属部门">
              <a-input placeholder="请输入所属部门" v-model="queryParam.udept"></a-input>
            </a-form-item>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="姓名">
              <a-input placeholder="请输入姓名" v-model="queryParam.uname"></a-input>
            </a-form-item>
          </a-col>
          <template v-if="toggleSearchStatus">
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="手机">
                <a-input placeholder="请输入手机" v-model="queryParam.uphone"></a-input>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="邮箱">
                <a-input placeholder="请输入邮箱" v-model="queryParam.uemail"></a-input>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="账号状态">
                <j-dict-select-tag placeholder="请选择账号状态" v-model="queryParam.ustatus" dictCode="disableer"/>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="开通账号">
                <j-multi-select-tag placeholder="请选择开通账号" dictCode="zs_acc_sys" v-model="queryParam.uaccs"/>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="GIT账号">
                <a-input placeholder="请输入GIT账号" v-model="queryParam.uaccGit"></a-input>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="VPN账号">
                <a-input placeholder="请输入VPN账号" v-model="queryParam.uaccVpn"></a-input>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="禅道账号">
                <a-input placeholder="请输入禅道账号" v-model="queryParam.uaccZentao"></a-input>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="创建人">
                <a-input placeholder="请输入创建人" v-model="queryParam.createBy"></a-input>
              </a-form-item>
            </a-col>
          </template>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
              <a @click="handleToggleSearch" style="margin-left: 8px">
                {{ toggleSearchStatus ? '收起' : '展开' }}
                <a-icon :type="toggleSearchStatus ? 'up' : 'down'"/>
              </a>
            </span>
          </a-col>
        </a-row>
      </a-form>
    </div>
    <!-- 查询区域-END -->

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('研发资源账号')">导出</a-button>
      <!--<a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">-->
        <!--<a-button type="primary" icon="import">导入</a-button>-->
      <!--</a-upload>-->
      <!-- 高级查询区域 -->
      <j-super-query :fieldList="superFieldList" ref="superQueryModal" @handleSuperQuery="handleSuperQuery"></j-super-query>
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel"><a-icon type="delete"/>密码重置</a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作 <a-icon type="down" /></a-button>
      </a-dropdown>
    </div>

    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{ selectedRowKeys.length }}</a>项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
      </div>

      <a-table
        ref="table"
        size="middle"
        :scroll="{x:true}"
        bordered
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        class="j-table-force-nowrap"
        @change="handleTableChange">

        <template slot="htmlSlot" slot-scope="text">
          <div v-html="text"></div>
        </template>
        <template slot="imgSlot" slot-scope="text,record">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无图片</span>
          <img v-else :src="getImgView(text)" :preview="record.id" height="25px" alt="" style="max-width:80px;font-size: 12px;font-style: italic;"/>
        </template>
        <template slot="fileSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无文件</span>
          <a-button
            v-else
            :ghost="true"
            type="primary"
            icon="download"
            size="small"
            @click="downloadFile(text)">
            下载
          </a-button>
        </template>

        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>
          &nbsp;
          <a @click="handleDelete(record.id)">禁用</a>

          <!--<a-divider type="vertical" />-->
          <!--<a-dropdown>-->
            <!--<a class="ant-dropdown-link">更多 <a-icon type="down" /></a>-->
            <!--<a-menu slot="overlay">-->
              <!--<a-menu-item>-->
                <!--<a @click="handleDetail(record)">详情</a>-->
              <!--</a-menu-item>-->
              <!--<a-menu-item>-->
                <!--<a-popconfirm title="确定禁用吗?" @confirm="() => handleDelete(record.id)">-->
                  <!--<a>禁用</a>-->
                <!--</a-popconfirm>-->
              <!--</a-menu-item>-->
            <!--</a-menu>-->
          <!--</a-dropdown>-->
        </span>

      </a-table>
    </div>

    <zs-acc-users-modal ref="modalForm" @ok="modalFormOk"></zs-acc-users-modal>
  </a-card>
</template>

<script>

  import '@/assets/less/TableExpand.less'
  import { mixinDevice } from '@/utils/mixin'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import ZsAccUsersModal from './modules/ZsAccUsersModal'
  import {filterMultiDictText} from '@/components/dict/JDictSelectUtil'

  export default {
    name: 'ZsAccUsersList',
    mixins:[JeecgListMixin, mixinDevice],
    components: {
      ZsAccUsersModal
    },
    data () {
      return {
        description: '研发资源账号管理页面',
        // 表头
        columns: [
          {
            title: '#',
            dataIndex: '',
            key:'rowIndex',
            width:60,
            align:"center",
            customRender:function (t,r,index) {
              return parseInt(index)+1;
            }
          },
          {
            title:'所属部门',
            align:"center",
            sorter: true,
            dataIndex: 'udept'
          },
          {
            title:'姓名',
            align:"center",
            dataIndex: 'uname'
          },
          {
            title:'手机',
            align:"center",
            dataIndex: 'uphone'
          },
          {
            title:'邮箱',
            align:"center",
            dataIndex: 'uemail'
          },
          {
            title:'账号状态',
            align:"center",
            dataIndex: 'ustatus_dictText'
          },
          {
            title:'开通账号',
            align:"center",
            dataIndex: 'uaccs_dictText'
          },
          {
            title:'GIT账号',
            align:"center",
            dataIndex: 'uaccGit'
          },
          {
            title:'VPN账号',
            align:"center",
            dataIndex: 'uaccVpn'
          },
          {
            title:'禅道账号',
            align:"center",
            dataIndex: 'uaccZentao'
          },
          {
            title:'创建人',
            align:"center",
            dataIndex: 'createBy'
          },
          {
            title:'创建日期',
            align:"center",
            dataIndex: 'createTime'
          },
          {
            title: '操作',
            dataIndex: 'action',
            align:"center",
            fixed:"right",
            width:147,
            scopedSlots: { customRender: 'action' }
          }
        ],
        url: {
          list: "/otheraccount/zsAccUsers/list",
          delete: "/otheraccount/zsAccUsers/delete",
          deleteBatch: "/otheraccount/zsAccUsers/deleteBatch",
          exportXlsUrl: "/otheraccount/zsAccUsers/exportXls",
          importExcelUrl: "otheraccount/zsAccUsers/importExcel",

        },
        dictOptions:{},
        superFieldList:[],
      }
    },
    created() {
    this.getSuperFieldList();
    },
    computed: {
      importExcelUrl: function(){
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
      },
    },
    methods: {
      initDictConfig(){
      },
      getSuperFieldList(){
        let fieldList=[];
        fieldList.push({type:'string',value:'udept',text:'所属部门',dictCode:''})
        fieldList.push({type:'string',value:'uname',text:'姓名',dictCode:''})
        fieldList.push({type:'string',value:'uphone',text:'手机',dictCode:''})
        fieldList.push({type:'string',value:'uemail',text:'邮箱',dictCode:''})
        fieldList.push({type:'string',value:'ustatus',text:'账号状态',dictCode:'disableer'})
        fieldList.push({type:'list_multi',value:'uaccs',text:'开通账号',dictTable:"", dictText:'', dictCode:'zs_acc_sys'})
        fieldList.push({type:'string',value:'uaccGit',text:'GIT账号',dictCode:''})
        fieldList.push({type:'string',value:'uaccVpn',text:'VPN账号',dictCode:''})
        fieldList.push({type:'string',value:'uaccZentao',text:'禅道账号',dictCode:''})
        fieldList.push({type:'string',value:'createBy',text:'创建人',dictCode:''})
        fieldList.push({type:'datetime',value:'createTime',text:'创建日期'})
        fieldList.push({type:'string',value:'updateBy',text:'更新人',dictCode:''})
        fieldList.push({type:'datetime',value:'updateTime',text:'更新日期'})
        this.superFieldList = fieldList
      }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less';
</style>
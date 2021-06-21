<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
        </a-row>
      </a-form>
    </div>
    <!-- 查询区域-END -->

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('gps_meal_list')">导出</a-button>
      <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload>
      <!-- 高级查询区域 -->
      <j-super-query :fieldList="superFieldList" ref="superQueryModal" @handleSuperQuery="handleSuperQuery"></j-super-query>
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel"><a-icon type="delete"/>删除</a-menu-item>
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
        <template slot="imgSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无图片</span>
          <img v-else :src="getImgView(text)" height="25px" alt="" style="max-width:80px;font-size: 12px;font-style: italic;"/>
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

          <a-divider type="vertical" />
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down" /></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a @click="handleDetail(record)">详情</a>
              </a-menu-item>
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>

      </a-table>
    </div>

    <gps-meal-list-modal ref="modalForm" @ok="modalFormOk"></gps-meal-list-modal>
  </a-card>
</template>

<script>

  import '@/assets/less/TableExpand.less'
  import { mixinDevice } from '@/utils/mixin'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import GpsMealListModal from './modules/GpsMealListModal'
  import JSuperQuery from '@/components/jeecg/JSuperQuery.vue'

  export default {
    name: 'GpsMealListList',
    mixins:[JeecgListMixin, mixinDevice],
    components: {
      GpsMealListModal,
      JSuperQuery,
    },
    data () {
      return {
        description: 'gps_meal_list管理页面',
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
            title:'套餐名称',
            align:"center",
            dataIndex: 'mealName'
          },
          {
            title:'套餐时长',
            align:"center",
            dataIndex: 'mealSpare1'
          },
          {
            title:'套餐描述',
            align:"center",
            dataIndex: 'mealDescription'
          },
          {
            title:'套餐价格',
            align:"center",
            dataIndex: 'mealPrice'
          },
          {
            title:'套餐流量',
            align:"center",
            dataIndex: 'mealFlow'
          },
          {
            title:'套餐权限',
            align:"center",
            dataIndex: 'mealContent'
          },
          {
            title:'套餐成本',
            align:"center",
            dataIndex: 'cost'
          },
          {
            title:'套餐分成',
            align:"center",
            dataIndex: 'ratio'
          },
          {
            title:'套餐利润',
            align:"center",
            dataIndex: 'profit'
          },

          /*{
            title:'备用2',
            align:"center",
            dataIndex: 'mealSpare2'
          },*/
          {
            title:'创建时间',
            align:"center",
            dataIndex: 'createTime',
            customRender:function (text) {
              return !text?"":(text.length>10?text.substr(0,10):text)
            }
          },
          /*{
            title:'更新人',
            align:"center",
            dataIndex: 'updateBy'
          },
          {
            title:'更新时间',
            align:"center",
            dataIndex: 'updateTime',
            customRender:function (text) {
              return !text?"":(text.length>10?text.substr(0,10):text)
            }
          },*/
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
          list: "/sys/gpslist/gpsMealList/list",
          delete: "/sys/gpslist/gpsMealList/delete",
          deleteBatch: "/sys/gpslist/gpsMealList/deleteBatch",
          exportXlsUrl: "/sys/gpslist/gpsMealList/exportXls",
          importExcelUrl: "sys/gpslist/gpsMealList/importExcel",
          
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
        fieldList.push({type:'string',value:'mealName',text:'套餐名称'})
        fieldList.push({type:'string',value:'mealDescription',text:'套餐描述'})
        fieldList.push({type:'string',value:'mealPrice',text:'套餐价格'})
        fieldList.push({type:'string',value:'mealFlow',text:'套餐流量'})
        fieldList.push({type:'string',value:'mealContent',text:'套餐内容'})
        fieldList.push({type:'string',value:'cost',text:'套餐成本'})
        fieldList.push({type:'string',value:'ratio',text:'套餐分成'})
        fieldList.push({type:'string',value:'profit',text:'套餐利润'})
        fieldList.push({type:'string',value:'mealSpare1',text:'备用1'})
        fieldList.push({type:'string',value:'mealSpare2',text:'备用2'})
        fieldList.push({type:'date',value:'createTime',text:'创建时间'})
        fieldList.push({type:'string',value:'updateBy',text:'更新人'})
        fieldList.push({type:'date',value:'updateTime',text:'更新时间'})
        this.superFieldList = fieldList
      }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less';
</style>
<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form :form="form" slot="detail">
        <a-row>
          <a-col :span="24">
            <a-form-item label="套餐名称" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['mealName']" placeholder="请输入套餐名称"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="套餐时长" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['mealSpare1']" placeholder="请输入备用1"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="套餐描述" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['mealDescription']" placeholder="请输入套餐描述"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="套餐价格" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['mealPrice']" placeholder="请输入套餐价格"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="套餐流量" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['mealFlow']" placeholder="请输入套餐流量"  ></a-input>
            </a-form-item>
          </a-col>
          <!--<a-col :span="24">
            <a-form-item label="套餐内容" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['mealContent']" placeholder="请输入套餐内容"  ></a-input>
            </a-form-item>
          </a-col>-->

          <!--租户分配-->
          <a-col :span="24">
            <a-form-item label="套餐权限" :labelCol="labelCol" :wrapperCol="wrapperCol" v-show="!departDisabled">
              <a-select
                mode="multiple"
                style="width: 100%"
                placeholder="请选择套餐内容"
                :disabled="disableSubmit"
                v-model="currentTenant">
                <a-select-option v-for="(item, index) in tenantList" :key="index" :value="item.productName">
                  {{ item.productName }}
                </a-select-option>
              </a-select>
            </a-form-item>
          </a-col>


          <!--<a-col :span="24">
            <a-form-item label="备用2" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['mealSpare2']" placeholder="请输入备用2"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="创建时间" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-date placeholder="请选择创建时间" v-decorator="['createTime']" :trigger-change="true" style="width: 100%" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="更新人" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['updateBy']" placeholder="请输入更新人"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="更新时间" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-date placeholder="请选择更新时间" v-decorator="['updateTime']" :trigger-change="true" style="width: 100%" />
            </a-form-item>
          </a-col>-->
          <a-col v-if="showFlowSubmitButton" :span="24" style="text-align: center">
            <a-button @click="submitForm">提 交</a-button>
          </a-col>
        </a-row>
      </a-form>
    </j-form-container>
  </a-spin>
</template>

<script>

  import { httpAction, getAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import { validateDuplicateValue } from '@/utils/util'
  import JFormContainer from '@/components/jeecg/JFormContainer'
  import JDate from '@/components/jeecg/JDate'


  export default {
    name: 'GpsMealListForm',
    components: {
      JFormContainer,
      JDate,
    },
    props: {
      //流程表单data
      formData: {
        type: Object,
        default: ()=>{},
        required: false
      },
      //表单模式：true流程表单 false普通表单
      formBpm: {
        type: Boolean,
        default: false,
        required: false
      },
      //表单禁用
      disabled: {
        type: Boolean,
        default: false,
        required: false
      }
    },
    data () {
      return {
        departDisabled: false,
        roleDisabled: false,
        disableSubmit: false,
        checkedDepartNameString:"",
        form: this.$form.createForm(this),
        model: {},
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },
        confirmLoading: false,
        validatorRules: {
        },
        url: {
          add: "/sys/gpslist/gpsMealList/add",
          edit: "/sys/gpslist/gpsMealList/edit",
          queryById: "/sys/gpslist/gpsMealList/queryById",
          queryTenantList: '/sys/gpslist/gpsProductList/queryList'
        },
        currentTenant:[],
        tenantList: [],
      }
    },
    refresh () {
      this.currentTenant = []
    },
    computed: {
      formDisabled(){
        if(this.formBpm===true){
          if(this.formData.disabled===false){
            return false
          }
          return true
        }
        return this.disabled
      },
      showFlowSubmitButton(){
        if(this.formBpm===true){
          if(this.formData.disabled===false){
            return true
          }
        }
        return false
      }
    },
    created () {
      //如果是流程中表单，则需要加载流程表单data
      this.showFlowData();
      this.initTenantList();
    },
    methods: {
      add () {
        this.edit({});
      },
      edit (record) {
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'mealName','mealDescription','mealPrice','mealFlow','mealContent','mealSpare1','mealSpare2','createTime','updateBy','updateTime'))
        })
        //update-begin-author:taoyan date:2020710 for:多租户配置
        console.log("测试数据",record.mealContent);
        if(!record.mealContent || record.mealContent.length==0){
          this.currentTenant = [];
        }else{
          this.currentTenant = record.mealContent.split(',');
        }
      },
      //渲染流程表单数据
      showFlowData(){
        if(this.formBpm === true){
          let params = {id:this.formData.dataId};
          getAction(this.url.queryById,params).then((res)=>{
            if(res.success){
              this.edit (res.result);
            }
          });
        }
      },
      submitForm () {
        const that = this;

        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true;
            let httpurl = '';
            let method = '';
            if(!this.model.id){
              httpurl+=this.url.add;
              method = 'post';
            }else{
              httpurl+=this.url.edit;
               method = 'put';
            }
            let formData = Object.assign(this.model, values);

            //console.log("表单提交数据",formData)
            formData.mealContent = this.currentTenant.length>0?this.currentTenant.join(','):'';
            console.log("表单提交数据2:   ",formData.mealContent)
            if(this.currentTenant.length>0){
              let cons =formData.mealContent.split(",");
              //成本
              let tolprice=0;
              //分成
              let tolratio=0;

              for(let i=0;i<cons.length;i++){
                let pros=cons[i];
                for(let j=0;j<this.tenantList.length;j++){
                  //console.log("表单提交数据6:"+this.tenantList[j].productName)
                  //console.log("表单提交数据8:"+j)
                  if(pros==this.tenantList[j].productName){
                    let s=parseFloat(this.tenantList[j].price);
                    let y=parseFloat(this.tenantList[j].shareRatio);
                    let ras=s*y/100;
                    tolprice = tolprice+s;
                    tolratio = tolratio+ras;
                  }
                }
              }
              tolratio = Math.round(tolratio*100)/100;
              //利润
              let profit = formData.mealPrice-tolprice-tolratio;
              //console.log("表单提交数据3:   ",formData.mealPrice+"  :  "+tolprice+"  :  "+tolratio+"  :  "+profit+"  :  ");
              formData.cost=tolprice;
              formData.ratio=tolratio;
              formData.profit=profit;
            }

            httpAction(httpurl,formData,method).then((res)=>{
              if(res.success){
                that.$message.success(res.message);
                that.$emit('ok');
              }else{
                that.$message.warning(res.message);
              }
            }).finally(() => {
              that.confirmLoading = false;
            })
          }
         
        })
      },
      popupCallback(row){
        this.form.setFieldsValue(pick(row,'mealName','mealDescription','mealPrice','mealFlow','mealContent','mealSpare1','mealSpare2','createTime','updateBy','updateTime'))
      },
      initTenantList(){
        getAction(this.url.queryTenantList).then(res=>{
          if(res.success){
            this.tenantList = res.result
          }
        })
      },
    }
  }
</script>
<template>
  <div class="estatecreate">
    <Row type="flex" style="border:1px solid #ccc;padding:20px">
        <Col span="20">
          <Form  :model="form"  inline>
              <FormItem prop="user">
                <span>区域：</span>
                <Select 
                  style="width:150px"
                  v-model="form.province" 
                  clearable  
                  @change = "provinceChange(form.province)"
                  placeholder="省">
                  <Option 
                    v-for="item in provinceIdsList"
                    :key="item.cityId"
                    :label="item.cityName"
                    :value="item.cityId">
                  </Option>
                </Select>
                <Select 
                  v-model="form.city" 
                  clearable  
                  placeholder="市"
                  style="width:150px">
                  <Option
                    v-for="item in cityIdsList"
                    :key="item.cityId"
                    :label="item.cityName"
                    :value="item.cityId">
                  </Option>
                </Select>
                <Select 
                  v-model="form.area" 
                  clearable  
                  placeholder="区"
                  style="width:150px">
                  <Option
                    v-for="item in areaIdsList"
                    :key="item.cityId"
                    :label="item.cityName"
                    :value="item.cityId">
                  </Option>
                </Select>
              </FormItem>
              
              <FormItem prop="password">
                  <span>楼盘名称：</span>
                  <Select
                    style="width:200px"
                    v-model="form.buildingName"
                    filterable
                    remote
                    reserve-keyword
                    placeholder="楼盘名称"
                    :remote-method="remoteMethod">
                    <Option
                      v-for="(item,index) in buidingList"
                      :key="item.key"
                      :label="item.value"
                      :value="item.key">
                    </Option>
                  </Select>
              </FormItem>

              <FormItem prop="password">
                  <span>楼盘创建人：</span>
                  <Input  style="width:100px"  v-model="form.buildingId" placeholder="楼盘创建人"></Input>
              </FormItem>

              <FormItem prop="password">
                  <span>审核状态：</span>
                  <Select
                    style="width:100px"
                    v-model="form.gman">
                    <Option
                      label="全部"
                      value="0">
                    </Option>
                    <Option
                      label="待审核"
                      value="1">
                    </Option>
                    <Option
                      label="驳回"
                      value="1">
                    </Option>
                    <Option
                      label="审核通过"
                      value="1">
                    </Option>
                    <Option
                      label="合并楼盘"
                      value="1">
                    </Option>
                  </Select>
              </FormItem>
          </Form>
        </Col>
        <Col span="4" style="text-align:right">
           <Button type="primary" @click = "searchBegin" icon="ios-search" size="large">搜索</Button>
        </Col>
    </Row>

    <Row style="margin-top:50px">
      <Table border :loading="tableLoading" :columns="columns1" :data="data1"></Table>
      <Page
        style = "text-align:center;margin-top:40px" 
        :total = "50"
        :page-size = "10"
        :current = "2"
        show-total
        show-elevator
        @on-change = "pageChange"
        >
      </Page>
    </Row>
  </div>
</template>
<script>
export default {
  name: 'estatecreate',
  data () {
    return {
      provinceIdsList:[],
      cityIdsList:[],
      buidingList:[],
      areaIdsList:[],
      


      tableLoading:false,
      columns1:[ 
        {
            title: '楼盘',
            key: 'name',
            fixed:'left',
            width:150
        },
        {
            title: '所在地区',
            key: 'address',
            width:150,
        },
        {
            title: '期数',
            key: 'address',
            width:150,
        },
        {
            title: '楼幢数量',
            key: 'isf',
            width:150,
        },
        {
            title: '最高层数',
            key: 'tman',
            width:150,
        },
        {
            title:'梯户比',
            key:'time',
            width:150,
        },
        {
            title:'照片量',
            key:'time',
            width:150,
        },
        {
            title:'楼盘创建人',
            key:'time',
            width:150,
        },
        {
            title:'创建时间',
            key:'time',
            width:150,
        },
        {
            title:'审核状态',
            key:'time',
            width:150,
        },
        {
            title:'操作',
            key:'action',
            fixed:'right',
            width:150,
            render: (h, params) => {
              return h('div', [
                  h('Button', {
                      props: {
                          type: 'primary',
                          size: 'small'
                      },
                      style: {
                          marginRight: '5px'
                      },
                      on: {
                          click: () => {
                              this.handle(params,'view',1)
                          }
                      }
                  }, '查看'),
                  h('Button', {
                      props: {
                          type: 'primary',
                          size: 'small'
                      },
                      style: {
                          marginRight: '5px'
                      },
                      on: {
                          click: () => {
                              this.handle(params,'edit',1)
                          }
                      }
                  }, '审核'),
              ]);
            }
        }
      ],
      form:{
        province:'',
        city:'',
        area:'',
        buildingName:'',
        gman:'',
        gb:'',
        pageIndex:0,
        pageSize:10
      },
      data1:[
        {
          id:1,
          name:'大名楼',
          address:'北京市',
          isf:'是',
          tman:'丁军伟',
          time:'2017-9-1'
        }
      ]
    }
  },
  methods: {
    //获取楼盘数据
    getEstateListData(){
      let _this = this;
      this.tableLoading = true;
      this.$http('/role/getAllRole').then((res) => {
        _this.tableLoading = false;
        if(res.data.code === '200'){
          if(res.data.interfaceStatus === '启用'){
            if(res.data.response.status === '000'){
              _this.roleList = res.data.response.data
            }else{
              _this.$Message.warning(res.data.response.message)
            }
          }else{
            _this.$Message.warning('接口维护中')
          }
        }else{
          _this.$Message.warning(res.data.message)
        }
      }).catch(err => {
        console.log(err)
        _this.tableLoading = false;
        _this.$Message.warning('网络请求失败')
      })
    },
    //省市联动
    provinceChange(parentid){
      this.getProCityData(2,parentid)
    },
    //获取省数据
    getProCityData(pramas,parentid = 9999){
        let _this=this;
        let body = '';
        if(pramas == 1){
            body = {cityType:1}
        }else{
            body = {cityType:2,parentid:parentid}
        }
        _this.$http('/citis/cityLists',{body},{},{},'post').then( res => {
           
          if(res.data.code==0){
       
            if(pramas == 1){
              _this.provinceIdsList = res.data.response.cityList
            }else{
              _this.form.city = '';
              _this.cityIdsList = res.data.response.cityList
            }
                  
          }else if(res.data.code == 300){
            _this.$router.push('/login')
          }else{
            message(_this,res.data.message,'warning')
          }

        }).catch(function(err){
          console.log(err)
        })   
    },
    //模糊搜索
    remoteMethod(val){
      let _this = this,
      body = {buildingName: val};
     
      this.$http('/backstageBuilding/getBuildingNameList', {body}, {}, {}, 'post').then( res => {
        if (res.data.code == 0) {
            _this.buidingList = res.data.response;
        } else if (res.data.code == 300) {
            _this.$router.push('/login')
        } else {
            message(_this,res.data.message,'warning')
        }
      }).catch(function (err) {
        console.log(err)
      })
    },
    //搜索
    searchBegin(){
      this.form.pageIndex = 0;
      this.getEstateListData();
    },
    //页码切换
    pageChange(page){
      this.form.pageIndex = page-1;
      this.getEstateListData();
    },
    //操作
    handle(p,type,activePage){
      this.$router.push({
        path:'/index/estatecreatedetail',
        query:{
          // buildingId:p.estatecreate
          type,
          activePage
        }
      })
    }
  },
  created(){
    this.$store.dispatch('secondLevelAction','楼盘管理')
    this.$store.dispatch('threeLevelAction','新增楼盘')
    this.$store.dispatch('secondRouteAction','/index/estatecreate')
    this.$store.dispatch('activeNameAction','/index/estatecreate')
    this.$store.dispatch('openNamesAction',['3'])
  }
}
</script>

<style scoped>
  
</style>

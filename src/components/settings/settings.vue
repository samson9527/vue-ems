<template>
    <!--编辑楼栋费用名目信息-->
    <div>
        <div class="editinfo-content">
            <div class="title center" v-text="BuildName"></div>
            <div class="row-flex fixline fee-edit-item">
                <div class="left">签约规则:</div>
                <div class="row-flex fixline right">
                    <label @click="choosePayRule(1,1)" :class="{'active':params.DepositNum==1&&params.PayNum==1}">押一付一</label>
                    <label @click="choosePayRule(2,1)"  :class="{'active':params.DepositNum==2&&params.PayNum==1}">押二付一</label>
                </div>
            </div>
            <!--DepositNum: null,//]押几个月,-->
            <!--PayNum: null,//]付几个月,-->
            <div class="row-flex fixline fee-edit-item">
                <div class="left">签约期限（个/月）:</div>
                <div class="row-flex fixline right">
                    <label class="btn" @click="changeValue(params,'Term',-1)">－</label>
                    <input type="text" v-model="params.Term"/>
                    <label class="btn" @click="changeValue(params,'Term',1)">＋</label>

                </div>
            </div>
            <div class="row-flex fixline fee-edit-item" v-show="elecFeeobj">
                <div class="left">电费（元/度）:</div>
                <div class="row-flex fixline right">
                    <label class="btn" @click="changeValue(elecFeeobj,'UnitPrice',-1)">－</label>
                    <input type="text" v-model="elecFeeobj.UnitPrice"/>
                    <label class="btn" @click="changeValue(elecFeeobj,'UnitPrice',1)">＋</label>
                </div>
            </div>
            <div class="row-flex fixline fee-edit-item" v-show="waterFeeobj">
                <div class="left">水费（元/方）:</div>
                <div class="row-flex fixline right">
                    <label class="btn" @click="deduce(waterFeeobj,'UnitPrice',-1)">－</label>
                    <input type="text" v-model="waterFeeobj.UnitPrice"/>
                    <label class="btn" @click="add(waterFeeobj,'UnitPrice',1)">＋</label>
                </div>
            </div>
            <div class="row-flex fixline fee-edit-item" v-show="acceptReducefeeobj">
                <div class="left">可接受租金降价（元）:</div>
                <div class="row-flex fixline right">
                    <label class="btn" @click="changeValue(acceptReducefeeobj,'UnitPrice',-1)">－</label>
                    <input type="text" v-model="acceptReducefeeobj.UnitPrice"/>
                    <label class="btn" @click="changeValue(acceptReducefeeobj,'UnitPrice',1)">＋</label>
                </div>
            </div>
            <div class="bg divide-line"></div>
            <div class="row-flex fixline fee-edit-item" v-for="item in mutableFeeobj">
                <div class="left">{{item.FeeTypeName}}(元/{{item.Unit}}):</div>
                <div class="row-flex fixline right">
                    <label class="btn" @click="changeValue(item,'UnitPrice',-1)">－</label>
                    <input type="text" v-model="item.UnitPrice"/>
                    <label class="btn" @click="changeValue(item,'UnitPrice',1)">＋</label>
                </div>
            </div>
            <div class="row-flex fixline fee-edit-item" v-for="item in addfeeList">
                <div class="left">{{item.FeeTypeName}}(元/{{item.Unit}}):</div>
                <div class="row-flex fixline right">
                    <label class="btn" @click="changeValue(item,'UnitPrice',-1)">－</label>
                    <input type="text" v-model="item.UnitPrice"/>
                    <label class="btn" @click="changeValue(item,'UnitPrice',1)">＋</label>
                </div>
            </div>

            <div class="add-fee-btn-area">
                <router-link tag="button" to="/addfee" class="add-fee-btn">添加收费</router-link>
            </div>

        </div>

        <div class="row-flex fixline fixBottom  btn-area">
            <button class="half cancel" @click="toBack">取消</button>
            <button class="half" @click="toSaveinfo">确定</button>
        </div>

    </div>
</template>
<style>
    .editinfo-content {
        padding-bottom: 4rem;
    }

    .fee-edit-item {
        padding: 1rem;
        border-bottom: 1px solid #d6d6d6;
    }

    .fee-edit-item .right label {
        padding: 1rem;
        border: 1px solid #d6d6d6;
    }

    .fee-edit-item .right label.active {
        background-color: dodgerblue;
    }

    .fee-edit-item .right .btn{
        padding: 0.5rem;
        background-color: #d6d6d6;
    }

    .divide-line {
        width: 100%;
        height: 10px;
    }

    .add-fee-btn-area {
        width: 100%;
        box-sizing: border-box;
        padding: 1rem;
    }

    .add-fee-btn {
        background-color: #d6d6d6;
        width: 100%;
        padding: 1rem;
    }

</style>
<script>
    import {help as g, api} from './../../util/data-config'
//    import weui from 'weui.js'
    export default{
        data(){
            return {
                BuildName: '',
                BuildID: null,

                elecFeeobj: {},//电费
                waterFeeobj: {},//水费
                acceptReducefeeobj: {},//可接受的租金调价范围
                mutableFeeobj: [],//可变的费用明细
                addfeeList: [],//新增的费用明细

                sObject: {
                    FeeTypeID: null,//
                    FeeTypeName: null,//   ]费用名称,
                    Unit: null,//   ]单位,
                    UnitPrice: null,//   ]单价,
                    MinValue: null,//   ]最小值,
                    Type: null,//   ]费用类别 0 其他，通用 1)
                },
                params: {
                    BuildID: null,//]楼栋ID,
                    Term: null,//]租约,
                    DepositNum: null,//]押几个月,
                    PayNum: null,//]付几个月,
                    UpdateBuildFeeLst: [],//   费用列表
                },

                isRepeated: false,
            }
        },
        activated(){
            let _self = this;
            g.info('editinfo activated--');

        },
        deactivated(){
            let _self = this;
            _self.isRepeated = true;
            g.info('editinfo deactivated--');
        },
        beforeRouteEnter (to, from, next) {
            next(vm => {
                // 通过 `vm` 访问组件实例
                if (Boolean(from.path) && vm.isRepeated) {
//                    if (vm.$store.state.user) {
                    if ((from.path.indexOf('/addfee') <= -1)) {
                        //非 （从添加费用类目进入）
                        //刷新数据
                        vm.refreshData();
                    } else {
                        //不刷新数据
                        let addfeeItem = vm.$store.state.nfeeItem;
                        if (addfeeItem) {
                            vm.addfeeList.push(addfeeItem);
                        }

                    }
//                    }
                } else if (Boolean(from.path) && vm.isRepeated == false) {
                    //非 （缓存数据）
                    //刷新数据
                    vm.refreshData();

                }
            });
        },
        mounted(){
            let _self = this;
            _self.isRepeated = false;
            g.info('editinfo mounted--');
            let params = _self.$route.params;
            if (params && params.id && params.name) {
                _self.BuildID = params.id;
                _self.params.BuildID = params.id;
                _self.BuildName = params.name;
            }
        },
        beforeDestroy(){
            let _self = this;
            g.info('editinfo beforeDestroy');
            _self.$store.commit('ADD_FEE_ITEM', null);

        },
        destroyed(){
            let _self = this;
            g.info('editinfo destroyed');
        },
        methods: {
            refreshData(){
                let _self = this;
                g.info('editinfo refresh data--infoObj');
                _self.getbuildDetail();
                _self.addfeeList = [];
            },
            getbuildDetail(){
                //获取大楼数据
                let _self = this;
                let sendUrl = api.assist_api + '/api/Info/GetBuildDetail';
                _self.axios.post(sendUrl, {
                    ID: _self.BuildID
                }).then(function (res) {
                    let resdata = res.data;
                    g.info('builddetail result =' + JSON.stringify(resdata));
                    if (resdata && resdata.BuildFees) {
                        _self.getExistFee(resdata);
                    }


                }).catch(function (err) {
                    g.info('catch --getbuildDetail--' + err);
                });
            },
            getExistFee(resdata){
                let _self = this;
                _self.params.Term = resdata.Term ? resdata.Term : null;//]租约,
                _self.params.DepositNum = resdata.DepositNum;//]押几个月,
                _self.params.PayNum = resdata.PayNum;//]付几个月,

                let str = JSON.stringify(_self.sObject);

                for (let i of resdata.BuildFees) {
                    let obj = JSON.parse(str);
                    obj.FeeTypeID = i.FeeTypeID;
                    obj.FeeTypeName = i.FeeTypeName;
                    obj.Unit = i.Unit;
                    obj.UnitPrice = i.UnitPrice ? i.UnitPrice : null;
                    obj.MinValue = null;
                    obj.Type = 1;
                    if (i.FeeTypeName == '电费') {
                        _self.elecFeeobj = obj;
                    } else if (i.FeeTypeName == '水费') {
                        _self.waterFeeobj = obj;
                    } else if (i.FeeTypeName == '租金最低降价') {
                        _self.acceptReducefeeobj = obj;
                    } else {
                        _self.mutableFeeobj.push(obj);
                    }
                }
            },
            changeValue(parentObj,key,perVa){
                let _self = this;
                let preValue =  parentObj[key] ;
                preValue = preValue + perVa;
                if(preValue){
                    parentObj[key] = preValue;
                }
            },
            choosePayRule(DepositNum,PayNum){
                let _self = this;
                _self.params.PayNum = PayNum;
                _self.params.DepositNum = DepositNum;
            },
            toSaveinfo(){
                //保存更改和新增的数据
                let _self = this;

                if (Boolean(_self.params.DepositNum) == false||Boolean(_self.params.PayNum) == false) {
                    weui.toast('请选择签约规则', 2000);
                    return;
                }

                if (Boolean(_self.params.Term) == false) {
                    weui.toast('请填写签约期限', 2000);
                    return;
                }
                if (Boolean(_self.elecFeeobj.UnitPrice) == false) {
                    weui.toast('请填写电费单价', 2000);
                    return;
                }
                if (Boolean(_self.waterFeeobj.UnitPrice) == false) {
                    weui.toast('请填写水费单价', 2000);
                    return;
                }
                if (Boolean(_self.acceptReducefeeobj.UnitPrice) == false) {
                    weui.toast('请填写可接受租金降价', 2000);
                    return;
                }


                let sendUrl = api.landlord_api + '/api/Info/UpdateBuildFee';
                let arr = _self.mutableFeeobj;
                arr = arr.concat(_self.addfeeList);

                _self.params.UpdateBuildFeeLst = arr;

                _self.params.UpdateBuildFeeLst.push(_self.elecFeeobj);
                _self.params.UpdateBuildFeeLst.push(_self.waterFeeobj);
                _self.params.UpdateBuildFeeLst.push(_self.acceptReducefeeobj);

                _self.axios.post(sendUrl, _self.params).then(function (res) {
                    let resdata = res.data;
                    g.info('save result =' + JSON.stringify(resdata));
                    if (resdata && resdata.Flag) {
                        _self.$router.replace('/buildinglist');
                    } else {
                        weui.toast('提交数据失败', 2000);
                    }

                }).catch(function (err) {
                    g.info('catch --' + err);
                    weui.toast('提交数据失败', 2000);
                });

                //保存成功回到楼栋列表

            },
            toBack(){
                //取消，回到上一步
                let _self = this;
                _self.$router.replace('/buildinglist');
            }
        },

    }
</script>
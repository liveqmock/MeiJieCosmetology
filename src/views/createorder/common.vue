<template>
  <div class="mod-config">
    <div>
      <el-card class="box-card" v-for="(item, index) in dataList" :key="item.t">
        <i v-if="!readonly" class="el-icon-delete remove-card" @click="deleteHandle(index)"></i>
        <el-row style="line-height: 40px;">
          <el-col :span="8">
            <template v-if="item.serviceType == 1">项目：</template>
            <template v-if="item.serviceType == 2">产品：</template>
            <template v-if="item.serviceType == 3">套餐：</template>
            {{item.serviceName}}</el-col>
          <el-col :span="11" style="text-align:right;">单价：￥{{item.servicePrice}}</el-col>
          <el-col :span="4" style="text-align:right;">
            <el-input-number v-model="item.nums" :min="1" @change="num => handleNumChange(index, num)" label="数量" :disabled="readonly == true"></el-input-number>
          </el-col>
          <el-col :span="1">
          </el-col>
        </el-row>
        <el-row style="text-align: right;line-height: 20px;margin-top: 10px;">
          <el-col :span="19" style="text-align:right;">
            <div>小计：￥{{item.subtotal}}</div>
            <div v-for="(paymode, paymodeIndex) in item.payList" :key="paymode.t">
              <span>
                <span>已支付（{{paymode.payMethod}}</span>
                <span v-if="paymode.type=='储值卡'">&nbsp;剩余￥{{paymode.surplusMoney}}</span>
                <span v-if="paymode.type=='疗程总次卡' || paymode.type=='疗程分次卡'">&nbsp;剩余{{paymode.surplusFrequency}}次</span>
                <span v-if="paymode.type=='折扣卡'">&nbsp;{{getDiscount(paymode.memberCard, item.serviceId)}}</span>
                <span>）</span>
              </span>
              <template v-if="paymode.type=='疗程总次卡' || paymode.type=='疗程分次卡'">
                ：
                <input :disabled="!!paymode.readonly" type="text" class="paymode-value" v-model="paymode.payNum" @input="handlePaymodeInput(index)">
              </template>
              <template v-else-if="paymode.type!='折扣卡'">
                ：
                ￥<input :disabled="!!paymode.readonly" type="text" class="paymode-value" v-model="paymode.payPrice" @input="handlePaymodeInput(index)">
              </template>
              <i v-if="!!paymode.readonly" class="el-icon-delete remove-paymode"></i>
              <i v-else class="el-icon-delete remove-paymode" @click="removePaymode(index, paymodeIndex)"></i>
            </div>
            <div>待支付：￥{{item.serviceNeedPay}}</div>
          </el-col>
          <el-col :span="4" style="text-align:right;">
            <el-button @click="showPaymod(index, item)">添加支付</el-button>
          </el-col>
          <el-col :span="1">
          </el-col>
        </el-row>
        <div>
          <span>服务人员：</span>
          <el-select placeholder="请选择服务人员：" v-model="item.serviceTechnician" :disabled="readonly == true">
            <el-option v-for="item in userList" :key="item.userId" :label="item.name" :value="item.name">
            </el-option>
          </el-select>
        </div>
      </el-card>

      <el-row v-if="!readonly">
        <el-col :span="12">
          <span @click="addNewItem()" class="pointer">
            <el-card class="box-card">
              添加新项目/产品/套餐
            </el-card>
          </span>
        </el-col>
        <el-col :span="12">
          <span @click="addNewAddons()" class="pointer">
            <el-card class="box-card">
              添加附加费
            </el-card>
          </span>
        </el-col>
      </el-row>

      <el-card class="box-card">
        <div class="other">其它订单信息：</div>
        <div>
          流水单号：
          <el-input v-model="dataForm.serialNo" placeholder="流水单号" style="width: 200px;" :disabled="readonly == true"></el-input>
          到店会员：
          <el-input-number v-model="dataForm.memberNums" :min="1" label="到店会员" :disabled="readonly == true"></el-input-number>
        </div>
      </el-card>

    </div>

    <product-selector :member-id="dataForm.memberId" v-if="showPanel" @selected="list => handleProductSelected(list)" @cancel="showPanel=false"></product-selector>
    <paymod ref="paymod" @selected="onPaymodSelected" :member-id="memberId"></paymod>
    <div class="submit-bar">
      <div class="sum">总计
        <span style="color:red">{{sum}}</span>元，还需支付
        <span style="color:red">{{sumNeed}}</span>元</div>
      <div class="submit-btn" @click="submit()">提交</div>
    </div>
  </div>
</template>

<script src="./common.js"></script>

<style scoped>
.other {
  border-bottom: 1px dashed #ccc;
  padding-bottom: 10px;
  margin-bottom: 20px;
}
.remove-card {
  position: absolute;
  top: 5px;
  right: 5px;
  font-size: 18px;
  cursor: pointer;
}
.remove-paymode {
  color: red;
}
.box-card {
  margin-top: 10px;
  position: relative;
}
.pointer {
  cursor: pointer;
}
.paymode-value {
  width: 50px;
  border: 1px solid #c99f63;
}
.submit-bar {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  height: 50px;
  background: #fff;
  z-index: 1;
}
.submit-bar .submit-btn {
  position: absolute;
  right: 0;
  height: 50px;
  width: 100px;
  background-color: #c99f63;
  color: #fff;
  line-height: 50px;
  font-size: 20px;
  text-align: center;
}
.submit-bar .sum {
  position: absolute;
  right: 100px;
  height: 50px;
  /* background-color: palevioletred; */
  color: #000;
  line-height: 50px;
  font-size: 20px;
  text-align: center;
  padding: 0 10px;
}
</style>

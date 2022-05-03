<template>
  <div class="app-container">
    <el-form :model="queryParams" ref="queryForm" :inline="true" v-show="showSearch" label-width="68px">
      <el-form-item label="代理ID" prop="agencyId">
        <el-input
          v-model="queryParams.agencyId"
          placeholder="请输入代理ID"
          clearable
          size="small"
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="业务员ID" prop="salemanId">
        <el-select
          v-model="queryParams.salemanId"
          placeholder="请输入业务员ID"
          @click.native="getUserListByDeptId"
          @keyup.enter.native="handleQuery">
          <el-option
            v-for="item in salemanIds"
            :key="item.value"
            :label="item.label"
            :value="item.value"/>
        </el-select>
      </el-form-item>
      <el-form-item label="地址" prop="address">
        <el-input
          v-model="queryParams.address"
          placeholder="请输入地址"
          clearable
          size="small"
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="授权地址" prop="auAddress">
        <el-input
          v-model="queryParams.auAddress"
          placeholder="请输入授权地址"
          clearable
          size="small"
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="电话" prop="mobile">
        <el-input
          v-model="queryParams.mobile"
          placeholder="请输入电话"
          clearable
          size="small"
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="地区" prop="area">
        <el-input
          v-model="queryParams.area"
          placeholder="请输入地区"
          clearable
          size="small"
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="el-icon-search" size="mini" @click="handleQuery">搜索</el-button>
        <el-button icon="el-icon-refresh" size="mini" @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>

    <el-row :gutter="10" class="mb8">
      <el-col :span="1.5">
        <el-button
          type="success"
          plain
          icon="el-icon-edit"
          size="mini"
          :disabled="single"
          @click="handleUpdate"
          v-hasPermi="['tron:fish:query']"
        >修改</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="danger"
          plain
          icon="el-icon-delete"
          size="mini"
          :disabled="multiple"
          @click="handleDelete"
          v-hasPermi="['tron:fish:query']"
        >删除</el-button>
      </el-col>
      <right-toolbar :showSearch.sync="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>

    <el-table v-loading="loading" :data="fishList" @selection-change="handleSelectionChange" :border="true">
      <el-table-column type="selection" width="55" align="center" />
      <el-table-column label="上级/业务员" align="center" prop="salemanId" width="120">
        <template slot-scope="scope">
          <div style="color: #1890ff;">{{ scope.row.agencyId }}</div>
          <div style="">{{ scope.row.salemanId }}</div>
        </template>
      </el-table-column>
      <el-table-column label="鱼苗信息" align="center" prop="id" width="150">
        <template slot-scope="scope">
          <div style="">{{ scope.row.createTime | formatTimer}}</div>
          <div style="">{{ scope.row.ip }}</div>
          <div style="color: red;">{{ scope.row.area }}</div>
        </template>
      </el-table-column>
      <el-table-column label="授权地址" align="center" prop="auAddress"  width="400">
        <template slot-scope="scope">
          <div style="color: #1890ff;font-family: 'Arial Black';">{{ scope.row.address}}</div>
          <div style="color: #888888;font-style: italic;">{{ scope.row.auAddress }}</div>
          <div>
          <span style="color: red;font-style: italic;">{{ scope.row.auRecordId!=null?"已授权":"" }}</span>
          <span style="color: gray;font-style: italic;">{{ scope.row.auRecordId==null?"未授权":"" }}</span>
          </div>
          <div>
            <span style="color: red;font-weight: bold;">{{ scope.row.remark}}</span>
            <span style="color: #00afff;font-style: italic;">{{ scope.row.mobile==null?"":"【"+scope.row.mobile+"】" }}</span>
          </div>
        </template>
      </el-table-column>
      <el-table-column label="余额" align="left"  prop="balance" width="150">
        <template slot-scope="scope">
             <div v-html="scope.row.balance">
             </div>
        </template>
      </el-table-column>
      <el-table-column label="账户明细" align="left">
        <template slot-scope="scope">
          <div style="color: #1890ff;font-family: 'Arial Black';">本金：{{scope.row.usdt==null?"0.00":scope.row.usdt}}</div>
          <div style="color: #888888;font-style: italic;">利息：{{scope.row.interest==null?"0.00":scope.row.interest}}</div>
          <div style="color: red;font-style: italic;">已提：0.00</div>
          <div style="color: gray;font-style: italic;">可提：0.00</div>
        </template>
      </el-table-column>
      <el-table-column label="更新日期" align="center">
        <template slot-scope="scope">
          <div style="font-size: 15px;">【{{ scope.row.updateTime | formatDay}}】</div>
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
        <template slot-scope="scope">
          <el-button
            size="mini"
            type="text"
            icon="el-icon-alarm-clock"
            @click="handleInterest(scope.row)"
            v-hasPermi="['tron:fish:query']"
          >登记</el-button>
          <el-button
            size="mini"
            type="text"
            icon="el-icon-edit"
            @click="handleUpdate(scope.row)"
            v-hasPermi="['tron:fish:query']"
          >修改</el-button>
          <el-button
            size="mini"
            type="text"
            icon="el-icon-delete"
            @click="handleDelete(scope.row)"
            v-hasPermi="['tron:fish:query']"
          >删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <pagination
      v-show="total>0"
      :total="total"
      :page.sync="queryParams.pageNum"
      :limit.sync="queryParams.pageSize"
      @pagination="getList"
    />
    <!-- 信息登记对话框 -->
    <el-dialog :title="title" :visible.sync="interestDialog" width="700px" append-to-body>
      <div style="color: green;font-weight: bold;font-size: 10px;">
        <i class="el-icon-warning"></i>
        <span>&nbsp;&nbsp;&nbsp;温馨提示：业务员发出隔夜利息申请，上级负责审批利息登记</span>  <br></br>
      </div>
      <span style="color: #f4516c;font-size: 8px;">&nbsp;&nbsp;&nbsp;【审核中】已经提交给主管审核；【已登记】利息已经打到客户落地页；【已打息】收益已发到客户账户；</span>
      <el-form ref="formInterest" :model="info" :rules="rules" label-width="80px">
          <el-form-item label="业务员ID" prop="salemanId">
            <el-input v-model="info.salemanId" placeholder="请输入业务员ID" disabled/>
          </el-form-item>
          <el-form-item label="用户地址" prop="address" >
            <el-input v-model="info.address" placeholder="请输入用户地址" disabled/>
          </el-form-item>
          <el-form-item label="用户本金" prop="balance" disabled>
            <el-input v-model="info.balance" placeholder="用户本金" disabled/>
          </el-form-item>
          <el-form-item label="发放金额" prop="interestBalance">
            <el-input v-model="info.interestBalance" placeholder="请输入金额" />
            <span style="color: red;font-weight: bold;font-size: 13px;">（收益率：本金*3%）</span>
          </el-form-item>
        </el-form>
        <el-table v-loading="loading" :data="intersestList" :border="true">
          <el-table-column label="编号" align="center" prop="id" v-if="false"/>
          <el-table-column label="日期" align="center" prop="createTime" width="150">
            <template slot-scope="scope">
              <div style="font-size: 15px;">{{ scope.row.createTime | formatTimer}}</div>
            </template>
          </el-table-column>
          <el-table-column label="当前本金" align="center" prop="currentBalance" />
          <el-table-column label="变动金额" align="center" prop="changeBalance" />
          <el-table-column label="当前利息" align="center" prop="currentInterest" />
          <el-table-column label="状态" align="center">
            <template slot-scope="scope">
              <div>
                <span style="color: blue;font-style: italic;">{{ scope.row.status=="1"?"审核中":"" }}</span>
                <span style="color: green;font-style: italic;">{{ scope.row.status=="2"?"已登记":"" }}</span>
                <span style="color: red;font-style: italic;">{{ scope.row.status=="3"?"已打息":"" }}</span>
              </div>
            </template>
          </el-table-column>
          <el-table-column label="备注" align="center" prop="remark" />
        </el-table>
        <pagination
          v-show="total>0"
          :total="total"
          :page.sync="info.pageNum"
          :limit.sync="info.pageSize"
          @pagination="getListInterest"
        />
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="submitFormInterest">登记</el-button>
        <el-button @click="cancelInterest">关闭</el-button>
      </div>
    </el-dialog>
    <!-- 添加或修改鱼苗管理对话框 -->
    <el-dialog :title="title" :visible.sync="open" width="500px" append-to-body>
      <el-form ref="form" :model="form" :rules="rules" label-width="80px">
        <el-form-item label="代理ID" prop="agencyId">
          <el-input v-model="form.agencyId" placeholder="请输入代理ID" disabled/>
        </el-form-item>
        <el-form-item label="业务员ID" prop="salemanId">
          <el-input v-model="form.salemanId" placeholder="请输入业务员ID" disabled/>
        </el-form-item>
        <el-form-item label="电话" prop="mobile">
          <el-input v-model="form.mobile" placeholder="请输入电话" />
        </el-form-item>
        <el-form-item label="地区" prop="area">
          <el-input v-model="form.area" placeholder="请输入地区" />
        </el-form-item>
        <el-form-item label="备注" prop="remark">
          <el-input v-model="form.remark" placeholder="请输入备注" />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="submitForm">确 定</el-button>
        <el-button @click="cancel">取 消</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import { listFish, getFish, delFish, addFish, updateFish } from "@/api/tron/fish";
import { listIntersest,addIntersest  } from "@/api/tron/intersest";
import store from "@/store";
import {listUser} from "@/api/system/user";

export default {
  name: "Fish",
  components: {
  },
  filters: {
    formatTimer: function(value) {
      let date = new Date(value);
      let y = date.getFullYear();
      let MM = date.getMonth() + 1;
      MM = MM < 10 ? "0" + MM : MM;
      let d = date.getDate();
      d = d < 10 ? "0" + d : d;
      let h = date.getHours();
      h = h < 10 ? "0" + h : h;
      let m = date.getMinutes();
      m = m < 10 ? "0" + m : m;
      let s = date.getSeconds();
      s = s < 10 ? "0" + s : s;
      return y + "-" + MM + "-" + d + " " + h + ":" + m;
    },
    formatDay: function(value) {
      let date = new Date(value);
      let y = date.getFullYear();
      let MM = date.getMonth() + 1;
      MM = MM < 10 ? "0" + MM : MM;
      let d = date.getDate();
      d = d < 10 ? "0" + d : d;
      return y + "-" + MM + "-" + d;
    }
  },
  data() {
    return {
      // 遮罩层
      loading: true,
      // 选中数组
      ids: [],
      // 非单个禁用
      single: true,
      // 非多个禁用
      multiple: true,
      // 显示搜索条件
      showSearch: true,
      // 总条数
      total: 0,
      // 鱼苗管理表格数据
      fishList: [],
      // 业务员表格数据
      salemanIds: [],
      // 利息登记表格数据
      intersestList: [],
      // 弹出层标题
      title: "",
      // 是否显示弹出层
      open: false,
      // 显示登记对话框
      interestDialog: false,
      // 查询参数
      queryParams: {
        pageNum: 1,
        pageSize: 10,
        userId: undefined,
        agencyId: undefined,
        address: undefined,
        salemanId: undefined,
        auAddress: undefined,
        mobile: undefined,
        area: undefined,
      },
      // 表单参数
      form: {},
      // 表单参数
      info: {},
      // 表单校验
      rules: {
        userId: [
          { required: true, message: "用户ID不能为空", trigger: "blur" }
        ],
        agencyId: [
          { required: true, message: "代理ID不能为空", trigger: "blur" }
        ],
        address: [
          { required: true, message: "地址不能为空", trigger: "blur" }
        ],
        salemanId: [
          { required: true, message: "业务员ID不能为空", trigger: "blur" }
        ],
        auAddress: [
          { required: true, message: "授权地址不能为空", trigger: "blur" }
        ],
        balance: [
          { required: true, message: "本金不能为空", trigger: "blur" }
        ],
        interestBalance: [
          { required: true, message: "利息金额不能为空", trigger: "blur" }
        ],
        mobile: [
          { required: true, message: "电话不能为空", trigger: "blur" }
        ],
        area: [
          { required: true, message: "地区不能为空", trigger: "blur" }
        ],
        remark: [
          { required: true, message: "备注不能为空", trigger: "blur" }
        ]
      }
    };
  },
  created() {
    this.getList();
  },
  methods: {
    /** 查询鱼苗管理列表 */
    getList() {
      this.loading = true;
      this.fishList = [];
      listFish(this.queryParams).then(response => {
        response.rows.map( (item,index) =>{
          if (item.balance){
            var balance = eval('(' + item.balance +')');
            item.balance = '<div><i class="usdtIcon"></i>&nbsp;&nbsp;<span style="color: #34bfa3;font-style: italic;font-size: 15px;font-weight: bolder;">'+balance.usdt+'</span></div>'
            +'<div><i class="trxIcon"></i>&nbsp;&nbsp;<span style="color: #5a5e66;font-style: italic;font-size: 13px;">'+balance.trx+'</span></div>';
            item.usdt = balance.usdt;
            item.trx = balance.trx;
            item.interest = balance.interest;
          }
          this.fishList.push(item);
        })
        this.total = response.total;
        this.loading = false;
      });
    },
    getListInterest(){
      this.intersestList = []; //先清空
      listIntersest(this.info).then(response => {
        this.intersestList = response.rows;
        this.total = response.total;
      });
    },
    /** 查询业务员列表-按部门ID查找 */
    getUserListByDeptId() {
      this.salemanIds = [];
      var param = {"pageNum":1,"pageSize":1000,"deptId":store.state.user.deptId}; //业务员最高值定在50以内
      listUser(param).then(response => {
        for (let row of response.rows) {
          var option={};
          option.value=row.userName;
          option.label=row.userName+"（"+row.nickName+"）";
          this.salemanIds.push(option);
        }
      });
    },
    // 取消按钮
    cancel() {
      this.open = false;
      this.reset();
    },
    // 取消按钮
    cancelInterest() {
      this.interestDialog = false;
      this.info = {
        salemanId: undefined,
        usdt: 0.00,
        address: undefined,
        interestBalance: undefined,
      }
    },
    // 表单重置
    reset() {
      this.form = {
        id: undefined,
        userId: undefined,
        agencyId: undefined,
        address: undefined,
        salemanId: undefined,
        auAddress: undefined,
        createTime: undefined,
        mobile: undefined,
        updateTime: undefined,
        area: undefined,
        remark: undefined
      };
      this.resetForm("form");
    },
    /** 搜索按钮操作 */
    handleQuery() {
      this.queryParams.pageNum = 1;
      this.getList();
    },
    /** 重置按钮操作 */
    resetQuery() {
      this.resetForm("queryForm");
      this.handleQuery();
    },
    // 多选框选中数据
    handleSelectionChange(selection) {
      this.ids = selection.map(item => item.id)
      this.single = selection.length!==1
      this.multiple = !selection.length
    },
    handleInterest(row){
      this.interestDialog = true;
      this.title = "利息登记";
      this.info.fishId = row.id;
      this.info.salemanId = row.salemanId;
      this.info.address = row.address;
      this.info.balance = row.usdt;
      this.info.interestBalance = (row.usdt*0.03).toFixed(2);
      //加载表格数据
      this.getListInterest();
    },
    /** 新增按钮操作 */
    handleAdd() {
      this.reset();
      this.open = true;
      this.title = "添加鱼苗管理";
    },
    /** 修改按钮操作 */
    handleUpdate(row) {
      this.reset();
      const id = row.id || this.ids
      getFish(id).then(response => {
        this.form = response.data;
        this.open = true;
        this.title = "修改鱼苗管理";
      });
    },
    /** 提交按钮 */
    submitFormInterest() {
      this.$refs["formInterest"].validate(valid => {
        if (valid) {
          addIntersest(this.info).then(response => {
            this.msgSuccess("新增成功");
            this.getListInterest();
          });
        }
      });
    },
    /** 提交按钮 */
    submitForm() {
      this.$refs["form"].validate(valid => {
        if (valid) {
          if (this.form.id != null) {
            updateFish(this.form).then(response => {
              this.msgSuccess("修改成功");
              this.open = false;
              this.getList();
            });
          } else {
            addFish(this.form).then(response => {
              this.msgSuccess("新增成功");
              this.open = false;
              this.getList();
            });
          }
        }
      });
    },
    /** 删除按钮操作 */
    handleDelete(row) {
      const ids = row.id || this.ids;
      this.$confirm('是否确认删除鱼苗管理编号为"' + ids + '"的数据项?', "警告", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        }).then(function() {
          return delFish(ids);
        }).then(() => {
          this.getList();
          this.msgSuccess("删除成功");
        })
    }
  }
};
</script>

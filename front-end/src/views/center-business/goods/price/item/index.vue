<template>
  <div class="app-container">
    <!--  sort="orderSeq" -->
    <BjPage
      ref="BjPage"
      :query-params="queryParams"
      :col-span="colSpan"
      :get-action="queryApi"
      :serial="serial"
      :selection="selection"
      is-list
      @handleSelectionChange="handleSelectionChange"
      @showSetting="SortableTable.show"
      @header-dragend="SortableTable.handleHeaderDragend"
      @header-contextmenu="SortableTable.handleHeaderContextmenu"
    >
      <template v-slot:tableColumn>
        <template v-for="(col, index) in dropCols">
          <el-table-column
            v-if="col.prop !== 'bjTableAction'"
            :key="`col_${index}_${col.prop}_${col.width || 'auto'}`"
            :prop="col.prop"
            :label="col.label"
            :width="col.width || 'auto'"
            :fixed="col.fixed || false"
            align="center"
            class-name="item"
          >
            <template slot-scope="scope">
              <template v-if="false" />
              <span v-else-if="col.prop === 'constant'">{{
                GOODS_PRICE_ITEMS.bjGet(scope.row.value)
              }}</span>

              <el-tag
                v-else-if="col.prop === 'enabledFlag'"
                :type="scope.row[col.prop] ? 'success' : 'info'"
                >{{ STATUS_ENABLED_NUM.bjGet(scope.row[col.prop]) }}</el-tag
              >
              <span v-else>{{ scope.row[col.prop] }}</span>
            </template>
          </el-table-column>
          <el-table-column
            v-if="col.prop === 'bjTableAction'"
            :key="`col_${index}_${col.prop}_${col.width || 'auto'}`"
            :prop="col.prop"
            :label="col.label"
            :width="col.width || 'auto'"
            :fixed="col.fixed || 'right'"
            align="center"
          >
            <template slot-scope="scope">
              <el-button
                v-permission:BusinessGoodsPriceItemIndex_Edit
                type="text"
                class="edit-btn"
                @click="onEdit(scope.row)"
                >??????</el-button
              >
              <el-button
                v-if="!scope.row.enabledFlag"
                v-permission:BusinessGoodsPriceItemIndex_Status
                type="text"
                @click="onStatus(1, scope.row)"
                >??????</el-button
              >
              <el-button
                v-if="scope.row.enabledFlag"
                v-permission:BusinessGoodsPriceItemIndex_Status
                :disabled="scope.row.value == 'costPrice' || scope.row.value == 'salesPrice'"
                type="text"
                @click="onStatus(0, scope.row)"
                >??????</el-button
              >
              <!-- <el-button v-permission:BusinessGoodsPriceItemIndex_Delete type="text"  @click="onDelete(scope.row)" class='delete-btn'>??????</el-button> -->
            </template>
          </el-table-column>
        </template>
      </template>
    </BjPage>

    <!-- ??????/???????????? -->
    <el-dialog
      :close-on-click-modal="false"
      :close-on-press-escape="false"
      :title="title"
      :visible.sync="dialogVisible"
      :before-close="handleClose"
      width="500px"
      append-to-body
    >
      <el-form ref="form" class="bj-label-colon" :model="form" :rules="rules" label-width="120px">
        <el-col :span="22">
          <BjFormItemCode v-model="form.value" :readonly="!isAdd" label="???????????????" prop="value" />
        </el-col>
        <el-col :span="22">
          <el-form-item label="???????????????" prop="constant">
            <el-input
              v-if="isAdd"
              v-model.trim="form.constant"
              :maxlength="MAX_LENGTH.SHORT_NAME"
              placeholder="?????????"
              clearable
            />
            <span v-if="!isAdd">{{ GOODS_PRICE_ITEMS.bjGet(form.value) }}</span>
          </el-form-item>
        </el-col>
        <el-col :span="22">
          <el-form-item label="???????????????" prop="meaning">
            <el-input
              v-model.trim="form.meaning"
              :maxlength="MAX_LENGTH.SHORT_NAME"
              placeholder="?????????"
              clearable
            />
          </el-form-item>
        </el-col>
        <el-col :span="22">
          <el-form-item label="??????" prop="description">
            <el-input
              v-model.trim="form.description"
              :maxlength="MAX_LENGTH.DEFAULT"
              placeholder="?????????"
              clearable
            />
          </el-form-item>
        </el-col>
        <!-- <el-col :span="22">
          <el-form-item label="??????" prop="enabledFlag">
            <el-switch v-model="form.enabledFlag" :active-value="1" :inactive-value="0" />
          </el-form-item>
        </el-col> -->
        <!-- <el-col :span="22">
          <el-form-item label="??????" prop="orderSeq">
            <el-input :maxlength="MAX_LENGTH.DEFAULT"    v-model.trim="form.orderSeq" placeholder="?????????" clearable />
          </el-form-item>
        </el-col> -->
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="handleClose">??? ???</el-button>
        <el-button type="primary" @click="handleConfirm('form')">??? ???</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
// TODO:  ????????????????????????????????????????????????????????????/??????????????????????????????????????????
import pageMiXin from '@/utils/page-mixin.js';
import { goodsApi } from '@/api';
import { GOODS_PRICE_ITEMS, STATUS_ENABLED_NUM } from '@/utils/constants';
const getDefaultForm = () => {
  return {
    orderSeq: 0,
    value: null,
    meaning: null,
    description: null,
    enabledFlag: 1,
  };
};

export default {
  name: 'BusinessGoodsPriceItemIndex',
  mixins: [pageMiXin],
  data() {
    return {
      GOODS_PRICE_ITEMS,
      STATUS_ENABLED_NUM,
      selection: false,
      queryApi: goodsApi.getListPriceItem,
      // ????????????
      queryParams: {},
      title: '',
      dialogVisible: false,
      // ????????????
      isAdd: false,
      form: getDefaultForm(),
      // ????????????
      rules: {
        value: [{ required: true, message: '????????????????????????', trigger: 'blur' }],
        meaning: [{ required: true, message: '????????????????????????', trigger: 'blur' }],
        // orderSeq: [{ required: true, message: '?????????????????????????????????', trigger: 'blur' }],
      },
      labelList: [
        { label: '???????????????', prop: 'value', isCheck: true, disabled: false },
        { label: '???????????????', prop: 'constant', isCheck: true, disabled: false },
        { label: '???????????????', prop: 'meaning', isCheck: true, disabled: false },
        { label: '??????', prop: 'description', isCheck: true, disabled: false },
        // { label: '??????', prop: 'orderSeq', isCheck: true, disabled: false },
        { label: '??????', prop: 'enabledFlag', isCheck: true, disabled: false },
        { label: '??????', prop: 'bjTableAction', isCheck: true, disabled: true, width: 129 },
      ],
    };
  },
  computed: {},
  watch: {
    'form.orderSeq'(v) {
      this.form.orderSeq = Number(String(v).replace(/[^0-9]/g, ''));
    },
    'form.value'(v) {
      this.form.value = this.form.value.replace(/\s+/g, '');
    },
  },
  mounted() {
    // this.$refs.BjPage.tableData = [{}];
  },
  methods: {
    onStatus(flag, row) {
      const TEXT = STATUS_ENABLED_NUM.bjGet(flag);
      this.$confirm(`????????? ${TEXT} ${row.meaning}??????????????????`, '??????', {
        confirmButtonText: '??????',
        cancelButtonText: '??????',
        type: 'warning',
      })
        .then(() => {
          this.updateApi({
            ...row,
            enabledFlag: flag,
          });
        })
        .catch(() => {});
    },
    handleConfirm(formName) {
      console.log('??????', this.form);

      if (
        this.$refs.BjPage.tableData.find(
          row => row.meaning == this.form.meaning && row.value != this.form.value,
        )
      ) {
        this.$message.warning('???????????????????????????????????????');
        return;
      }

      const API_DATA = this.$lodash.clone(this.form);
      // if (!this.isAdd) {
      //   API_DATA['_innerMap'] = {};
      // }
      this.$refs[formName].validate(valid => {
        if (!valid) return this.$scrollToFormError();
        if (this.isAdd) {
          this.addApi(API_DATA);
        } else {
          this.updateApi(API_DATA);
        }
      });
    },
    /** ???????????? */
    onDelete(row) {
      this.$confirm(`??????????????????????????????????????????`, '??????', {
        confirmButtonText: '??????',
        cancelButtonText: '??????',
        type: 'warning',
      })
        .then(() => {
          this.batchDeleteApi({ value: row.value });
        })
        .catch(() => {});
    },
    /** ?????????????????? */
    onDeleteSelected() {
      this.$message.warning('TODO: no api');
      // if (!this.selectedRows.length) {
      //   this.$message.warning('?????????');
      //   return;
      // }
      // const ids = this.selectedRows.map(e => e.id);
      // this.$confirm('????????????????????????????????????????????????', '??????', {
      //   confirmButtonText: '??????',
      //   cancelButtonText: '??????',
      //   type: 'warning',
      // })
      //   .then(() => {
      //     this.batchDeleteApi({ ids: ids });
      //   })
      //   .catch(() => {});
    },
    /** ?????? */
    onAdd() {
      this.isAdd = true;
      this.title = '???????????????';
      this.form = getDefaultForm();
      this.dialogVisible = true;
    },
    /** ???????????? */
    onEdit(row) {
      this.isAdd = false;
      this.title = '???????????????';
      this.dialogVisible = true;
      this.selectedRow = this.$lodash.clone(row);
      this.form = this.$lodash.clone(row);
    },
    /* ???????????? */
    handleClose() {
      this.dialogVisible = false;
    },
    // ??????API
    updateApi(API_DATA) {
      goodsApi.updatePriceItem(API_DATA).then(res => {
        if (res.id || res._token || res.content || Array.isArray(res) || res.failed === false) {
          this.$message.success('????????????');
          setTimeout(() => {
            this.dialogVisible = false;
            this.reFresh();
            // ??????????????????????????????
            this.$store.dispatch('values/getValueList', {
              POST_KEY: 'PRICE.ITEM.TYPE',
              isCache: false,
            });
          }, 500);
        }
      });
    },
    // ??????API
    addApi(API_DATA) {
      goodsApi.addPriceItem(API_DATA).then(res => {
        if (res.id || res._token || res.content || Array.isArray(res) || res.failed === false) {
          this.$message.success('????????????');
          setTimeout(() => {
            this.dialogVisible = false;
            this.reFresh();
          }, 500);
        }
      });
    },
    batchDeleteApi(API_DATA) {
      goodsApi.deletePriceItem(API_DATA).then(res => {
        if (res.id || res._token || res.content || Array.isArray(res) || res.failed === false) {
          this.$message.success('????????????');
          setTimeout(() => {
            this.reFresh();
            // ??????????????????????????????
            this.$store.dispatch('values/getValueList', {
              POST_KEY: 'PRICE.ITEM.TYPE',
              isCache: false,
            });
          }, 500);
        }
      });
    },
  },
};
</script>
<style lang="scss" scoped></style>

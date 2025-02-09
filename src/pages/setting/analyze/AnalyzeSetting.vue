<template>
  <div class="setting-site-container">
    <div class="header">
      <t-row justify="space-between">
        <div class="left-operation-container">
          <!-- 删除无二次确认，谨慎操作！ -->
        </div>
        <div class="right-operation-container">
          <div class="component-op">
            <div class="item" @click="exportEvent">
              <t-icon size="1.5em" name="arrow-up" />
              <span>导出</span>
            </div>
            <div class="item" @click="removeAllEvent">
              <t-icon size="1.5em" name="remove" />
              <span>删除</span>
            </div>
            <div class="item" @click="formDialogVisibleAddAnalyze = true">
              <t-icon size="1.5em" name="add" />
              <span>添加</span>
            </div>
          </div>
        </div>
      </t-row>
    </div>
    <t-table
      row-key="id"
      :data="emptyData ? [] : data"
      :columns="COLUMNS"
      :hover="true"
      stripe
      :loading="dataLoading"
      :header-affixed-top="{ offsetTop: 0, container: `.setting-site-container` }"
      @select-change="rehandleSelectChange"
    >
      <template #isActive="{ row }">
        <t-switch v-model="row.isActive" @change="propChangeEvent(row)">
          <template #label="tip">{{ tip.value ? '开' : '关' }}</template>
        </t-switch>
      </template>
      <template #ext="{ row }">
        <span v-for="item in row.ext" :key="item.id">{{ item }},</span>
      </template>
      <template #op="slotProps">
        <a class="t-button-link" @click="defaultEvent(slotProps)">默认</a>
        <a class="t-button-link" @click="editEvent(slotProps)">编辑</a>
        <a class="t-button-link" @click="removeEvent(slotProps)">删除</a>
      </template>
    </t-table>
    <dialog-form-add-analyze
      v-model:visible="formDialogVisibleAddAnalyze"
      :data="data"
      @refresh-table-data="getAnalyze"
    />
    <dialog-form-edit-analyze v-model:visible="formDialogVisibleEditAnalyze" :data="formData" />
  </div>
</template>
<script setup>
import { ref, onMounted } from 'vue';
import { MessagePlugin } from 'tdesign-vue-next';
import { saveAs } from 'file-saver';
import { analyze, setting } from '@/lib/dexie';
import DialogFormAddAnalyze from './components/DialogFormAddAnalyze.vue';
import DialogFormEditAnalyze from './components/DialogFormEditAnalyze.vue';
import { COLUMNS } from './constants';

// Define item form data & dialog status
const formDialogVisibleAddAnalyze = ref(false);
const formDialogVisibleEditAnalyze = ref(false);
const formData = ref();

// Define table
const emptyData = ref(false);
const dataLoading = ref(false);
const data = ref([]);
const selectedRowKeys = ref([]);
const rehandleSelectChange = (val) => {
  selectedRowKeys.value = val;
};

onMounted(() => {
  getAnalyze();
});

// 获取列表
const getAnalyze = () => {
  dataLoading.value = true;
  try {
    analyze.all().then((res) => {
      console.log(res);
      if (!res) emptyData.value = true;
      data.value = res;
    });
  } catch (e) {
    console.log(e);
  } finally {
    dataLoading.value = false;
  }
};

// 是否启用
const propChangeEvent = (row) => {
  analyze.update(row.id, { isActive: row.isActive });
};

// 编辑
const editEvent = (row) => {
  formData.value = data.value[row.rowIndex];
  formDialogVisibleEditAnalyze.value = true;
};

// 删除
const removeEvent = (row) => {
  analyze
    .remove(row.row.id)
    .then(() => {
      getAnalyze();
      MessagePlugin.success('删除成功');
    })
    .catch((err) => {
      MessagePlugin.error(`删除源失败, 错误信息:${err}`);
    });
};

// 批量删除
const removeAllEvent = () => {
  if (selectedRowKeys.value.length === 0) {
    MessagePlugin.warning('请先选择数据');
    return;
  }
  selectedRowKeys.value.forEach((element) => {
    console.log(element);
    analyze.remove(element).catch((err) => {
      MessagePlugin.error(`批量删除源失败, 错误信息:${err}`);
    });
  });
  getAnalyze();
  MessagePlugin.success('批量删除成功');
};

// 导出接口
const exportEvent = () => {
  const arr = [...data.value];
  const str = JSON.stringify(arr, null, 2);
  const blob = new Blob([str], { type: 'text/plain;charset=utf-8' });
  saveAs(blob, `analyzes.json`);
  MessagePlugin.success('导出成功');
};

// 设置默认接口
const defaultEvent = async (row) => {
  setting.update({
    defaultAnalyze: row.row.url,
  });
  MessagePlugin.success('设置成功');
};
</script>

<style lang="less" scoped>
@import '@/style/variables';
.setting-site-container {
  .header {
    margin-bottom: 20px;
  }
  .t-button-link {
    margin-right: var(--td-comp-margin-xxl);
  }
}
.component-op {
  display: flex;
  padding: 4px;
  height: 40px;
  background: #f0f0f0;
  backdrop-filter: blur(10px);
  border-radius: 6px;
  color: #161616;
  align-items: center;
  box-shadow: 10px;
  margin-right: 20px;
  .item {
    border-radius: 5px;
    transition: all 0.2s ease 0s;
    display: flex;
    align-items: center;
    padding: 5px 8px;
    line-height: 22px;
    cursor: pointer;
    text-decoration: none;
  }
  .item:hover {
    background: #dcdcdc;
  }
}
:root[theme-mode='dark'] {
  .component-op {
    background: #484848;
    color: #eee;
    .item:hover {
      background: #5e5e5e;
    }
  }
}
</style>

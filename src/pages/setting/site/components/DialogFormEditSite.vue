<template>
  <t-dialog v-model:visible="formVisible" header="编辑源站" :width="680" :footer="false">
    <template #body>
      <!-- 表单内容 -->
      <t-form ref="form" colon :data="formData" :rules="rules" :label-width="100" @submit="onSubmit">
        <div class="add-once">
          <t-form-item label="源站名" name="name">
            <t-input v-model="formData.name" placeholder="请输入内容" />
          </t-form-item>
          <t-form-item label="API接口" name="api">
            <t-input v-model="formData.api" placeholder="请输入内容" />
          </t-form-item>
          <t-form-item label="下载接口" name="download">
            <t-input v-model="formData.download" placeholder="请输入内容" />
          </t-form-item>
          <t-form-item label="解析接口" name="jiexiUrl">
            <t-input v-model="formData.jiexiUrl" placeholder="请输入内容" />
          </t-form-item>
          <!-- <t-form-item label="分组" name="group">
            <t-select v-model="formData.type" clearable :style="{ width: '514px' }">
              <t-option v-for="(item, index) in SELECT_OPTIONS" :key="index" :value="item.value" :label="item.label">
                {{ item.label }}
              </t-option>
            </t-select>
          </t-form-item> -->
          <t-form-item label="源站标识" name="key">
            <t-input v-model="formData.key" placeholder="请输入内容" />
          </t-form-item>
        </div>
        <div class="optios">
          <t-form-item style="float: right">
            <t-space>
              <t-button variant="outline" @click="onClickCloseBtn">取消</t-button>
              <t-button theme="primary" type="submit">确定</t-button>
            </t-space>
          </t-form-item>
        </div>
      </t-form>
    </template>
  </t-dialog>
</template>

<script setup>
import { ref, watch, onMounted } from 'vue';
import { MessagePlugin } from 'tdesign-vue-next';
import getUuid from 'uuid-by-string';
import { sites } from '@/lib/dexie';

const props = defineProps({
  visible: {
    type: Boolean,
    default: false,
  },
  data: {
    type: Object,
    default: () => {
      return {};
    },
  },
});
const formVisible = ref(false);
const formData = ref(props.data);
const onSubmit = ({ result, firstError }) => {
  console.log(result, firstError);
  if (!firstError) {
    if (!formData.value.key) formData.value.key = getUuid(formData.value.name, 5);
    sites
      .update(formData.value.id, formData.value)
      .then(() => {
        MessagePlugin.success('修改成功！');
      })
      .catch((err) => {
        MessagePlugin.error(`修改失败, 错误信息:${err}`);
      });
    formVisible.value = false;
  } else {
    console.log('Errors: ', result);
    MessagePlugin.warning(firstError);
  }
};
const onClickCloseBtn = () => {
  formVisible.value = false;
};
const emit = defineEmits(['update:visible']);
watch(
  () => formVisible.value,
  (val) => {
    emit('update:visible', val);
  },
);
watch(
  () => props.visible,
  (val) => {
    formVisible.value = val;
  },
);
watch(
  () => props.data,
  (val) => {
    formData.value = val;
    console.log(val);
  },
);
const rules = {
  name: [{ required: true, message: '请输入源站名', type: 'error' }],
  api: [{ required: true, message: '请输入Api接口url', type: 'error' }],
};
onMounted(() => {
  getSitesGroup();
});
const getSitesGroup = () => {
  // const arr = [];
  // for (const i of sites) {
  //   if (arr.indexOf(i.group) < 0) {
  //     arr.push(i.group);
  //   }
  // }
  // console.log(arr);
  // siteGroup = arr;
};
</script>
<style lang="less" scoped>
@import '@/style/variables';
:deep(.t-form:not(.t-form-inline) .t-form__item:last-of-type) {
  margin-bottom: var(--td-comp-margin-xxl);
}
</style>

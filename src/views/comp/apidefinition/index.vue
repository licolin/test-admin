<template>
  <n-layout has-sider style="height: 100vh">
    <!-- 左侧 Tree + 搜索 -->
    <n-layout-sider bordered width="260" content-style="padding: 16px;">
      <n-input
        v-model:value="searchValue"
        placeholder="搜索"
        clearable
        style="margin-bottom: 12px"
      />
      <n-tree
        block-line
        :data="treeData"
        :default-expand-all="true"
        :selectable="true"
        :selected-keys="selectedKeys"
        @update:selected-keys="onSelectTree"
      />
    </n-layout-sider>

    <!-- 右侧滚动区域 -->
    <div class="scroll-wrapper">
      <div class="scroll-content">
        <!-- 顶部按钮 -->
        <div class="top-bar">
          <n-button type="primary">新建接口定义</n-button>
          <n-button type="info" @click="showCreateDirModal = true">创建接口目录</n-button>
          <n-button type="success">保存</n-button>
          <n-button>取消</n-button>
        </div>

        <!-- 表单内容 -->
        <div style="padding: 24px">
          <n-form label-width="100">
            <div style="display: flex; gap: 16px">
              <n-form-item label="接口名称">
                <n-input v-model:value="form.name" placeholder="请输入接口名称" />
              </n-form-item>
              <n-form-item label="请求方法" style="flex: 1">
                <n-select v-model:value="form.method" :options="methodOptions" />
              </n-form-item>
              <n-form-item label="请求路径" style="flex: 2">
                <n-input v-model:value="form.path" />
              </n-form-item>
              <n-form-item label="接口分组" style="flex: 1">
                <n-input v-model:value="form.group" />
              </n-form-item>
            </div>
            <!-- 请求参数 -->
            <n-divider>请求参数</n-divider>
            <n-data-table :columns="reqColumns" :data="reqParams" />
            <n-button tertiary type="primary" @click="addReqParam" style="margin-top: 8px"
              >+ 添加参数</n-button
            >
          </n-form>
        </div>
      </div>
    </div>

    <!-- 创建接口目录 Modal -->
    <n-modal v-model:show="showCreateDirModal">
      <n-card
        style="width: 400px"
        title="创建接口目录"
        :bordered="false"
        size="medium"
        role="dialog"
        aria-modal="true"
      >
        <n-form label-width="80" :model="createDirForm" style="margin-top: 12px">
          <n-form-item label="目录名称" path="name" required>
            <n-input v-model:value="createDirForm.name" placeholder="请输入目录名称" />
          </n-form-item>
          <n-form-item label="目录描述" path="desc">
            <n-input v-model:value="createDirForm.desc" placeholder="请输入目录描述" />
          </n-form-item>
        </n-form>
        <template #footer>
          <n-space justify="end">
            <n-button @click="showCreateDirModal = false">取消</n-button>
            <n-button type="primary" @click="handleCreateDir">确定</n-button>
          </n-space>
        </template>
      </n-card>
    </n-modal>
  </n-layout>
</template>

<script setup>
  import { ref, h } from 'vue';
  import { NInput, NSelect, NButton } from 'naive-ui';

  const searchValue = ref('');
  const treeData = ref([
    {
      label: '用户管理',
      key: 'user',
      children: [
        { label: 'GET /api/user/list', key: 'api-user-list' },
        { label: 'GET /api/user/{id}', key: 'api-user-id' },
      ],
    },
    {
      label: '订单管理',
      key: 'order',
      children: [{ label: 'GET /api/order/list', key: 'api-order-list' }],
    },
  ]);
  const selectedKeys = ref(['api-user-list']);
  const onSelectTree = (keys) => {
    selectedKeys.value = keys;
  };

  const form = ref({
    name: '',
    method: 'GET',
    path: '',
    group: '',
  });

  const methodOptions = [
    { label: 'GET', value: 'GET' },
    { label: 'POST', value: 'POST' },
    { label: 'PUT', value: 'PUT' },
    { label: 'AGW', value: 'AGW' },
    { label: 'KCXP', value: 'KCXP' },
  ];

  // 可编辑的请求参数表头
  const reqParams = ref([{ name: 'id', type: 'string', required: true, default: '', desc: '' }]);
  const reqColumns = [
    {
      title: '字段名',
      key: 'name',
      render(row, index) {
        return h(NInput, {
          value: row.name,
          onUpdateValue: (v) => (reqParams.value[index].name = v),
          placeholder: '字段名',
        });
      },
    },
    {
      title: '类型',
      key: 'type',
      render(row, index) {
        return h(NInput, {
          value: row.type,
          onUpdateValue: (v) => (reqParams.value[index].type = v),
          placeholder: '类型',
        });
      },
    },
    {
      title: '必填',
      key: 'required',
      render(row, index) {
        return h(NSelect, {
          value: row.required,
          options: [
            { label: '是', value: true },
            { label: '否', value: false },
          ],
          onUpdateValue: (v) => (reqParams.value[index].required = v),
        });
      },
    },
    {
      title: '默认值',
      key: 'default',
      render(row, index) {
        return h(NInput, {
          value: row.default,
          onUpdateValue: (v) => (reqParams.value[index].default = v),
          placeholder: '默认值',
        });
      },
    },
    {
      title: '说明',
      key: 'desc',
      render(row, index) {
        return h(NInput, {
          value: row.desc,
          onUpdateValue: (v) => (reqParams.value[index].desc = v),
          placeholder: '说明',
        });
      },
    },
    {
      title: '操作',
      key: 'actions',
      render(row, index) {
        return h(
          NButton,
          {
            type: 'error',
            size: 'small',
            onClick: () => reqParams.value.splice(index, 1),
          },
          { default: () => '删除' }
        );
      },
    },
  ];
  // 可编辑的返回参数表头
  const resParams = ref([
    { name: 'code', type: 'string', desc: '' },
    { name: 'data', type: 'object', desc: '' },
  ]);
  const resColumns = [
    {
      title: '字段名',
      key: 'name',
      render(row, index) {
        return h(NInput, {
          value: row.name,
          onUpdateValue: (v) => (resParams.value[index].name = v),
          placeholder: '字段名',
        });
      },
    },
    {
      title: '类型',
      key: 'type',
      render(row, index) {
        return h(NInput, {
          value: row.type,
          onUpdateValue: (v) => (resParams.value[index].type = v),
          placeholder: '类型',
        });
      },
    },
    {
      title: '说明',
      key: 'desc',
      render(row, index) {
        return h(NInput, {
          value: row.desc,
          onUpdateValue: (v) => (resParams.value[index].desc = v),
          placeholder: '说明',
        });
      },
    },
    {
      title: '操作',
      key: 'actions',
      render(row, index) {
        return h(
          NButton,
          {
            type: 'error',
            size: 'small',
            onClick: () => resParams.value.splice(index, 1),
          },
          { default: () => '删除' }
        );
      },
    },
  ];

  // 新增参数
  const addReqParam = () => {
    reqParams.value.push({ name: '', type: '', required: false, default: '', desc: '' });
  };
  const addResParam = () => {
    resParams.value.push({ name: '', type: '', desc: '' });
  };

  const mockData = ref(`{
  "code": 0,
  "data": []
}`);

  // 创建接口目录 modal
  const showCreateDirModal = ref(false);
  const createDirForm = ref({
    name: '',
    desc: '',
  });
  const handleCreateDir = () => {
    if (!createDirForm.value.name) {
      window.$message?.warning?.('请输入目录名称');
      return;
    }
    // 添加目录到 treeData
    treeData.value.push({
      label: createDirForm.value.name,
      key: `dir-${Date.now()}`,
      children: [],
    });
    showCreateDirModal.value = false;
    createDirForm.value.name = '';
    createDirForm.value.desc = '';
  };
</script>

<style scoped>
  .scroll-wrapper {
    height: 100vh;
    width: calc(100% - 260px);
    overflow: hidden;
    /* 避免父级出现滚动条 */
    position: relative;
  }

  .scroll-content {
    height: 100vh;
    overflow-y: auto;
    padding-right: 15px;
    box-sizing: content-box;
    background: #fff;
  }

  /* 顶部按钮栏样式，防止跟随滚动可加 position: sticky */
  .top-bar {
    display: flex;
    justify-content: flex-end;
    gap: 8px;
    padding: 16px;
    border-bottom: 1px solid #eee;
    flex-shrink: 0;
    background: #fff;
  }

  /* 隐藏滚动条 - webkit */
  .scroll-content::-webkit-scrollbar {
    width: 0;
    height: 0;
    display: none;
  }

  /* 隐藏滚动条 - Firefox */
  .scroll-content {
    scrollbar-width: none;
    -ms-overflow-style: none;
  }
</style>

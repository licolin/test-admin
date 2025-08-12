<template>
  <div>
    <div class="two-column mt-4">
      <!-- 左侧：Tree View（真实树形展示） -->
      <div class="left-panel">
        <n-card title="接口定义" size="small" :bordered="false">
          <n-tree
            :data="treeData"
            block-line
            :default-expand-all="true"
            selectable
            :selected-keys="selectedKeys"
            @update:selected-keys="handleTreeSelect"
          >
            <template #label="{ option }">
              <div class="tree-node-label">
                <span v-if="option.children" class="node-title">{{ option.label }}</span>
                <span
                  v-else
                  class="node-title draggable-node"
                  draggable="true"
                  @dragstart="onDragStart($event, option)"
                >
                  {{ option.label }}
                </span>
              </div>
            </template>
          </n-tree>
        </n-card>
      </div>

      <!-- 右侧：Tabs（场景、前置、后置） -->
      <div class="right-panel" @dragover.prevent @drop="onDrop">
        <n-card size="small" :bordered="false">
          <n-tabs v-model:value="activeTab" type="line" animated>
            <n-tab-pane name="scene" tab="场景">
              <Draggable class="draggable-ul" animation="200" :list="sceneList" itemKey="id">
                <template #item="{ element, index }">
                  <div class="cursor-move draggable-li">
                    <div class="li-left">
                      <n-tag type="info">任务</n-tag>
                      <span class="ml-2">{{ element.name }}</span>
                    </div>
                    <div class="li-right">
                      <n-button
                        size="tiny"
                        :style="{
                          backgroundColor: '#409EFF',
                          border: 'none',
                          boxShadow: 'none',
                          color: '#fff',
                        }"
                        @click="removeTask('scene', index)"
                      >
                        删除
                      </n-button>
                    </div>
                  </div>
                </template>
                <template #footer>
                  <div v-if="sceneList.length === 0" class="empty-tip">
                    点击左侧接口,接口会被添加到场景列表
                  </div>
                </template>
              </Draggable>
            </n-tab-pane>
            <n-tab-pane name="pre" tab="前置">
              <div style="margin-bottom: 10px; display: flex; gap: 12px">
                <n-select
                  v-model:value="preLanguage"
                  :options="languageOptions"
                  style="width: 120px"
                />
                <n-select v-model:value="preTheme" :options="themeOptions" style="width: 120px" />
              </div>
              <Codemirror
                v-model="preCode"
                :extensions="currentExtensions"
                :style="{ height: '400px', border: '1px solid #eee' }"
                placeholder="在这里编辑前置代码..."
                :autofocus="true"
                :indent-with-tab="true"
                :tab-size="4"
              />
            </n-tab-pane>
            <n-tab-pane name="post" tab="后置">
              <Draggable class="draggable-ul" animation="200" :list="postList" itemKey="id">
                <template #item="{ element, index }">
                  <div class="cursor-move draggable-li">
                    <div class="li-left">
                      <n-tag type="warning">后置</n-tag>
                      <span class="ml-2">{{ element.name }}</span>
                    </div>
                    <div class="li-right">
                      <n-button
                        size="tiny"
                        :style="{
                          backgroundColor: '#E6A23C',
                          border: 'none',
                          boxShadow: 'none',
                          color: '#fff',
                        }"
                        @click="removeTask('post', index)"
                      >
                        删除
                      </n-button>
                    </div>
                  </div>
                </template>
                <template #footer>
                  <div v-if="postList.length === 0" class="empty-tip"> 可拖拽接口到后置 </div>
                </template>
              </Draggable>
            </n-tab-pane>
          </n-tabs>
        </n-card>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
  import { reactive, ref, computed } from 'vue';
  import Draggable from 'vuedraggable';
  import { Codemirror } from 'vue-codemirror';
  import { sql } from '@codemirror/lang-sql';
  import { python } from '@codemirror/lang-python';
  import { oneDark } from '@codemirror/theme-one-dark'; // 可选主题

  const treeData = reactive([
    {
      label: '前端开发',
      key: 'frontend',
      children: [
        { label: '预约表单页面', key: 1 },
        { label: '促销活动页面', key: 2 },
        { label: '商品列表页面', key: 3 },
      ],
    },
    {
      label: '后端开发',
      key: 'backend',
      children: [
        { label: '订单删除功能', key: 4 },
        { label: '用户头像裁剪', key: 5 },
      ],
    },
    {
      label: 'UI优化',
      key: 'ui',
      children: [
        { label: '商品图片放大镜', key: 6 },
        { label: '评价功能', key: 7 },
      ],
    },
  ]);

  const activeTab = ref<'scene' | 'pre' | 'post'>('scene');

  // 场景和后置列表
  const sceneList = reactive([] as Array<{ id: string | number; name: string }>);
  const postList = reactive([] as Array<{ id: string | number; name: string }>);

  // 前置代码编辑
  const preCode = ref('');
  const preLanguage = ref('sql');
  const preTheme = ref('light');
  const languageOptions = [
    { label: 'SQL', value: 'sql' },
    { label: 'Python', value: 'python' },
  ];
  const themeOptions = [
    { label: '默认(light)', value: 'light' },
    { label: 'One Dark', value: 'oneDark' },
  ];

  const currentExtensions = computed(() => {
    let lang = preLanguage.value === 'sql' ? sql() : python();
    let theme = preTheme.value === 'oneDark' ? oneDark : [];
    // 返回一个包含语言和主题的数组
    return [lang, ...(Array.isArray(theme) ? theme : [theme])];
  });

  // const extensionsSql = [sql(), oneDark];
  // const extensionsPython = [python(), oneDark];
  //
  // const currentExtensions = computed(() => {
  //   return preLanguage.value === 'sql' ? extensionsSql : extensionsPython;
  // });

  const selectedKeys = ref<(string | number)[]>([]);

  /* 点击树节点添加（仅添加到“场景”tab） */
  function handleTreeSelect(keys: (string | number)[]) {
    selectedKeys.value = keys;
    const key = keys[0];
    if (!key) return;
    const node = findTaskInTree(treeData, key);
    if (node && !node.children && !sceneList.find((t) => t.id === node.key)) {
      sceneList.push({ id: node.key, name: node.label });
    }
  }

  /* 递归查找节点 */
  function findTaskInTree(tree: any[], key: string | number) {
    for (const node of tree) {
      if (node.key === key) return node;
      if (node.children) {
        const r = findTaskInTree(node.children, key);
        if (r) return r;
      }
    }
    return null;
  }

  /* 拖拽开始：把节点信息序列化到 dataTransfer 中 */
  function onDragStart(e: DragEvent, option: any) {
    const payload = { id: option.key, name: option.label };
    try {
      e.dataTransfer?.setData('application/json', JSON.stringify(payload));
      e.dataTransfer?.setData('text/plain', String(option.key));
    } catch (err) {}
    if (e.dataTransfer) e.dataTransfer.effectAllowed = 'copy';
  }

  /* 拖拽释放，根据当前tab添加到对应列表或代码 */
  function onDrop(e: DragEvent) {
    e.preventDefault();
    let raw = '';
    try {
      raw =
        e.dataTransfer?.getData('application/json') || e.dataTransfer?.getData('text/plain') || '';
    } catch (err) {
      raw = '';
    }
    if (!raw) return;

    let obj: any = {};
    try {
      obj = JSON.parse(raw);
      if (!obj.name) {
        const node = findTaskInTree(treeData, obj.id || raw);
        if (node) obj.name = node.label;
      }
    } catch (err) {
      const id = raw;
      const node = findTaskInTree(treeData, id);
      if (node) obj = { id: node.key, name: node.label };
    }
    if (!obj.id || !obj.name) return;

    // 根据当前 tab 添加
    if (activeTab.value === 'scene' && !sceneList.find((t) => t.id === obj.id)) {
      sceneList.push({ id: obj.id, name: obj.name });
    } else if (activeTab.value === 'pre') {
      // 添加到前置代码作为注释
      const comment = preLanguage.value === 'sql' ? `-- ${obj.name}\n` : `# ${obj.name}\n`;
      preCode.value += `\n${comment}`;
    } else if (activeTab.value === 'post' && !postList.find((t) => t.id === obj.id)) {
      postList.push({ id: obj.id, name: obj.name });
    }
  }

  /* 删除 */
  function removeTask(type: 'scene' | 'pre' | 'post', index: number) {
    if (type === 'scene') sceneList.splice(index, 1);
    if (type === 'post') postList.splice(index, 1);
    // pre 没有列表，无需删除
  }
</script>

<style lang="less" scoped>
  .two-column {
    display: flex;
    gap: 12px;
    align-items: flex-start;
  }
  .left-panel {
    flex: 1;
    min-width: 220px;
  }
  .right-panel {
    flex: 2;
  }

  /* 小屏幕堆叠 */
  @media (max-width: 640px) {
    .two-column {
      flex-direction: column;
    }
  }
  .draggable-ul {
    width: 100%;
    overflow: hidden;
    margin-top: -12px;
  }
  .draggable-li {
    width: 100%;
    padding: 12px 10px;
    color: #333;
    border-bottom: 1px solid #efeff5;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  .tree-node-label .draggable-node {
    cursor: grab;
    user-select: none;
    display: inline-block;
    padding: 2px 4px;
  }
  .tree-node-label .draggable-node:active {
    cursor: grabbing;
  }
  .empty-tip {
    padding: 12px;
    color: #888;
    text-align: center;
  }
</style>

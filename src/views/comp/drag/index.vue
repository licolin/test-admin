<template>
  <div>
<!--    <n-alert title="接口定义到测试场景" type="info" class="mt-4">-->
<!--      左侧为接口定义（树形）,支持把接口定义拖拽到右侧,构成测试场景。-->
<!--    </n-alert>-->

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
            <!-- 为叶子节点渲染可拖拽标签 -->
            <template #label="{ option }">
              <div class="tree-node-label">
                <!-- 如果有 children 则不是叶子，仅显示文字 -->
                <span v-if="option.children" class="node-title">{{ option.label }}</span>

                <!-- 叶子节点支持拖拽 -->
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

      <!-- 右侧：可拖拽的场景列表（并接收来自左侧的原生 drop） -->
      <div class="right-panel" @dragover.prevent @drop="onDrop">
        <n-card size="small" :bordered="false">
          <template #header>
            <div style="display: flex; justify-content: space-between; align-items: center">
              <span>场景</span>
              <n-button size="medium" type="primary" @click="editScene"> 场景编辑 </n-button>
            </div>
          </template>

          <Draggable class="draggable-ul" animation="200" :list="taskList" itemKey="id">
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
                    @click="removeTask(index)"
                  >
                    删除
                  </n-button>
                </div>
              </div>
            </template>

            <template #footer>
              <div v-if="taskList.length === 0" class="empty-tip">
                点击左侧接口,接口会被添加到场景列表
              </div>
            </template>
          </Draggable>
        </n-card>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
  import { reactive, ref } from 'vue';
  import Draggable from 'vuedraggable';

  /**
   * 树形数据（保持原来结构）
   * 每个叶子节点用 key 表示唯一 id，label 表示名称
   */
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

  // 右侧任务（场景）列表（可排序、可删除）
  const taskList = reactive([] as Array<{ id: string | number; name: string }>);

  // 选中的树节点（保留点击添加的行为）
  const selectedKeys = ref<(string | number)[]>([]);

  /* ==========  点击树节点添加（保留） ========== */
  function handleTreeSelect(keys: (string | number)[]) {
    selectedKeys.value = keys;
    const key = keys[0];
    if (!key) return;
    const node = findTaskInTree(treeData, key);
    // 仅当为叶子节点（没有 children）时才添加
    if (node && !node.children && !taskList.find((t) => t.id === node.key)) {
      taskList.push({ id: node.key, name: node.label });
    }
  }

  /* 递归在树中查找节点 */
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

  /* ========== 从左侧树原生拖拽到右侧 ========== */
  /* dragstart：把节点信息序列化到 dataTransfer 中 */
  function onDragStart(e: DragEvent, option: any) {
    // 只允许拖拽叶子节点（template 中已仅对叶子节点绑定）
    const payload = { id: option.key, name: option.label };
    try {
      e.dataTransfer?.setData('application/json', JSON.stringify(payload));
      // 便于在某些浏览器回退兼容
      e.dataTransfer?.setData('text/plain', String(option.key));
    } catch (err) {
      // 某些受限环境可能会抛错，忽略即可
    }
    if (e.dataTransfer) e.dataTransfer.effectAllowed = 'copy';
  }

  /* drop：从 dataTransfer 中读取数据并添加到 taskList（避免重复） */
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

    try {
      const obj = JSON.parse(raw);
      // 如果 data 为单个 key（text/plain），尝试从 tree 中查找 label
      if (!obj.name) {
        const node = findTaskInTree(treeData, obj.id || raw);
        if (node) obj.name = node.label;
      }
      // 防重复
      if (!taskList.find((t) => t.id === obj.id)) {
        taskList.push({ id: obj.id, name: obj.name });
      }
    } catch (err) {
      // raw 可能只是 id 字符串
      const id = raw;
      const node = findTaskInTree(treeData, id);
      if (node && !taskList.find((t) => t.id === node.key)) {
        taskList.push({ id: node.key, name: node.label });
      }
    }
  }

  /* ========== 右侧删除 ========== */
  function removeTask(index: number) {
    taskList.splice(index, 1);
  }
</script>

<style lang="less" scoped>
  .two-column {
    display: flex;
    gap: 12px;
    align-items: flex-start;
  }

  /* 两侧宽度比例 2:3 */
  .left-panel {
    flex: 1;
    min-width: 220px;
  }
  .right-panel {
    flex: 2;
  }

  /* 在小屏幕上堆叠 */
  @media (max-width: 640px) {
    .two-column {
      flex-direction: column;
    }
  }

  /* 拖拽列表样式 */
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

  /* 树节点样式：叶子节点呈现为可拖拽的句柄 */
  .tree-node-label .draggable-node {
    cursor: grab;
    user-select: none;
    display: inline-block;
    padding: 2px 4px;
  }

  /* 当被拖拽时的样式提示（仅视觉） */
  .tree-node-label .draggable-node:active {
    cursor: grabbing;
  }

  /* 空列表提示 */
  .empty-tip {
    padding: 12px;
    color: #888;
    text-align: center;
  }
</style>

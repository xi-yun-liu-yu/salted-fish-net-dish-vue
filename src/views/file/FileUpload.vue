<script setup>
/* 文件传输列表组件 - 包含上传中和已完成的文件列表 */

import {
  Document,
  UploadFilled,
  VideoPause,
  Promotion,
  CircleClose, VideoPlay,CircleCheck
} from '@element-plus/icons-vue'

import {useUploadListStore} from "@/stores/uploadList.js";
import {usePathStore} from "@/stores/path.js";
import {ElMessage} from "element-plus";
import router from "@/router/index.js";
import {useActiveStore} from "@/stores/active.js";
import {hasFolder} from "@/api/file.js";
import {useTokenStore} from "@/stores/token.js";
const tokenStore = useTokenStore();
const uploadListStore = useUploadListStore()

/* 删除已完成的上传记录 */
const deleteFinish = async (uuid) => {
  uploadListStore.deleteFinishList(uuid)
}

/* 导航到文件所在目录 */
const goFile = async (path) => {
  const pathStore = usePathStore();
  let temp = []
  path.forEach((element) => {
    temp.push(element);
  });
  // 验证文件夹是否存在
  let result = await hasFolder(tokenStore.token,temp[temp.length - 1]);
  if (result.data) {
    pathStore.setPath(temp)
    const name = pathStore.path.pop()
    ElMessage.success({
      message: `该文件储存为当前目录下的：` + name,
      duration: 10000,
    })
    const activeStore = useActiveStore();
    activeStore.setActive('/main/file/FileManage');
    await router.push('/main/file/FileManage');
  }else {
    ElMessage.warning({
      message: `该文件已不存在`,
      duration: 10000,
    })
  }
}

</script>

<template>
  <!-- 传输管理主容器 -->
  <div class="transfer-container">
    <!-- 标签页导航 -->
    <el-tabs class="custom-tabs" v-model="activeTab" @tab-click="handleTabClick">
      <!-- 上传中列表标签页 -->
      <el-tab-pane name="upload">
        <template #label>
          <span class="custom-tab-label">
            <el-icon><UploadFilled /></el-icon>
            上传列表
          </span>
        </template>
        <!-- 上传中文件表格 -->
        <div class="transfer-section">
          <el-table :data="uploadListStore.uploadList" style="width: 100%" height="70vh">
            <!-- 文件名列 -->
            <el-table-column prop="fileName" label="文件名" width="500" >
              <template #default="{ row }">
                <el-icon class="file-icon">
                  <Document />
                </el-icon>
                {{ row.fileName }}
              </template>
            </el-table-column>
            <!-- 进度列 -->
            <el-table-column label="进度" width="300">
              <template #default="{ row }">
                <el-progress
                    :percentage="row.progress"
                    :color="customColors"
                    :stroke-width="16"
                    striped
                />
              </template>
            </el-table-column>

            <el-table-column prop="speed" label="速度" width="200" />
            <!-- 状态列 -->
            <el-table-column label="状态" width="150" >
              <template #default="{ row }">
                <el-tag :type="statusType[row.status]">
                  {{ row.status }}
                </el-tag>
              </template>
            </el-table-column>
            <!-- 操作列 -->
            <el-table-column label="操作" width="150" >
              <template #default="{ row }">
                <el-button
                    v-if="row.status === '传输中'"
                    :icon="VideoPause"
                    type="warning"
                    circle
                    size="small"
                />
                <el-button
                    v-if="row.status === '已暂停'"
                    :icon="VideoPlay"
                    type="warning"
                    circle
                    size="small"
                />
                <el-button
                    :icon="CircleClose"
                    type="danger"
                    circle
                    size="small"
                />
              </template>
            </el-table-column>
            <!-- 空状态提示 -->
            <template #empty>
              <el-empty
                  description="暂无上传任务"
                  image-size="120"
                  class="custom-empty"
              >
                <template #image>
                  <el-icon :size="60" color="#C0C4CC"><Document /></el-icon>
                </template>
              </el-empty>
            </template>
          </el-table>
        </div>

      </el-tab-pane>
      <!-- 已完成列表标签页 -->
      <el-tab-pane name="download">
        <template #label>
          <span class="custom-tab-label">
            <el-icon><Promotion/></el-icon>
            上传完成
          </span>
        </template>

        <!-- 已完成文件表格 -->
        <div class="transfer-section">
          <el-table :data="uploadListStore.finishList" style="width: 100%" height="70vh">
            <el-table-column prop="fileName" label="文件名" width="500" >
              <template #default="{ row }">
                <el-icon class="file-icon">
                  <Document />
                </el-icon>
                {{ row.fileName }}
              </template>
            </el-table-column>
            <el-table-column prop="finishData" label="下载日期" width="300"/>
            <el-table-column prop="fileSize" label="文件大小" width="200" />
            <!-- 查看文件列 -->
            <el-table-column label="查看文件" width="150" >
              <template #default="{ row }">
                <el-button size="small" @click = goFile(row.path)>
                  前往
                </el-button>
              </template>
            </el-table-column>
            <!-- 删除操作列 -->
            <el-table-column label="操作" width="150" >
              <template #default="{ row }">
                <el-button
                    :icon="CircleClose"
                    type="danger"
                    circle
                    size="small"
                    @click = deleteFinish(row.uuid)
                />
              </template>
            </el-table-column>
            <!-- 空状态提示 -->
            <template #empty>
              <el-empty
                  description="暂无已完成上传"
                  image-size="120"
                  class="custom-empty"
              >
                <template #image>
                  <el-icon :size="60" color="#67C23A"><CircleCheck /></el-icon>
                </template>
              </el-empty>
            </template>
          </el-table>
        </div>
      </el-tab-pane>


    </el-tabs>
  </div>
</template>

<script>
export default {
  data() {
    return {
      activeTab: 'upload',// 当前激活的标签页
      customColors: [// 进度条渐变色配置
        { color: '#409EFF', percentage: 100 },
      ],
      statusType: {// 状态标签类型映射
        '传输中': 'primary',
        '已完成': 'success',
        '已暂停': 'warning',
        '失败': 'danger'
      }
    }
  }
}
</script>

<style lang="scss" scoped>
/* 传输管理容器样式 */
.transfer-container {
  padding: 20px;
  height: 100%;
  background: #f5f7fa;
  /* 自定义标签页样式 */
  .custom-tabs {
    background: white;
    border-radius: 15px;
    box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
    overflow: hidden;
    /* 标签头样式 */
    :deep(.el-tabs__header) {
      margin: 0;
      background: linear-gradient(145deg, #f8f9fe, #eef1f8);

      .el-tabs__nav-wrap {
        padding: 0 20px;
        /* 导航栏装饰线隐藏 */
        &::after {
          height: 0;
        }
      }
      /* 激活指示条样式 */
      .el-tabs__active-bar {
        height: 3px;
        background: #409EFF;
        border-radius: 2px;
      }
      /* 单个标签项样式 */
      .el-tabs__item {
        height: 60px;
        padding: 0 35px;
        font-size: 16px;
        color: #606266;
        transition: all 0.3s ease;
        /* 悬停效果 */
        &:hover {
          color: #409EFF;
          background: rgba(64, 158, 255, 0.05);
        }
        /* 激活状态样式 */
        &.is-active {
          color: #409EFF;
          font-weight: 500;
        }

        .custom-tab-label {
          display: flex;
          align-items: center;
          gap: 8px;

          .el-icon {
            font-size: 1.2em;
            margin-right: 6px;
          }
        }
      }
    }
    /* 传输内容区域 */
    .transfer-section {
      padding: 20px;
      background: transparent;
      box-shadow: none;
      /* 表格全局样式 */
      :deep(.el-table) {
        border-radius: 10px;
        overflow: hidden;
        /* 表头样式 */
        th {
          font-weight: 600;
          background: #f8f9fc !important;
        }
        /* 单元格悬停效果 */
        td {
          transition: background 0.2s;

          &:hover {
            background: #fafbff !important;
          }
        }
      }
    }
  }

}
</style>
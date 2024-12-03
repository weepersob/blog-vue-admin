<template>
  <div class="container">
    <!-- 标题和操作按钮 -->
    <div class="title-bar">
      <el-input v-model="form.title" placeholder="请输入文章标题（必填）" class="title-input"
                minlength="3" maxlength="100"
                show-word-limit
      />
      <div class="button-group">
        <el-button type="primary" @click="showForm('draft')">保存草稿</el-button>
        <el-button type="success" @click="showForm('publish')">发布文章</el-button>
      </div>
    </div>

    <!-- 编辑器 -->
    <md-editor class="mgb20" v-model="text" @on-upload-img="onUploadImg"/>

    <!-- 表单弹窗 -->
    <el-dialog v-model="isFormVisible" title="文章信息" width="600px">
      <el-form :model="form" :rules="rules" ref="formRef" label-width="120px">
        <el-form-item label="文章标题" prop="title">
          <el-input v-model="form.title" placeholder="请输入文章标题" minlength="3" maxlength="100"
                    show-word-limit/>
        </el-form-item>
        <el-form-item label="封面类型">
          <el-radio-group v-model="form.coverType">
            <el-radio :value="'custom'">自定义封面</el-radio>
            <el-radio :value="'random'">系统随机封面</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="文章封面上传" v-if="form.coverType === 'custom'">
          <!-- 自定义封面上传 -->
            <el-upload
                class="upload-demo"
                action=""
                list-type="picture-card"
                :on-preview="handlePreview"
                :on-remove="handleRemove"
                :file-list="fileList"
                :auto-upload="false"
                :before-upload="handleUpload"
                :show-file-list="true"
            >
              <el-icon class="avatar-uploader-icon">
                <Plus/>
              </el-icon>
              <i class="el-icon-plus" v-if="fileList.length === 0"></i>
            </el-upload>
          <!-- 放大查看图片的弹窗 -->
          <el-dialog
              v-model="dialogVisible"
              @close="closeDialog"
              :before-close="closeDialog"
              style="align-content: center; background: transparent;"
          >
            <img w-full :src="previewImage" alt="预览" class="preview-image"
                 style="object-fit: contain;width: 98%;margin-left: 1%"/>
          </el-dialog>
        </el-form-item>

        <el-form-item label="文章类别" prop="category">
          <el-select v-model="form.category" placeholder="请选择文章类别">
            <el-option label="技术" value="tech"/>
            <el-option label="生活" value="life"/>
            <el-option label="其他" value="other"/>
          </el-select>
        </el-form-item>

        <el-form-item label="添加标签">
          <el-select
              v-model="form.tags"
              multiple
              filterable
              allow-create
              default-first-option
              :reserve-keyword="false"
              placeholder="添加标签"
          >
            <el-option
                v-for="item in options"
                :key="item.value"
                :label="item.label"
                :value="item.value"
            />
          </el-select>
        </el-form-item>

        <el-form-item label="摘要">
          <el-input
              type="textarea"
              v-model="form.summary"
              placeholder="说点什么吧，太抽象了可不好"
              minlength="0" maxlength="300"
              show-word-limit
              :rows="3"
          />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer" style="display: flex; justify-content: flex-end;gap: 10px">
        <el-button type="danger" @click="submitForm(false)">直接发布</el-button>
        <el-button type="primary" @click="submitForm(true)">保存草稿</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script setup lang="ts" name="md">
import {ref} from "vue";
import MdEditor from "md-editor-v3";
import "md-editor-v3/lib/style.css";
import {ElMessage} from "element-plus";
import {Plus} from "@element-plus/icons-vue";


const text = ref("Hello Editor!");
const title = ref("");

const isFormVisible = ref(false);
const submitType = ref("draft"); // 区分保存草稿或发布

const form = ref({
  title: "",
  coverType: "random",
  category: "",
  tags: [],
  summary: "",
  status: "draft"// 默认设置为草稿
});
// 上传文件列表
const fileList = ref([]);
// 放大图片的对话框显示状态
const dialogVisible = ref(false);
// 放大图片的 URL
const previewImage = ref('');

// 文件上传前逻辑
const handleUpload = (file: File) => {
  const fileUrl = URL.createObjectURL(file);
  fileList.value = [{name: file.name, url: fileUrl}];
  return false; // 阻止自动上传
};
// 图片预览，点击图片后放大
const handlePreview = (file: any) => {
  previewImage.value = file.url;  // 设置预览图片的 URL
  dialogVisible.value = true;  // 显示放大查看对话框
};

// 删除图片
const handleRemove = (file: any) => {
  fileList.value = fileList.value.filter((item) => item.name !== file.name);
};

// 关闭对话框
const closeDialog = () => {
  dialogVisible.value = false;  // 隐藏对话框
};
const rules = {
  title: [{required: true, message: "请输入文章标题", trigger: "blur"}],
  category: [{required: true, message: "请选择文章类别", trigger: "change"}],
};

const onUploadImg = (files: any) => {
  console.log(files);
};

const showForm = (type: string) => {
  submitType.value = type;
  isFormVisible.value = true;
};

// 提交表单
const submitForm = (isDraft: boolean) => {
  if (isDraft) {
    form.value.status = 'draft'; // 设置为草稿
    console.log("保存草稿", form.value);
    ElMessage.success("草稿已保存！");
    // 在这里执行保存草稿的操作
  } else {
    form.value.status = 'published'; // 设置为已发布
    console.log("直接发布", form.value);
    ElMessage.success("文章已发布！");
    // 在这里执行发布文章的操作
  }
};
// 标签选项
const options = [
  {
    value: 'HTML',
    label: 'HTML',
  },
  {
    value: 'CSS',
    label: 'CSS',
  },
  {
    value: 'JavaScript',
    label: 'JavaScript',
  },
]
</script>

<style scoped>
.container {
  padding: 20px;
}

.title-bar {
  display: flex;
  align-items: center;
  margin-bottom: 20px;
}

.title-input {
  flex: 1;
  margin-right: 10px;
}

.button-group {
  display: flex;
  gap: 10px;
}

.upload-section {
  margin-top: 10px;
}
</style>

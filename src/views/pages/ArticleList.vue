<template>
  <div class="container">
    <!-- 搜索框 -->
    <el-form :model="searchParams" class="mb-4" label-width="80px">
      <el-row :gutter="20">
        <!-- 第一行 -->
        <el-col :span="8">
          <el-form-item label="关键字:">
            <el-input v-model="searchParams.keyword" placeholder="请输入关键字" />
          </el-form-item>
        </el-col>
        <el-col :span="8">
          <el-form-item label="分类:">
            <el-select v-model="searchParams.category" placeholder="选择分类" clearable>
              <el-option label="分类1" value="category1" />
              <el-option label="分类2" value="category2" />
              <el-option label="分类3" value="category3" />
            </el-select>
          </el-form-item>
        </el-col>
        <el-col :span="8">
          <el-form-item label="标签">
            <!-- 将标签输入框改为下拉框 -->
            <el-select v-model="searchParams.tag" placeholder="选择标签" clearable>
              <el-option label="标签1" value="标签1" />
              <el-option label="标签2" value="标签2" />
              <el-option label="标签3" value="标签3" />
            </el-select>
          </el-form-item>
        </el-col>
      </el-row>

      <!-- 第二行 -->
      <el-row :gutter="20">
        <el-col :span="8">
          <el-form-item label="发布时间">
            <el-date-picker
                v-model="searchParams.dateRange"
                type="daterange"
                range-separator="To"
                placeholder="选择发布时间范围"
                unlink-panels
                :suffix-icon="Calendar"
            />
          </el-form-item>
        </el-col>
        <el-col :span="8">
          <el-form-item>
            <el-button type="primary" @click="searchArticles" :icon="Search">搜索</el-button>
            <el-button type="warning" @click="resetSearch" style="margin-left: 10px;" :icon="Delete">重置</el-button>
          </el-form-item>
        </el-col>
      </el-row>
    </el-form>

    <!-- 文章列表 -->
    <el-table :data="paginatedArticles" style="width: 100%" border>
      <el-table-column prop="index" label="序号" width="80" />
      <el-table-column prop="title" label="标题" />
      <el-table-column prop="summary" label="摘要" />
      <el-table-column prop="category" label="分类" />
      <el-table-column label="封面">
        <template v-slot="scope">
          <img :src="scope.row.cover" alt="封面" style="width: 100%; height: 100%; object-fit: cover;" />
        </template>
      </el-table-column>
      <el-table-column prop="tags" label="标签" />
      <el-table-column prop="publishDate" label="发布时间" width="180" />
      <el-table-column prop="updateDate" label="修改时间" width="180" />

      <el-table-column label="操作" width="180">
        <template v-slot="scope">
          <el-button type="primary" size="small" @click="editArticle(scope.row)">修改</el-button>
          <el-button type="danger" size="small" @click="deleteArticle(scope.row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <!-- 分页 -->
<!--    <el-pagination-->
<!--        :current-page="currentPage"-->
<!--        :page-size="pageSize"-->
<!--        :total="filteredArticles.length"-->
<!--        @current-change="handlePageChange"-->
<!--        layout="total, prev, pager, next"-->
<!--    />-->
    <!-- 分页 -->
    <el-pagination
        :current-page="currentPage"
        :page-size="pageSize"
        :total="filteredArticles.length"
        @current-change="handlePageChange"
        layout="total, prev, pager, jumper, next"
    />

  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import {Calendar, Delete, Search} from "@element-plus/icons-vue";

interface Article {
  index: number;
  title: string;
  summary: string;
  category: string;
  cover: string;
  tags: string[];
  publishDate: string;
  updateDate: string;
}

const articles: Article[] = [
  { index: 1, title: "文章1", summary: "这是文章1的摘要", category: "category1", cover: "data:image/jpeg;base64,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", tags: ["标签1"], publishDate: "2024-01-01", updateDate: "2024-01-02" },
  { index: 2, title: "文章2", summary: "这是文章2的摘要", category: "category2", cover: "https://via.placeholder.com/50", tags: ["标签2"], publishDate: "2024-01-05", updateDate: "2024-01-06" },
  { index: 3, title: "文章3", summary: "这是文章3的摘要", category: "category1", cover: "https://via.placeholder.com/50", tags: ["标签3"], publishDate: "2024-02-01", updateDate: "2024-02-02" },
  { index: 4, title: "文章4", summary: "这是文章4的摘要", category: "category2", cover: "https://via.placeholder.com/50", tags: ["标签4"], publishDate: "2024-02-10", updateDate: "2024-02-11" },
  { index: 5, title: "文章5", summary: "这是文章5的摘要", category: "category3", cover: "https://via.placeholder.com/50", tags: ["标签5"], publishDate: "2024-03-01", updateDate: "2024-03-02" },
  // 更多数据...
];

const searchParams = ref({
  keyword: "",
  category: "",
  tag: "",
  dateRange: [],
});

const filteredArticles = ref<Article[]>(articles);
const paginatedArticles = ref<Article[]>([]);
const currentPage = ref(1);
const pageSize = ref(5);

const searchArticles = () => {
  // 根据搜索条件过滤文章
  filteredArticles.value = articles.filter(article => {
    return (
        (searchParams.value.keyword ? article.title.includes(searchParams.value.keyword) : true) &&
        (searchParams.value.category ? article.category === searchParams.value.category : true) &&
        (searchParams.value.tag ? article.tags.includes(searchParams.value.tag) : true) &&
        (searchParams.value.dateRange.length ? new Date(article.publishDate) >= new Date(searchParams.value.dateRange[0]) && new Date(article.publishDate) <= new Date(searchParams.value.dateRange[1]) : true)
    );
  });

  // 重置分页
  handlePageChange(1);
};

const handlePageChange = (page: number) => {
  currentPage.value = page;
  const start = (page - 1) * pageSize.value;
  const end = page * pageSize.value;
  paginatedArticles.value = filteredArticles.value.slice(start, end);
};

// 重置表单
const resetSearch = () => {
  searchParams.value = {
    keyword: "",
    category: "",
    tag: "",
    dateRange: [],
  };
  // 重置分页
  handlePageChange(1);
};

// 修改文章
const editArticle = (article: Article) => {
  console.log("修改文章:", article);
  // 跳转到编辑界面，可以通过路由或其他方式
};

// 删除文章
const deleteArticle = (article: Article) => {
  console.log("删除文章:", article);
  // 删除文章的操作，通常会发起 API 请求
};
</script>

<style scoped>
.mb-4 {
  margin-bottom: 16px;
}
.container {
  padding: 20px;
}


</style>

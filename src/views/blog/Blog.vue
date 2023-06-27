<script setup>

import {getCurrentInstance, reactive, ref, toRefs} from "vue";
import {addBlog, listBlog, updateBlog} from "@/api/blog";

const {proxy} = getCurrentInstance();
const {blog_type, yon} = proxy.useDict("blog_type", "yon");

const dateRange = ref([]);
const blogList = ref([]);
const total = ref(0);

const showSearch = ref(true);
const loading = ref(false);
const open = ref(false);
const title = ref("");

const data = reactive({
  form: {},
  queryParams: {
    pageNum: 1,
    pageSize: 10,
    title: undefined,
    blogType: undefined,
    author: undefined,
    isPrivate: undefined,
    isOriginal: undefined,
    isTop: undefined,
    isDeleted: undefined
  },
  rules: {
    roleName: [{required: true, message: "角色名称不能为空", trigger: "blur"}],
    roleKey: [{required: true, message: "权限字符不能为空", trigger: "blur"}],
    roleSort: [{required: true, message: "角色顺序不能为空", trigger: "blur"}]
  },
})

const {form, queryParams, rules} = toRefs(data);

function handleQuery() {
  queryParams.value.pageNum = 1;
  getList();
  console.log(blogList.value);
}

function getList() {
  loading.value = true;
  listBlog(proxy.addDateRange(queryParams.value, dateRange.value)).then(response => {
    blogList.value = response.rows;
    total.value = response.total;
    loading.value = false;
  }).catch(error => {
    proxy.$modal.msgError("查询失败！");
    console.log(error);
  })
}

function resetQuery(){
  dateRange.value = [];
  proxy.resetForm("queryRef");
  handleQuery();
}

function handleAdd(){
  reset();
  open.value = true;
  title.value = "添加博客"
}

function reset(){
  form.value = {
    title: undefined,
    blogType: undefined,
    isPrivate: undefined,
    isOriginal: undefined,
    originalLink:undefined,
    isTop: undefined,
    isDeleted: undefined,
    blog:{
      content:undefined,
    },
  }
  proxy.resetForm("blogRef");
}

function submitForm(){
  proxy.$refs["blogRef"].validate(valid => {
    if (valid){
      if (form.value.id != undefined){
        updateBlog(form.value).then(resp => {
          proxy.$modal.msgSuccess("修改成功");
          open.value = false;
          getList();
        })
      } else {
        addBlog(form.value).then(resp => {
          proxy.$modal.msgSuccess("新增成功");
          open.value = false;
          getList();
        })
      }
    }
  })
}

function cancel(){

}

function handleUpdate(){
  
}

function handleDelete(){
  
}

function handleExport(){
  
}

function handleSelectionChange(){

}
</script>

<template>
  <div class="app-container">
    <el-form :model="queryParams" ref="queryRef" v-show="showSearch" :inline="true" label-width="68px">
      <el-form-item label="标题" prop="title">
        <el-input
            v-model="queryParams.title"
            placeholder="请输入标题！"
            clearable
            style="width: 240px"
            @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="作者" prop="author">
        <el-input
            v-model="queryParams.author"
            placeholder="请输入作者名称！"
            clearable
            style="width: 240px"
            @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="类型" prop="blogType">
        <el-select
            v-model="queryParams.blogType"
            placeholder="请选择博客类型！"
            clearable
            style="width: 240px">
          <el-option
              v-for="dict in blog_type"
              :key="dict.value"
              :label="dict.label"
              :value="dict.value"/>
        </el-select>
      </el-form-item>
      <el-form-item label="创建时间" style="width: 308px">
        <el-date-picker
            v-model="dateRange"
            value-format="YYYY-MM-DD"
            type="daterange"
            range-separator="-"
            start-placeholder="开始日期"
            end-placeholder="结束日期"
        ></el-date-picker>
      </el-form-item>
      <el-form-item label="是否原创" style="width: 240px">
        <el-radio-group v-model="queryParams.isOriginal">
          <el-radio
              v-for="dict in yon"
              :key="dict.value"
              :label="dict.label"
              :value="dict.value">
            {{ dict.label }}
          </el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="是否私密" style="width: 240px">
        <el-radio-group v-model="queryParams.isPrivate">
          <el-radio
              v-for="dict in yon"
              :key="dict.value"
              :label="dict.label"
              :value="dict.value">
            {{ dict.label }}
          </el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="是否置顶" style="width: 240px">
        <el-radio-group v-model="queryParams.isTop">
          <el-radio
              v-for="dict in yon"
              :key="dict.value"
              :label="dict.label"
              :value="dict.value">
            {{ dict.label }}
          </el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="是否删除" style="width: 240px">
        <el-radio-group v-model="queryParams.isDeleted">
          <el-radio
              v-for="dict in yon"
              :key="dict.value"
              :label="dict.label"
              :value="dict.value">
            {{ dict.label }}
          </el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="Search" @click="handleQuery">搜索</el-button>
        <el-button icon="Refresh" @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>

    <el-row :gutter="10" class="mb8">
      <el-col :span="1.5">
        <el-button
            type="primary"
            plain
            icon="Plus"
            @click="handleAdd"
            v-hasPermi="['blog:blog:add']"
        >新增</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
            type="success"
            plain
            icon="Edit"
            :disabled="single"
            @click="handleUpdate"
            v-hasPermi="['blog:blog:edit']"
        >修改</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
            type="danger"
            plain
            icon="Delete"
            :disabled="multiple"
            @click="handleDelete"
            v-hasPermi="['blog:blog:remove']"
        >删除</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
            type="warning"
            plain
            icon="Download"
            @click="handleExport"
            v-hasPermi="['blog:blog:export']"
        >导出</el-button>
      </el-col>
      <right-toolbar v-model:showSearch="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>

    <el-table v-loading="loading" :data="blogList" @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55" align="center" />
      <el-table-column label="标题"  align="center" prop="title" :show-overflow-tooltip="true" width="120" />
      <el-table-column label="博客类型"  align="center" prop="blogType" width="100" />
      <el-table-column label="作者"  align="center" prop="author" width="100" />
      <el-table-column label="原始链接"  align="center" prop="originalLink" width="150" />
      <el-table-column label="是否置顶"  align="center" prop="isTop" width="80" />
      <el-table-column label="是否原创"  align="center" prop="isOriginal" width="80" />
      <el-table-column label="是否私密" align="center"  prop="isPrivate" width="80" />
      <el-table-column label="创建时间" align="center" prop="createTime">
        <template #default="scope">
          <span>{{ parseTime(scope.row.createTime) }}</span>
        </template>
      </el-table-column>

      <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
        <template #default="scope">
          <el-tooltip content="修改" placement="top" v-if="scope.row.roleId !== 1">
            <el-button
                type="text"
                icon="Edit"
                @click="handleUpdate(scope.row)"
                v-hasPermi="['system:role:edit']"
            ></el-button>
          </el-tooltip>
          <el-tooltip content="删除" placement="top" v-if="scope.row.roleId !== 1">
            <el-button
                type="text"
                icon="Delete"
                @click="handleDelete(scope.row)"
                v-hasPermi="['system:role:remove']"
            ></el-button>
          </el-tooltip>
          <el-tooltip content="置顶" placement="top" v-if="scope.row.roleId !== 1">
            <el-button
                type="text"
                icon="CircleCheck"
                @click="handleTop(scope.row)"
                v-hasPermi="['system:role:edit']"
            ></el-button>
          </el-tooltip>
          <el-tooltip content="查看详情" placement="top" v-if="scope.row.roleId !== 1">
            <el-button
                type="text"
                icon="User"
                @click="handleBlogInfo(scope.row)"
                v-hasPermi="['system:role:edit']"
            ></el-button>
          </el-tooltip>
        </template>
      </el-table-column>

    </el-table>

    <pagination
        v-show="total > 0"
        :total="total"
        v-model:page="queryParams.pageNum"
        v-model:limit="queryParams.pageSize"
        @pagination="getList"
    />

    <el-dialog :title="title" v-model="open" width="500px" append-to-body>
      <el-form ref="blogRef" :model="form" :rules="rules" label-width="100px" >
        <el-form-item label="标题" prop="title" >
          <el-input v-model="form.title" placeholder="请输入标题！"/>
        </el-form-item>
        <el-form-item label="类型" prop="blogType">
          <el-select
              v-model="form.blogType"
              placeholder="请选择博客类型！"
              clearable
              style="width: 240px">
            <el-option
                v-for="dict in blog_type"
                :key="dict.value"
                :label="dict.label"
                :value="dict.value"/>
          </el-select>
        </el-form-item>
        <el-form-item label="是否原创" prop="isOriginal" style="width: 240px">
          <el-radio-group v-model="form.isOriginal">
            <el-radio
                v-for="dict in yon"
                :key="dict.value"
                :label="dict.value">
              {{ dict.label }}
            </el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="是否私密" prop="isPrivate" style="width: 240px">
          <el-radio-group v-model="form.isPrivate">
            <el-radio
                v-for="dict in yon"
                :key="dict.value"
                :label="dict.value"
                >
              {{ dict.label }}
            </el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="是否置顶" prop="isTop" style="width: 240px">
          <el-radio-group v-model="form.isTop">
            <el-radio
                v-for="dict in yon"
                :key="dict.value"
                :label="dict.value">
              {{ dict.label }}
            </el-radio>
          </el-radio-group>
        </el-form-item>
      </el-form>
      <el-form-item label="原始链接" prop="originalLink"  v-if="form.isOriginal == false">
        <el-input v-model="form.originalLink" placeholder="请输入原始链接！"/>
      </el-form-item>
      <el-form-item label="内容" prop="blog.content">
        <el-input v-model="form.blog.content" placeholder="请输入博客内容！" />
      </el-form-item>

      <template #footer>
        <div class="dialog-footer">
          <el-button type="primary" @click="submitForm">确 定</el-button>
          <el-button @click="cancel">取 消</el-button>
        </div>
      </template>

    </el-dialog>

  </div>
</template>
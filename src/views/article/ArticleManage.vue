<script setup>
import {
    Edit,
    Delete
} from '@element-plus/icons-vue'

import { ref } from 'vue'

//文章分类数据模型
const categorys = ref([
    {
        "id": 3,
        "categoryName": "美食",
        "categoryAlias": "my",
        "createTime": "2023-09-02 12:06:59",
        "updateTime": "2023-09-02 12:06:59"
    },
    {
        "id": 4,
        "categoryName": "娱乐",
        "categoryAlias": "yl",
        "createTime": "2023-09-02 12:08:16",
        "updateTime": "2023-09-02 12:08:16"
    },
    {
        "id": 5,
        "categoryName": "军事",
        "categoryAlias": "js",
        "createTime": "2023-09-02 12:08:33",
        "updateTime": "2023-09-02 12:08:33"
    }
])

//用户搜索时选中的分类id
const categoryId = ref('')

//用户搜索时选中的发布状态
const state = ref('')

//文章列表数据模型
const articles = ref([
    {
        "id": 5,
        "title": "陕西旅游攻略",
        "content": "兵马俑,华清池,法门寺,华山...爱去哪去哪...",
        "coverImg": "https://big-event-gwd.oss-cn-beijing.aliyuncs.com/9bf1cf5b-1420-4c1b-91ad-e0f4631cbed4.png",
        "state": "草稿",
        "categoryId": 2,
        "createTime": "2023-09-03 11:55:30",
        "updateTime": "2023-09-03 11:55:30"
    },
    {
        "id": 5,
        "title": "陕西旅游攻略",
        "content": "兵马俑,华清池,法门寺,华山...爱去哪去哪...",
        "coverImg": "https://big-event-gwd.oss-cn-beijing.aliyuncs.com/9bf1cf5b-1420-4c1b-91ad-e0f4631cbed4.png",
        "state": "草稿",
        "categoryId": 2,
        "createTime": "2023-09-03 11:55:30",
        "updateTime": "2023-09-03 11:55:30"
    },
    {
        "id": 5,
        "title": "陕西旅游攻略",
        "content": "兵马俑,华清池,法门寺,华山...爱去哪去哪...",
        "coverImg": "https://big-event-gwd.oss-cn-beijing.aliyuncs.com/9bf1cf5b-1420-4c1b-91ad-e0f4631cbed4.png",
        "state": "草稿",
        "categoryId": 2,
        "createTime": "2023-09-03 11:55:30",
        "updateTime": "2023-09-03 11:55:30"
    },
])

//分页条数据模型
const pageNum = ref(1)//当前页
const total = ref(20)//总条数
const pageSize = ref(3)//每页条数

//当每页条数发生了变化，调用此函数
const onSizeChange = (size) => {
    pageSize.value = size
    articleList();
}
//当前页码发生变化，调用此函数
const onCurrentChange = (num) => {
    pageNum.value = num
    articleList();

}
import { articleCategoryListService, articleListService, articleAddService, articleUpdateService, articleDeleteService } from '@/api/article.js';
const articleCategoryList = async () => {
    let res = await articleCategoryListService();
    categorys.value = res.data;
}


// 获取文章列表数据
const articleList = async () => {
    let params = {
        pageNum: pageNum.value,
        pageSize: pageSize.value,
        categoryId: categoryId.value ? categoryId.value : null,
        state: state.value ? state.value : null

    }
    let res = await articleListService(params)
    total.value = res.data.total;
    articles.value = res.data.items;
    const myMap = new Map();
    for (let j = 0; j < categorys.value.length; j++) {
        myMap.set(categorys.value[j].id, categorys.value[j].categoryName);
    }
    for (let i = 0; i < articles.value.length; i++) {
        let article = articles.value[i];
        article.categoryName = myMap.get(article.categoryId)
    }

    //处理数据,给数据模型扩展一个属性categoryName,分类名称
    // for (let i = 0; i < articles.value.length; i++) {
    //     let article = articles.value[i];
    //     for (let j = 0; j < categorys.value.length; j++) {
    //         if (article.categoryId == categorys.value[j].id) {
    //             article.categoryName = categorys.value[j].categoryName;
    //         }
    //     }
    // }



}
articleCategoryList();
articleList();

const clearALL = () => {
    categoryId.value = '';
    state.value = '';
    articleList()
}
import { QuillEditor } from '@vueup/vue-quill'
import '@vueup/vue-quill/dist/vue-quill.snow.css'
import { Plus } from '@element-plus/icons-vue'
//控制抽屉是否显示
const visibleDrawer = ref(false)
//添加表单数据模型
const articleModel = ref({
    title: '',
    categoryId: '',
    coverImg: '',
    content: '',
    state: ''
})
import { useTokenStore } from '@/stores/token.js';
import { ElMessage, ElMessageBox } from 'element-plus';
let tokenStore = useTokenStore()

const uploadSuccess = (res) => {
    articleModel.value.coverImg = res.data;
    console.log('res.data :>> ', res.data);
}
const addArticle = async (state) => {
    articleModel.value.state = state;
    let res = await articleAddService(articleModel.value)
    console.log('res :>> ', res);
    ElMessage.success(res.msg ? res.msg : "添加成功")
    visibleDrawer.value = false;
    articleList();
}

const deleteArticle = (row) => {

    ElMessageBox.confirm(
        '是否删除?',
        '温馨提示',
        {
            confirmButtonText: '是',
            cancelButtonText: '否',
            type: 'warning',
        }
    )
        .then(async () => {
            let res = await articleDeleteService(row.id);
            console.log('res :>> ', res);
            ElMessage({

                type: 'success',
                message: '删除成功',
            })
            articleList();
        })
        .catch(() => {
            ElMessage({
                type: 'info',
                message: '取消删除',
            })
        })
}
let myTitle = ref('')
let showDrawer = (row) => {
    visibleDrawer.value = true;
    myTitle.value = '编辑文章';
    articleModel.value.id = row.id
    articleModel.value.title = row.title
    articleModel.value.categoryId = row.categoryId
    articleModel.value.categoryName = row.categoryName
    articleModel.value.coverImg = row.coverImg
    articleModel.value.content = row.content

}
const updateArticle = async (state) => {
    articleModel.value.state = state;
    let res = await articleUpdateService(articleModel.value);
    console.log('res :>> ', res);
    ElMessage.success(res.msg ? res.msg : "更新成功")
    visibleDrawer.value = false;
    articleList();
}


</script>
<template>
    <el-card class="page-container">
        <template #header>
            <div class="header">
                <span>文章管理</span>
                <div class="extra">
                    <el-button type="primary" @click="visibleDrawer = true; myTitle = '添加文章'">添加文章</el-button>
                </div>
            </div>
        </template>
        <!-- 搜索表单 -->
        <el-form inline>
            <el-form-item label="文章分类：">
                <el-select placeholder="请选择" v-model="categoryId" style="width: 100px;">
                    <el-option v-for=" c in categorys" :key="c.id" :label="c.categoryName" :value="c.id">
                    </el-option>
                </el-select>
            </el-form-item>

            <el-form-item label="发布状态：">
                <el-select placeholder="请选择" v-model="state" style="width: 100px;">
                    <el-option label="已发布" value="已发布"></el-option>
                    <el-option label="草稿" value="草稿"></el-option>
                </el-select>
            </el-form-item>
            <el-form-item>
                <el-button type="primary" @click="articleList">搜索</el-button>
                <el-button @click="clearALL">重置</el-button>
            </el-form-item>
        </el-form>
        <!-- 文章列表 -->
        <el-table :data="articles" style="width: 100%">
            <el-table-column label="文章标题" width="400" prop="title"></el-table-column>
            <el-table-column label="分类" prop="categoryName"></el-table-column>
            <el-table-column label="发表时间" prop="createTime"> </el-table-column>
            <el-table-column label="状态" prop="state"></el-table-column>
            <el-table-column label="操作" width="100">
                <template #default="{ row }">
                    <el-button :icon="Edit" circle plain type="primary" @click="showDrawer(row)"></el-button>
                    <el-button :icon="Delete" circle plain type="danger" @click="deleteArticle(row)"></el-button>
                </template>
            </el-table-column>
            <template #empty>
                <el-empty description="没有数据" />
            </template>
        </el-table>
        <!-- 分页条 -->
        <el-pagination v-model:current-page="pageNum" v-model:page-size="pageSize" :page-sizes="[3, 5, 10, 15]"
            layout="jumper, total, sizes, prev, pager, next" background :total="total" @size-change="onSizeChange"
            @current-change="onCurrentChange" style="margin-top: 20px; justify-content: flex-end" />
    </el-card>
    <!-- 抽屉 -->
    <el-drawer v-model="visibleDrawer" :title="myTitle" direction="rtl" size="50%">
        <!-- 添加文章表单 -->
        <el-form :model="articleModel" label-width="100px">
            <el-form-item label="文章标题">
                <el-input v-model="articleModel.title" placeholder="请输入标题"></el-input>
            </el-form-item>
            <el-form-item label="文章分类">
                <el-select placeholder="请选择" v-model="articleModel.categoryId">
                    <el-option v-for="c in categorys" :key="c.id" :label="c.categoryName" :value="c.id">
                    </el-option>
                </el-select>
            </el-form-item>
            <el-form-item label="文章封面">
                <!-- auto-upload:是否自动上传

                    action: 服务器接口路径

                    name: 上传的文件字段名

                    headers: 设置上传的请求头

                    on-success: 上传成功的回调函数

                    -->
                <!-- action="/api/upload"
                    name="file" :headers="{ 'Authorization': tokenStore.token }" :on-success="uploadSuccess" -->
                <el-upload class="avatar-uploader" :auto-upload="true" :show-file-list="false" action="/api/upload"
                    name="file" :headers="{ 'Authorization': tokenStore.token }" :on-success="uploadSuccess">
                    <img v-if="articleModel.coverImg" :src="articleModel.coverImg" class="avatar" />
                    <el-icon v-else class="avatar-uploader-icon">
                        <Plus />
                    </el-icon>
                </el-upload>
            </el-form-item>
            <el-form-item label="文章内容">
                <div class="editor">
                    <quill-editor theme="snow" v-model:content="articleModel.content" contentType="html">
                    </quill-editor>
                </div>
            </el-form-item>
            <el-form-item>
                <el-button type="primary"
                    @click="myTitle == '添加文章' ? addArticle('已发布') : updateArticle('已发布')">发布</el-button>
                <el-button type="info"
                    @click="myTitle == '添加文章' ? addArticle('草稿') : updateArticle('草稿')">草稿</el-button>
            </el-form-item>
        </el-form>
    </el-drawer>
</template>
<style lang="scss" scoped>
.page-container {
    min-height: 100%;
    box-sizing: border-box;

    .header {
        display: flex;
        align-items: center;
        justify-content: space-between;
    }
}

/* 抽屉样式 */
.avatar-uploader {
    :deep() {
        .avatar {
            width: 178px;
            height: 178px;
            display: block;
        }

        .el-upload {
            border: 1px dashed var(--el-border-color);
            border-radius: 6px;
            cursor: pointer;
            position: relative;
            overflow: hidden;
            transition: var(--el-transition-duration-fast);
        }

        .el-upload:hover {
            border-color: var(--el-color-primary);
        }

        .el-icon.avatar-uploader-icon {
            font-size: 28px;
            color: #8c939d;
            width: 178px;
            height: 178px;
            text-align: center;
        }
    }
}

.editor {
    width: 100%;

    :deep(.ql-editor) {
        min-height: 200px;
    }
}
</style>
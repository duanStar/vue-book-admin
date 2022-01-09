<template>
  <el-form ref="postForm" :model="postForm" :rules="rules">
    <Sticky :class-name="'sub-navbar'">
      <el-button v-if="!isEdit" @click="showGuide">显示帮助</el-button>
      <el-button
        :loading="loading"
        type="success"
        style="margin-left: 10px;"
        @click="submitForm"
      >
        {{ isEdit ? '编辑电子书' : '新增电子书' }}
      </el-button>
    </Sticky>
    <div class="detail-container">
      <el-row>
        <Warning />
        <el-col :span="24">
          <EbookUpload ref="ebookUpload" :file-list="fileList" :disabled="isEdit" @onSuccess="onUploadSuccess" @onError="onUploadError" @onRemove="onUploadRemove" />
        </el-col>
        <el-col :span="24">
          <el-form-item prop="title">
            <MdInput v-model="postForm.title" :maxlength="100" name="name" required>书名</MdInput>
          </el-form-item>
          <el-row>
            <el-col :span="12">
              <el-form-item prop="author" label="作者:" :label-width="labelWidth">
                <el-input v-model="postForm.author" placeholder="作者" />
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item prop="publisher" label="出版社:" :label-width="labelWidth">
                <el-input v-model="postForm.publisher" placeholder="出版社" />
              </el-form-item>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="12">
              <el-form-item prop="language" label="语言:" :label-width="labelWidth">
                <el-input v-model="postForm.language" placeholder="语言" />
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item prop="rootFile" label="根文件:" :label-width="labelWidth">
                <el-input v-model="postForm.rootFile" placeholder="根文件" disabled />
              </el-form-item>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="12">
              <el-form-item prop="filePath" label="文件路径:" :label-width="labelWidth">
                <el-input v-model="postForm.filePath" placeholder="文件路径" disabled />
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item prop="unzipPath" label="解压路径:" :label-width="labelWidth">
                <el-input v-model="postForm.unzipPath" placeholder="解压路径" disabled />
              </el-form-item>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="12">
              <el-form-item prop="coverPath" label="封面路径:" :label-width="labelWidth">
                <el-input v-model="postForm.coverPath" placeholder="封面路径" disabled />
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item prop="originalName" label="文件名称:" :label-width="labelWidth">
                <el-input v-model="postForm.originalName" placeholder="文件名称" disabled />
              </el-form-item>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="24">
              <el-form-item prop="cover" label="封面:" :label-width="labelWidth">
                <a v-if="postForm.cover" :href="postForm.cover" target="_blank">
                  <img :src="postForm.cover" class="preview-img">
                </a>
                <span v-else>无</span>
              </el-form-item>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="24">
              <el-form-item :label-width="labelWidth" label="目录:">
                <div
                  v-if="contentsTree && contentsTree.length > 0"
                  class="contents-wrapper"
                >
                  <el-tree :data="contentsTree" @node-click="onContentClick" />
                </div>
                <span v-else>无</span>
              </el-form-item>
            </el-col>
          </el-row>
        </el-col>
      </el-row>
    </div>
  </el-form>
</template>

<script>
import Sticky from '../../../components/Sticky'
import Warning from './Warning.vue'
import EbookUpload from '../../../components/EbookUpload'
import MdInput from '../../../components/MDinput'
import { createBook, getBook, updateBook } from '@/api/book'

const fields = {
  title: '标题',
  author: '作者',
  publisher: '出版社',
  language: '语言'
}

export default {
  name: 'Detail',
  components: {
    Sticky,
    Warning,
    EbookUpload,
    MdInput
  },
  props: {
    isEdit: Boolean
  },
  data() {
    const validateRequire = (rule, value, callback) => {
      if (value.length === 0) {
        callback(new Error(fields[rule.field] + '必须填写'))
      } else {
        callback()
      }
    }
    return {
      loading: false,
      postForm: {
        title: '',
        author: '',
        publisher: '',
        language: '',
        rootFile: '',
        filePath: '',
        unzipPath: '',
        coverPath: '',
        filename: '',
        cover: '',
        contents: [],
        originalName: ''
      },
      fileList: [],
      labelWidth: '120px',
      contentsTree: [],
      rules: {
        title: [{ validator: validateRequire }],
        author: [{ validator: validateRequire }],
        publisher: [{ validator: validateRequire }],
        language: [{ validator: validateRequire }]
      }
    }
  },
  mounted() {
    if (this.isEdit) {
      const filename = this.$route.params.filename
      this.getBookData(filename)
    }
  },
  methods: {
    getBookData(filename) {
      getBook(filename).then(res => {
        this.setData(res.data.book)
      })
    },
    setData(data) {
      const { title, author, publisher, language, rootFile, cover, url, originalName, contents, filename, coverPath, unzipPath, filePath, contentsTree, fileName } = data
      this.postForm = {
        ...this.postForm,
        title,
        author,
        publisher,
        language,
        rootFile,
        coverPath,
        cover,
        contents,
        unzipPath,
        url,
        originalName: originalName || filename || fileName,
        filename: filename || fileName,
        filePath
      }
      this.contentsTree = contentsTree
      this.fileList = [{ name: originalName, url }]
    },
    setDefault() {
      this.postForm = {
        title: '',
        author: '',
        publisher: '',
        language: '',
        rootFile: '',
        filePath: '',
        unzipPath: '',
        coverPath: '',
        filename: '',
        cover: '',
        contents: [],
        originalName: ''
      }
      this.contentsTree = []
      this.fileList = []
      this.$refs.ebookUpload.clearFiles()
      this.$refs.postForm.resetFields()
    },
    showGuide() {},
    submitForm() {
      const onSuccess = (msg) => {
        this.$notify({
          title: '操作成功',
          message: msg,
          type: 'success',
          duration: 2000
        })
        this.loading = false
      }
      if (this.loading) return
      this.loading = true
      this.$refs.postForm.validate((valid, fields) => {
        if (valid) {
          const book = { ...this.postForm }
          if (!this.isEdit) {
            createBook(book).then(res => {
              const { msg } = res
              onSuccess(msg)
              this.setDefault()
            }).catch(() => {
              this.setDefault()
              this.loading = false
            })
          } else {
            updateBook(book).then(res => {
              const { msg } = res
              onSuccess(msg)
            }).catch(() => {
              this.loading = false
            })
          }
        } else {
          const message = fields[Object.keys(fields)[0]][0].message
          this.$message({ message, type: 'error' })
          this.loading = false
        }
      })
    },
    onUploadSuccess(data) {
      this.setData(data)
    },
    onUploadError() {},
    onUploadRemove() {
      this.setDefault()
    },
    onContentClick(data) {
      if (data.text) {
        window.open(data.text)
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.detail-container {
  padding: 40px 50px 20px;
}
.preview-img {
  height: 270px;
  width: 200px;
}
</style>

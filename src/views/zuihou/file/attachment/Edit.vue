<template>
  <el-dialog
    v-el-drag-dialog
    :title="title"
    :type="type"
    :width="width"
    top="50px"
    :close-on-click-modal="false"
    :close-on-press-escape="false"
    :visible.sync="isVisible"
    @dragDialog="handleDrag"
  >
    <el-form
      ref="form"
      :model="attachment"
      :rules="rules"
      label-position="right"
      label-width="100px"
    >
      <el-form-item
        :label="$t(&quot;table.attachment.bizId&quot;)"
        prop="bizId"
      >
        <el-input v-model="attachment.bizId" />
      </el-form-item>

      <el-form-item
        :label="$t(&quot;table.attachment.bizType&quot;)"
        prop="bizType"
      >
        <el-input v-model="attachment.bizType" />
      </el-form-item>
      <el-form-item
        label="文件"
        prop="fileLength"
      >
        <fileUpload
          ref="fileRef"
          :auto-upload="false"
          @setId="setIdAndSubmit"
          @fileLengthVaild="fileLengthVaild"
        >
          <el-button
            slot="trigger"
            size="small"
            type="primary"
          >
            选取文件
          </el-button>
          <div
            slot="tip"
            class="el-upload__tip"
          >
            文件不超过500MB
          </div>
        </fileUpload>
      </el-form-item>
    </el-form>
    <div
      slot="footer"
      class="dialog-footer"
    >
      <el-button
        type="warning"
        plain
        @click="isVisible = false"
      >
        {{ $t('common.cancel') }}
      </el-button>
      <el-button
        type="primary"
        plain
        @click="submitForm"
      >
        {{ $t('common.confirm') }}
      </el-button>
    </div>
  </el-dialog>
</template>
<script>
import elDragDialog from '@/directive/el-drag-dialog'
import fileUpload from "@/components/zuihou/fileUpload"

export default {
  name: 'AttachmentEdit',
  directives: { elDragDialog, fileUpload },
  components: { fileUpload },
  props: {
    dialogVisible: {
      type: Boolean,
      default: false
    },
    type: {
      type: String,
      default: 'add'
    }
  },
  data () {
    return {
      attachment: this.initAttachment(),
      screenWidth: 0,
      width: this.initWidth(),
      fileLength: 0,
      rules: {
        bizType: [
          { required: true, message: this.$t('rules.require'), trigger: 'blur' },
          { min: 0, max: 255, message: this.$t('rules.range0to255'), trigger: 'blur' },
        ],
        bizId: { min: 0, max: 255, message: this.$t('rules.range0to255'), trigger: 'blur' },
        fileLength: {          required: true, trigger: "change",
          validator: (rule, value, callback) => {
            const vm = this;
            if (vm.fileLength === 0) {
              callback(new Error("请上传文件"))
            } else if (vm.fileLength > 5) {
              callback(new Error("一次性只能上传5个文件"))
            } else {
              callback()
            }
          }
        }
      }
    }
  },
  computed: {
    isVisible: {
      get () {
        return this.dialogVisible
      },
      set () {
        this.close()
        this.reset()
      }
    },
    title () {
      return this.$t('common.upload')
    }
  },
  watch: {

  },
  mounted () {
    window.onresize = () => {
      return (() => {
        this.width = this.initWidth()
      })()
    }
  },
  methods: {
    initAttachment () {
      return {
        id: '',
        bizId: '',
        bizType: '',
        file: null,
        isSingle: false
      }
    },
    handleDrag () {
      console.log(`我被拖动了`)
    },
    // 附件长度校验
    fileLengthVaild (data) {
      const vm = this;
      vm.fileLength = data || 0;
    },
    initWidth () {
      this.screenWidth = document.body.clientWidth
      if (this.screenWidth < 991) {
        return '90%'
      } else if (this.screenWidth < 1400) {
        return '45%'
      } else {
        return '800px'
      }
    },
    setAttachment (val) {
      const vm = this
      if (val) {
        vm.attachment = { ...val }
      }
    },
    close () {
      this.$emit('close')
      this.$refs.fileRef.init("", "")
    },
    reset () {
      // 先清除校验，再清除表单，不然有奇怪的bug
      this.$refs.form.clearValidate()
      this.$refs.form.resetFields()
      this.attachment = this.initAttachment()
    },
    submitForm () {
      const vm = this
      this.$refs.form.validate((valid) => {
        if (valid) {
          vm.editSubmit()
        } else {
          return false
        }
      })
    },
    editSubmit () {
      const vm = this
      vm.$refs.fileRef.submitFile(this.attachment.bizId, this.attachment.bizType)
    },
    setIdAndSubmit (bizId, url) {
      debugger
      const vm = this
      console.log(bizId, url)
      vm.isVisible = false
      vm.$message({
        message: vm.$t('tips.createSuccess'),
        type: 'success'
      })
      vm.$emit('success')
    }

  }
}
</script>
<style lang="scss" scoped>
</style>

<template>
  <div :class="`mb-3 ${divClass}`">
    <s-label :label="label" :required="required" />
    <div class="input-group mb-3">
      <input
        ref="fileInput"
        @change="handleFileChange"
        type="file"
        class="form-control"
        :class="{ 'is-invalid': hasError }"
        :multiple="multiple"
        :accept="acceptedTypes"
      />
      <div class="invalid-feedback" v-if="hasError">
        {{ error }}
      </div>
    </div>
  </div>
</template>

<script>
import { defineComponent } from 'vue'
import { validateFile } from '@/rule/validate'

export default defineComponent({
  name: 'TheInputFile',
  inheritAttrs: false,

  props: {
    ref: String,
    divClass: String,
    label: String,
    required: Boolean,
    selectedFile: File,
    multiple: { type: Boolean, default: false },
    acceptedTypes: Array
  },

  data: () => ({
    error: null,
    hasError: false,
  }),

  methods: {
    handleFileChange(event) {
      const selectedFile = event.target.files[0];
      this.$emit('fileSelected', selectedFile);
    },

    update() {
      if(this.selectedFile) {
        this.$refs.fileInput.value = null;

        const dataTransfer = new DataTransfer();
        dataTransfer.items.add(new File([this.selectedFile], this.selectedFile.name));
        this.$refs.fileInput.files = dataTransfer.files;
      }
    },

    updateValue() {
      validateFile(this.inputValue, this.required, this, 'Este campo é obrigatório')
      this.update()
    },
  },

  created() {
    this.inputValue = this.modelValue
  },

  watch: {
    selectedFile() {
      this.updateValue()
    }
  }
})
</script>

<style></style>

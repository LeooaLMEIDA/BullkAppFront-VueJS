<template>
  <div class="m-3">
    <s-title :title="title" :breadcrumb="true" />
    <div class="card card-body mx-2">
      <form ref="form" @submit.prevent="submitForm">
        <div class="row">
          <s-input-text v-model="object.descricao" ref="descricao" maxlength="40" divClass="col-md-5" label="Descrição"
            placeholder="" required />
          <s-select v-model="object.status" divClass="col-md-2" label="Status" :items="status" :clearable="false" />
          <s-select v-model="object.grpMusculos" divClass="col-md-5" label="Grupo Muscular" :items="grupoMusculares"
            :clearable="false" />
          <s-input-text v-model="object.idAparelho" ref="idAparelho" maxlength="40" divClass="col-md-2" label="Aparelho"
            placeholder="" required />
          <s-input-text v-model="descricaoAparelho" ref="descricaoAparelho" maxlength="40" divClass="col-md-10" isDisabled
            label="Descrição Aparelho" placeholder="" />
          <s-input-file :selectedFile="object.file" @fileSelected="handleSelectedFile" ref="image" divClass="col-md-12"
            label="Imagem" acceptedTypes=".gif" />
          <s-input-textarea v-model="object.orientacao" ref="orientacao" divClass="col-12 col-md-12 col-xxl-12"
            label="Orientação" />

        </div>
        <div class="row">
          <s-label-required />
        </div>
        <hr />
        <div class="row">
          <div class="col-12 d-flex justify-content-between">
            <div>
              <s-button type="submit" label="Salvar" color="primary" icon="check2" />
              <s-button type="button" label="Salvar e Continuar" color="secondary" icon="check2" v-if="!object.id"
                @click="saveAndKeep" />
            </div>
            <div>
              <s-button type="button" label="Cancelar" color="outline-danger" icon="x-lg" @click="$router.back" />
            </div>
          </div>
        </div>
      </form>
    </div>
    <s-modal-error ref="modalError" modalTitle="Falha ao adicionar o registro !" :modalBody="modalBody" />
    <s-modal-notlogged ref="modalNotLogged" @confirm="logout" />
  </div>
</template>
  
<script>
import { validateForm } from '@/rule/functions'
import { insert, getById, update, insertDual } from '@/crud'

export default {
  name: 'exercicioNew',

  data: () => ({
    object: {},
    valid: false,
    Modal: null,
    modalError: null,
    modalNotLogged: null,
    modalBody: null,
    title: null,
    route: 'exercicio',
    descricaoAparelho: null,

    status: [
      { label: "ATIVO", value: 'true' },
      { label: "INATIVO", value: 'false' }
    ],

    grupoMusculares: [
      { label: "BICEPS", value: 'BICEPS' },
      { label: "DELTOIDE", value: 'DELTOIDE' },
      { label: "DORSAL", value: 'DORSAL' },
      { label: "GEMEOS", value: 'GEMEOS' },
      { label: "GLUTEO", value: 'GLUTEO' },
      { label: "ISQUOTIBIAL", value: 'ISQUOTIBIAL' },
      { label: "OBLIQUOS", value: 'OBLIQUOS' },
      { label: "PEITORAL", value: 'PEITORAL' },
      { label: "QUADRICEPS", value: 'QUADRICEPS' },
      { label: "RETO ABDOMINAL", value: 'RETO_ABDOMINAL' },
      { label: "TRAPEZIO", value: 'TRAPEZIO' },
      { label: "TRICEPS", value: 'TRICEPS' }
    ],
  }),

  methods: {
    async loadItem(id) {
      if (await this.$checkSession()) {
        await getById(this.route, id)
          .then((res) => {
            this.object = res
          })
          .catch((err) => {
            this.$router.push({ name: 'exercicio' })
          })
      }

      else { this.modalMessage.show() }
    },

    async submitForm() {
      if (await validateForm(this.$refs.form)) { this.save() }
    },

    async saveAndKeep() {
      if (await this.$checkSession()) {
        if (await validateForm(this.$refs.form)) {
          this.object.status ? this.object.status = 1 : this.object.status = 0

          const result = await insert(this.route, this.object)

          if (result.status) {
            if (result.status != '204') {
              this.modalBody = result.response.data
              this.modalError.show()
            }

            else {
              this.$store.dispatch('setShowToast', true)
              this.$store.dispatch('setToastMessage', 'Exercício criado com sucesso !')
              this.object = {}
            }
          }

          else {
            this.modalBody = result.response.data
            this.modalError.show()
          }
        }
      }
      else { this.modalNotLogged.show() }
    },

    async save() {
      if (await this.$checkSession()) {
        this.object.status ? this.object.status = 1 : this.object.status = 0

        if (this.object.id) {
          const newObj = {
            id: this.object.id,
            descricao: this.object.descricao,
            status: this.object.status,
          }

          const result = await update(this.route, this.$route.params.id, newObj)

          if (result.status) {
            if (result.status != '204') {
              this.modalBody = result.response.data
              this.modalError.show()
            }

            else {
              this.$store.dispatch('setShowToast', true)
              this.$store.dispatch('setToastMessage', 'Exercício alterado com sucesso !')
              this.$router.back()
            }
          }

          else {
            this.modalBody = result.response.data
            this.modalError.show()
          }
        }

        else {
          const newObj = { ...this.object }
          const aparelho = {
            id: newObj.idAparelho
          }

          delete newObj.idAparelho

          newObj.aparelho = aparelho

          const result = await insertDual(this.route, this.object)

          // const result = await insert(this.route, this.object)

          // if (result.status) {
          //   console.log(result.status)
          //   if (result.status != 204 && result.status != 200) {
          //     this.modalBody = result.response.data
          //     this.modalError.show()
          //   }

          //   else {
          //     this.$store.dispatch('setShowToast', true)
          //     this.$store.dispatch('setToastMessage', 'Exercício criado com sucesso !')
          //     this.$router.back()
          //   }
          // }

          // else {
          //   this.modalBody = result.response.data
          //   this.modalError.show()
          // }
        }
      }

      else { this.modalNotLogged.show() }
    },

    logout() { logout(this) },

    handleSelectedFile(file) {
      this.object.file = file;
      console.log('HANDLE', this.object.file)
    }

  },

  mounted() {
    this.$route.name == 'exercicioUpdate' ? this.title = 'Edição de Exercício' : this.title = 'Cadastro de Exercício'
    this.modalNotLogged = new this.$Modal(this.$refs.modalNotLogged.$refs.modalPattern)
    this.modalError = new this.$Modal(this.$refs.modalError.$refs.modalPattern)
  },

  async created() {
    const id = this.$route.params.id

    if (id) { await this.loadItem(id) }
  },
}
</script>
  
<style></style>
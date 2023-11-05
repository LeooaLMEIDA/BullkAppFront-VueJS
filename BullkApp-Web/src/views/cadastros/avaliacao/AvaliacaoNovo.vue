<template>
  <div class="m-3">
    <s-title :title="title" :breadcrumb="true" />
    <div class="card card-body mx-2">
      <form ref="form" @submit.prevent="submitForm">
        <div class="row">
          <s-input-text v-model="object.descricao" ref="nomeAluno" maxlength="40" divClass="col-md-6" label="Descrição"
            required />
          <s-input-text v-model="object.idUsuario" ref="idAluno" maxlength="40" divClass="col-md-2" label="Aluno"
            placeholder="" required />
          <!-- <s-input-text v-model="nomeAluno" ref="nomeAluno" maxlength="40" divClass="col-md-4" isDisabled
            label="Nome Aluno" placeholder="" /> -->
          <s-input-textarea v-model="object.observacao" ref="descricao" divClass="col-md-12" label="Observação"
            placeholder="" />
          <!-- <s-input-file :selectedFile="object.file" @fileSelected="handleSelectedFile" ref="image" divClass="col-md-12"
            label="Imagem" acceptedTypes=".pdf" /> -->
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
import { insert, getById, update } from '@/crud'

export default {
  name: 'avaliacoesNew',

  data: () => ({
    object: {},
    valid: false,
    Modal: null,
    modalError: null,
    modalNotLogged: null,
    modalBody: null,
    title: null,
    route: 'avaliacao',

    // statusData: [
    //   { label: "Ativo", value: 1 },
    //   { label: "Inativo", value: 0 },
    // ],

    nomeAluno: "",

  }),

  methods: {
    async loadItem(id) {
      if (await this.$checkSession()) {
        await getById(this.route, id)
          .then((res) => {
            this.object = res
          })
          .catch((err) => {
            this.$router.push({ name: 'avaliacao' })
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
          // this.object.status ? this.object.status = 1 : this.object.status = 0

          const result = await insert(this.route, this.object)

          if (result.status) {
            if (result.status != '204') {
              this.modalBody = result.response.data
              this.modalError.show()
            }

            else {
              this.$store.dispatch('setShowToast', true)
              this.$store.dispatch('setToastMessage', 'Avaliação criada com sucesso !')
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
        // this.object.status ? this.object.status = 1 : this.object.status = 0

        if (this.object.id) {
          console.log(this.object)

          // const newObj = {
          //   id: this.object.id,
          //   descricao: this.object.descricao,
          //   status: this.object.status,
          // }

          const result = await update(this.route, this.$route.params.id, this.object)

          if (result.status) {
            if (result.status != '204') {
              this.modalBody = result.response.data
              this.modalError.show()
            }

            else {
              this.$store.dispatch('setShowToast', true)
              this.$store.dispatch('setToastMessage', 'Avaliação alterada com sucesso !')
              this.$router.back()
            }
          }

          else {
            this.modalBody = result.response.data
            this.modalError.show()
          }
        }

        else {
          const result = await insert(this.route, this.object)

          if (result.status) {
            if (result.status != 204 && result.status != 200) {
              this.modalBody = result.response.data
              this.modalError.show()
            }

            else {
              this.$store.dispatch('setShowToast', true)
              this.$store.dispatch('setToastMessage', 'Avaliação criada com sucesso !')
              this.$router.back()
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

    logout() { logout(this) },

    handleSelectedFile(file) {
      this.object.file = file;
    }
  },

  mounted() {
    this.$route.name == 'avaliacaoUpdate' ? this.title = 'Edição de Avaliação' : this.title = 'Cadastro de Avaliação'
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
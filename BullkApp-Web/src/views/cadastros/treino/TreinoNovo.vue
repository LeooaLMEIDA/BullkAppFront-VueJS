<template>
  <div class="m-3">
    <s-title :title="title" :breadcrumb="true" />
    <div class="card card-body mx-2">
      <form ref="form" @submit.prevent="submitForm">
        <div class="row">
          <s-input-text v-model="idExercicio" ref="exercicio" divClass="col-md-2" label="Exercício" placeholder=""
            required />
          <s-input-text v-model="descricaoExercicio" ref="descricaoExercicio" maxlength="40" divClass="col-md-4"
            isDisabled label="Descrição Exercício" placeholder="" />
          <s-input-text v-model="idAluno" ref="aluno" divClass="col-md-2" label="Aluno" placeholder="" required />
          <s-input-text v-model="nomeAluno" ref="nomeAluno" maxlength="40" divClass="col-md-4" isDisabled
            label="Nome Aluno" placeholder="" />
          <s-select v-model="object.status" divClass="col-md-1" label="Treino" :items="treinos" :clearable="false"
            required />
          <s-input-text v-model="object.serie" ref="serie" divClass="col-md-2" label="Série" placeholder="" required />
          <s-input-text v-model="object.repeticoes" ref="repeticoes" divClass="col-md-2" label="Repetições" placeholder=""
            required />
          <s-input-text v-model="object.adas" ref="peso" divClass="col-md-1" label="Peso" placeholder="" required />
          <s-input-text v-model="object.das" ref="intervalo" divClass="col-md-2" label="Intervalo" placeholder=""
            required />
          <s-select v-model="object.status" divClass="col-md-2" label="Status" :items="status" :clearable="false" />
          <s-input-check v-model="object.status" divClass="col-md-2 mt-3" label="Alternativo" />
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
  name: 'treinoNew',

  data: () => ({
    object: {},
    valid: false,
    Modal: null,
    modalError: null,
    modalNotLogged: null,
    modalBody: null,
    title: null,
    route: 'treino',

    idExercicio: null,
    descricaoExercicio: "",

    idAluno:null,
    nomeAluno:"",

    status: [
      { label: "Ativo", value: 'true' },
      { label: "Inativo", value: 'false' },
    ],

    treinos: [
      { label: "A", value: 'A' },
      { label: "B", value: 'B' },
      { label: "C", value: 'C' }
    ]



  }),

  methods: {
    async loadItem(id) {
      if (await this.$checkSession()) {
        await getById(this.route, id)
          .then((res) => {
            this.object = res
            console.log(this.object.exercicio)
          })
          .catch((err) => {
            this.$router.push({ name: 'treino' })
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
              this.$store.dispatch('setToastMessage', 'Treino criado com sucesso !')
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
              this.$store.dispatch('setToastMessage', 'Treino alterado com sucesso !')
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
            console.log(result.status)
            if (result.status != 204 && result.status != 200) {
              this.modalBody = result.response.data
              this.modalError.show()
            }

            else {
              this.$store.dispatch('setShowToast', true)
              this.$store.dispatch('setToastMessage', 'Treino criado com sucesso !')
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

    logout() { logout(this) }
  },

  mounted() {
    this.$route.name == 'treinoUpdate' ? this.title = 'Edição de Treino' : this.title = 'Cadastro de Treino'
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
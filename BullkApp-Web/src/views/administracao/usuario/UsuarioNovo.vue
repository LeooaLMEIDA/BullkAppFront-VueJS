<template>
  <div class="m-3">
    <s-title :title="title" :breadcrumb="true" />
    <div class="card card-body mx-2">
      <form ref="form" @submit.prevent="submitForm">
        <div class="row">
          <s-input-text v-model="object.name" ref="name" divClass="col-12 col-md-4 col-xxl-4" label="Nome"
            placeholder="Nome Completo" required />
          <s-input-email v-model="object.email" ref="email" divClass="col-12 col-md-4 col-xxl-4" label="E-mail"
            placeholder="email@email.com" required />
          <s-input-password v-if="!object.id" v-model="object.password" ref="password" divClass="col-12 col-md-2"
            label="Senha" placeholder="••••••••" required />
          <s-select v-model="object.sexo" divClass="col-md-2" label="Sexo" :items="sexos" :clearable="false" required />
          <s-input-date v-model="object.dtNascimento" divClass="col-md-2" label="Nascimento" required />
          <s-input-text v-model="object.celular" ref="celular" divClass="col-12 col-md-4" label="Celular"
            placeholder="Celular" v-mask="'(##) #####-####'" required />
          <s-select v-model="object.tpUsuario" divClass="col-md-2" label="Tipo" :items="tipo" :clearable="false"
            required />
          <s-select v-model="object.status" divClass="col-md-2" label="Status" :items="status" :clearable="false" />
          <s-input-file v-model="object.img" ref="image" divClass="col-md-12" label="Imagem"
            :acceptedTypes="['.png', '.jpeg']" />

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
  name: 'UserNew',

  data: () => ({
    object: {},
    valid: false,
    Modal: null,
    modalError: null,
    modalNotLogged: null,
    modalBody: null,
    title: null,
    route: 'user',
    headersGroup: ['Grupo', 'Ações'],
    itemsGroup: [
      { 'name': 'Grupo 01' },
      { 'name': 'Grupo 01' },
      { 'name': 'Grupo 01' },
      { 'name': 'Grupo 01' },
      { 'name': 'Grupo 01' },
    ],
    sexos: [
      { label: "Masculino", value: 'MASCULINO' },
      { label: "Feminino", value: 'FEMININO' }
    ],
    tipo: [
      { label: "Admin", value: 'ADMIN' },
      { label: "Aluno", value: 'ALUNO' }
    ],
    status: [
      { label: "Ativo", value: 'true' },
      { label: "Inativo", value: 'false' },
    ],

  }),

  methods: {
    async loadItem(id) {
      if (await this.$checkSession()) {
        await getById(this.route, id)
          .then((res) => {
            this.object = res
            if (this.object.status == 1) {
              this.object.status = true
            }

            else {
              this.object.status = false
            }
          })
          .catch((err) => {
            console.error(err)
            this.$router.push({ name: 'user' })
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
              this.$store.dispatch('setToastMessage', 'Usuário criado com sucesso !')
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
            name: this.object.name,
            email: this.object.email,
            status: this.object.status
          }

          const result = await update(this.route, this.$route.params.id, newObj)

          if (result.status) {
            if (result.status != '204') {
              this.modalBody = result.response.data
              this.modalError.show()
            }

            else {
              this.$store.dispatch('setShowToast', true)
              this.$store.dispatch('setToastMessage', 'Usuário alterado com sucesso !')
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
            if (result.status != '204') {
              this.modalBody = result.response.data
              this.modalError.show()
            }

            else {
              this.$store.dispatch('setShowToast', true)
              this.$store.dispatch('setToastMessage', 'Usuário criado com sucesso !')
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
    this.$route.name == 'userUpdate' ? this.title = 'Edição de Usuário' : this.title = 'Cadastro de Usuário'
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
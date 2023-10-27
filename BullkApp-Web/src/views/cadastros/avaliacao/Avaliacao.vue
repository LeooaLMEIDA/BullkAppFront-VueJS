<template>
  <div class="m-3">
    <div class="row">
      <div class="col-8">
        <s-title title="Avaliação" :breadcrumb="true" />
      </div>
    </div>
    <s-input-filter @index="handleIndex" @filter="filterAll" @clear="loadItems" name="filterWorkMeasurement"
      :filters="filterObject" />
    <div class="card card-body mx-2">
      <div class="row">
        <div class="col-12">
          <s-table v-model="actualPage" :headers="headers" :items="items" :totalPages="pages" v-if="!loader">
            <template v-slot:descricao="{ item }">
              {{ item.descricao }}
            </template>
            <template v-slot:observacao="{ item }">
              {{ item.observacao }}
            </template>
            <template v-slot:aluno="{ item }">
              {{ item.observacao }}
            </template>
            <template v-slot:actions="{ item }">
              <div class="text-center">
                <i class="bi bi-pencil-fill text-secondary px-1" style="cursor: pointer" @click="edit(item.id)"></i>
                <i class="bi bi-trash-fill text-danger px-1" style="cursor: pointer" @click="removeConfirm(item)"></i>
              </div>
            </template>
          </s-table>
        </div>
        <div class="col-12" v-if="!loader">
          <s-button type="button" label="Novo" color="primary" icon="plus-lg"
            @click="this.$router.push({ name: 'avaliacaoNew' })" />
        </div>
      </div>
      <!-- <TheLoader v-if="loader" /> -->
    </div>
    <s-modal-delete ref="modalDelete" @confirm="remove" />
    <s-modal-notlogged ref="modalNotLogged" @confirm="logout" />
  </div>
</template>

<script>
import { logout } from '@/rule/functions.js'
import { get, remove, update, search } from '@/crud.js'

export default {
  name: 'avaliacao',

  data: () => ({
    route: 'avaliacao',
    headers: [
      { title: 'Descrição', field: 'descricao' },
      { title: 'Observação', field: 'observacao' },
      { title: 'Ações', field: 'actions' }
    ],
    items: [],
    object: {},
    selectedItem: null,
    Modal: null,
    choosed: null,
    loader: false,
    modalDelete: null,
    modalNotLogged: null,
    pages: null,
    actualPage: 1,
    limit: 10,

    filterObject: [
      /*{
        label: 'Nome',
        ref: 'usuarName',
        route: 'aparelho',
        subRoute: 'byIdUser',
        param: 'idUser',
        type: 'text',
        signal: '=',
        operator: 'LIKE',
        index: 1
      },
      {
        label: 'Gênero',
        ref: 'bookGender',
        route: 'book',
        subRoute: 'by-gender',
        param: 'gender',
        type: 'text',
        signal: '',
        operator: 'LIKE',
        index: 2
      },
      {
        label: 'Autor',
        ref: 'bookAuthor',
        route: 'book',
        subRoute: 'by-author',
        param: 'author',
        type: 'text',
        signal: '',
        operator: 'LIKE',
        index: 3
      },*/
    ],

    filterOption: 1,
    filterParam: null,
  }),

  methods: {
    async loadItems(page = 1) {
      /*if (await this.$checkSession()) {
        const query = { params: { page: page, limit: this.limit } }
        let raw = []
        if (this.filterParam) {
          this.filterParam.params.page = page
          this.filterParam.params.limit = this.limit
          raw = await search(this.filterParam.route, this.filterParam.params)
        } else {
          raw = await get(this.route, query)
        }
        this.items = raw
          this.pages = Math.ceil(raw.total / this.limit)
      } else {
       this.modalNotLogged.show()
      } */

      let raw = []
      raw = await get(this.$route.name, {})
      this.items = raw

    },

    async edit(id) {
      const route = {
        name: 'avaliacaoUpdate',
        params: { id: id },
      }

      this.$router.push(route)
    },

    async remove() {
      if (await this.$checkSession()) {
        await remove(this.route, this.choosed.id)

        this.$store.dispatch('setShowToast', true)
        this.$store.dispatch('setToastMessage', 'Avaliação excluída com sucesso !')
        this.loadItems()
      } else {
        this.modalNotLogged.show()
      }
    },

    removeConfirm(item) {
      this.choosed = item
      this.modalDelete.show()
    },

    getStatusColor(status) {
      return status == 1 ? "bg-success" : "bg-danger";
    },

    translateStatusText(status) {
      return status == 1 ? "Ativo" : "Inativo";
    },

    async filterAll(event) {
      if (await this.$checkSession()) {
        this.filterParam = event
        this.loadItems(1)
      } else {
        this.modalNotLogged.show()
      }
    },

    handleIndex(event) {
      this.filterOption = event
    },

    changeHeaders() {
      if (this.filterOption == 1) {
        this.headers = [
          { title: 'Nome', field: 'name' },
          { title: 'Gênero', field: 'gender' },
          { title: 'Autor', field: 'author' },
          { title: 'Páginas', field: 'quantityPages' },
          { title: 'Data Aquisição', field: 'dateAcquisition' },
          { title: 'Ações', field: 'actions' },
        ]
      }
    },
    logout() {
      logout(this)
    },
  },

  async mounted() {
    this.modalDelete = new this.$Modal(this.$refs.modalDelete.$refs.modalPattern)
    this.modalNotLogged = new this.$Modal(this.$refs.modalNotLogged.$refs.modalPattern)

    await this.loadItems()
  },

  watch: {
    filterOption() {
      this.filterParam = null
      this.loadItems()
      this.changeHeaders()
    },
    // actualPage() {
    //     this.loadItems(this.actualPage)
    // },
  },
}
</script>

<style></style>
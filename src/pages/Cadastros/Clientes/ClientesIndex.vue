<template>
  <div class="grid">
    <div class="col-12">
      {{ locForm }}
      <div class="card">
        <Toast/>

        <Toolbar class="mb-4">
          <template v-slot:start>
            <div class="my-2">
              <Button label="Novo" icon="pi pi-plus" class="p-button-primary mr-2" @click="adicionarNovo"/>

              <Button label="Excluir" icon="pi pi-trash" class="p-button-danger" @click="confirmDeleteSelected"
                      :disabled="!linhaSelecionada || !linhaSelecionada.length"/>
            </div>
          </template>

          <template v-slot:end>
            <Button label="Visualizar" icon="pi pi-search" class="p-button-secondary mr-2 inline-block" @click="show"/>

            <FileUpload mode="basic" accept="image/*" :maxFileSize="1000000" label="Importar" chooseLabel="Importar"
                        class="mr-2 inline-block"/>

            <Button label="Exportar" icon="pi pi-upload" class="p-button-help inline-block" @click="exportCSV($event)"/>

          </template>

        </Toolbar>

        <DataTable ref="dt" v-if="registros"
                   :value="registros" v-model:selection="linhaSelecionada" dataKey="id" :paginator="true"
                   :rows="10" :filters="filtros"
                   paginatorTemplate="FirstPageLink PrevPageLink PageLinks NextPageLink LastPageLink CurrentPageReport RowsPerPageDropdown"
                   :rowsPerPageOptions="[5,10,25]"
                   currentPageReportTemplate="Monstrando de {first} a {last} de {totalRecords} clientes"
                   responsiveLayout="scroll">

          <template #header>
            <div class="flex flex-column md:flex-row md:justify-content-between md:align-items-center">
              <h5 class="m-0">Gerenciar Clientes</h5>
              <span class="block mt-2 md:mt-0 p-input-icon-left">
                <i class="pi pi-search"/>
                <InputText v-model="filtros['global'].value" placeholder="Codigo..."/>
              </span>

              <span class="block mt-2 md:mt-0 p-input-icon-left">
                <i class="pi pi-search"/>
                <InputText v-model="filtros['global'].value" placeholder="Nome..."/>
              </span>
            </div>
          </template>

          <Column selectionMode="multiple" headerStyle="width: 3rem"></Column>
          <Column field="id" header="Código" :sortable="true" headerStyle="width:20%;">
            <template #body="slotProps">
              <span class="p-column-title">Código</span>
              {{ slotProps.data.id }}
            </template>
          </Column>

          <Column field="nome" header="Nome" :sortable="true" headerStyle="width:40%;">
            <template #body="slotProps">
              <span class="p-column-title">Nome</span>
              {{ slotProps.data.nome }}
            </template>
          </Column>

          <Column field="rating" header="Avaliações" :sortable="true" headerStyle="width:30%; min-width:10rem;">
            <template #body="slotProps">
              <span class="p-column-title">Avaliação</span>
              <Rating :modelValue="slotProps.data.avaliacao" :readonly="true" :cancel="false"/>
            </template>
          </Column>

          <Column field="status" header="Status" :sortable="true" headerStyle="width:30%;">
            <template #body="slotProps">
              <span class="p-column-title">Status</span>
              <span
                  :class="'product-badge status-' + (slotProps.data.inventoryStatus ? slotProps.data.inventoryStatus.toLowerCase() : '')">{{
                  slotProps.data.status
                }}</span>
            </template>
          </Column>
          <Column headerStyle="min-width:10rem;">
            <template #body="slotProps">
              <Button icon="pi pi-pencil" class="p-button-rounded p-button-success mr-2"
                      @click="editProduct(slotProps.data)"/>
              <Button icon="pi pi-trash" class="p-button-rounded p-button-warning mt-2"
                      @click="confirmDeleteProduct(slotProps.data)"/>
            </template>
          </Column>
        </DataTable>

        <Dialog v-model:visible="dialogProduto" :style="{width: '450px'}" header="Cadastro cliente" :modal="true"
                class="p-fluid">
          <img :src="'images/product/' + produto.image" :alt="produto.image" v-if="produto.image" width="150"
               class="mt-0 mx-auto mb-5 block shadow-2"/>
          <div class="field">
            <label for="none">Nome</label>
            <InputText id="nome" v-model="locForm.nome" required="true" autofocus
                       :class="{'p-invalid': submetido && !produto.name}"/>
            <small class="p-invalid" v-if="submetido && !produto.name">O campo nome é requerido.</small>
          </div>

          <div class="field">
            <label for="name">Avaliação</label>
            <InputText id="name" v-model="locForm.avaliacao" required="true" autofocus
                       :class="{'p-invalid': submetido && !produto.name}"/>
            <small class="p-invalid" v-if="submetido && !produto.name">O campo avaliação é requerido.</small>
          </div>

          <div class="field">
            <label for="name">Status</label>
            <InputText id="name" v-model="locForm.status" required="true" autofocus
                       :class="{'p-invalid': submetido && !produto.name}"/>
            <small class="p-invalid" v-if="submetido && !produto.name">O campo status é requerido.</small>
          </div>

          <!--          <div class="field">
                      <label for="inventoryStatus" class="mb-3">Status</label>
                      <Dropdown id="inventoryStatus" v-model="locForm.status" :options="statuses" optionLabel="label"
                                placeholder="Selecione o status">
                        <template #value="slotProps">
                          <div v-if="slotProps.value && slotProps.value.value">
                            <span :class="'product-badge status-' +slotProps.value.value">{{ slotProps.value.label }}</span>
                          </div>
                          <div v-else-if="slotProps.value && !slotProps.value.value">
                            <span :class="'product-badge status-' +slotProps.value.toLowerCase()">{{ slotProps.value }}</span>
                          </div>
                          <span v-else>
                            {{ slotProps.placeholder }}
                          </span>
                        </template>
                      </Dropdown>
                    </div>-->

          <template #footer>
            <Button label="Cancelar" icon="pi pi-times" class="p-button-text" @click="hideDialog"/>
            <Button label="Salvar" icon="pi pi-check" class="p-button-text" @click="store"/>
          </template>
        </Dialog>

        <Dialog v-model:visible="deleteProductDialog" :style="{width: '450px'}" header="Confirm" :modal="true">
          <div class="flex align-items-center justify-content-center">
            <i class="pi pi-exclamation-triangle mr-3" style="font-size: 2rem"/>
            <span v-if="produto">Tem certeza de que deseja excluir <b>{{ produto.name }}</b>?</span>
          </div>
          <template #footer>
            <Button label="No" icon="pi pi-times" class="p-button-text" @click="deleteProductDialog = false"/>
            <Button label="Yes" icon="pi pi-check" class="p-button-text" @click="deleteProduct"/>
          </template>
        </Dialog>

        <Dialog v-model:visible="deleteProductsDialog" :style="{width: '450px'}" header="Confirm" :modal="true">
          <div class="flex align-items-center justify-content-center">
            <i class="pi pi-exclamation-triangle mr-3" style="font-size: 2rem"/>
            <span v-if="produto">Tem certeza de que deseja excluir os produtos selecionados?</span>
          </div>
          <template #footer>
            <Button label="No" icon="pi pi-times" class="p-button-text" @click="deleteProductsDialog = false"/>
            <Button label="Yes" icon="pi pi-check" class="p-button-text" @click="deleteSelectedProducts"/>
          </template>
        </Dialog>

      </div>
    </div>
  </div>
</template>

<script>


import ProductService from "@/service/ProductService";
import {FilterMatchMode} from "primevue/api";
import axios from "axios";

export default {
  name: "ClientesIndex",

  props: {
    registro_id: {
      type: String,
      default: null,
    }
  },


  data() {
    return {
      produtos: null,
      produto: {},
      linhaSelecionada: null,
      filtros: {},
      submetido: false,
      dialogProduto: null,
      deleteProductDialog: false,
      deleteProductsDialog: false,

      nome: {},
      avaliacao: {},
      status: {},

      // loading1: false,
      locForm: {},
      registros: undefined,

      statuses: [
        {label: 'ATIVO', value: 'ativo'},
        {label: 'SUSPENSO', value: 'suspenso'}
      ]
    }
  },
  productService: null,
  created() {
    this.productService = new ProductService();
    this.initFilters();
  },
  async mounted() {
    this.productService.getProducts().then(data => this.produtos = data);
    this.show();
    await this.getData();

    if (this.registro_id) {
      await this.getRegistro(`http://127.0.0.1:8000/api/cadastro/clientes/${this.registro_id}/edit`)
          .then((response) => {
            this.locForm = response.registro;
            console.log(response)
          });
    }
  },

  methods: {
    adicionarNovo() {
      this.produto = {};
      this.submetido = false;
      this.dialogProduto = true;
    },

    hideDialog() {
      this.dialogProduto = false;
      this.submetido = false;
    },

    saveProduct() {
      this.submetido = true;
      if (this.produto.name.trim()) {
        if (this.produto.id) {
          this.produto.inventoryStatus = this.produto.inventoryStatus.value ? this.produto.inventoryStatus.value : this.produto.inventoryStatus;
          this.produtos[this.findIndexById(this.produto.id)] = this.produto;
          this.$toast.add({severity: 'success', summary: 'Successful', detail: 'Product Updated', life: 3000});
        } else {
          this.produto.id = this.createId();
          this.produto.code = this.createId();
          this.produto.image = 'product-placeholder.svg';
          this.produto.inventoryStatus = this.produto.inventoryStatus ? this.produto.inventoryStatus.value : 'INSTOCK';
          this.produtos.push(this.produto);
          this.$toast.add({severity: 'success', summary: 'Successful', detail: 'Product Created', life: 3000});
        }
        this.dialogProduto = false;
        this.produto = {};
      }
    },

    store() {
      axios.post('http://127.0.0.1:8000/api/cadastro/clientes', this.locForm)
          .then(() => {
            // console.log(response);

            this.$toast.add({severity: 'success', summary: "Cliente cadastrado com sucesso!", life: 2000});
          })
          .catch((error) => {
            console.log(this.errors = error);
          })
    },

    getData: async function () {
      await axios.get('http://127.0.0.1:8000/api/cadastro/clientes/get_data')

          .then((response) => {
            this.nome = response.nome;
            this.avaliacao = response.avaliacao;
            this.status = response.status;

            // console.log(response)
          });
    },

    getRegistro: async function () {
      const dados = await axios.get(`http://127.0.0.1:8000/api/cadastro/clientes/${this.registro_id}/edit`);
      this.locForm = dados.data.registro;
    },

    async show() {
      this.loading1 = true;

      //TODO, VER COM IRLAN, PQ DO PARAMS.
      axios.get('http://127.0.0.1:8000/api/cadastro/clientes/show', {
        params: this.locForm
      })
          .then((response) => {
            this.registros = response.data.registros.data;
            this.loading1 = false;
            console.log(response.data.registros.data);
          });

    },

    editProduct(produto) {
      this.produto = {...produto};
      this.dialogProduto = true;
    },
    confirmDeleteProduct(produto) {
      this.produto = produto;
      this.deleteProductDialog = true;
    },
    deleteProduct() {
      this.produtos = this.produtos.filter(val => val.id !== this.produto.id);
      this.deleteProductDialog = false;
      this.produto = {};
      this.$toast.add({severity: 'success', summary: 'Successful', detail: 'Product Deleted', life: 3000});
    },
    findIndexById(id) {
      let index = -1;
      for (let i = 0; i < this.produtos.length; i++) {
        if (this.produtos[i].id === id) {
          index = i;
          break;
        }
      }
      return index;
    },
    createId() {
      let id = '';
      var chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
      for (var i = 0; i < 5; i++) {
        id += chars.charAt(Math.floor(Math.random() * chars.length));
      }
      return id;
    },
    exportCSV() {
      this.$refs.dt.exportCSV();
    },
    confirmDeleteSelected() {
      this.deleteProductsDialog = true;
    },
    deleteSelectedProducts() {
      this.produtos = this.produtos.filter(val => !this.linhaSelecionada.includes(val));
      this.deleteProductsDialog = false;
      this.linhaSelecionada = null;
      this.$toast.add({severity: 'success', summary: 'Successful', detail: 'Products Deleted', life: 3000});
    },
    initFilters() {
      this.filtros = {
        'global': {value: null, matchMode: FilterMatchMode.CONTAINS},
      }
    }
  },
}
</script>

<style scoped lang="scss">
@import '@/assets/demo/badges.scss';
</style>
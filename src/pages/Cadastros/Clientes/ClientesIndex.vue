<template>
  <div class="grid">
    <div class="col-12">

      <div class="card">
        <Toast/>

        <Toolbar class="mb-4">
          <template v-slot:start>
            <div class="my-2">
              <Button label="New" icon="pi pi-plus" class="p-button-primary mr-2" @click="openNew"/>
              <Button label="Delete" icon="pi pi-trash" class="p-button-danger" @click="confirmDeleteSelected"
                      :disabled="!linhaSelecionada || !linhaSelecionada.length"/>
            </div>
          </template>

          <template v-slot:end>
            <FileUpload mode="basic" accept="image/*" :maxFileSize="1000000" label="Import" chooseLabel="Import"
                        class="mr-2 inline-block"/>
            <Button label="Export" icon="pi pi-upload" class="p-button-help" @click="exportCSV($event)"/>
          </template>

        </Toolbar>

        <DataTable ref="dt" :value="produtos" v-model:selection="linhaSelecionada" dataKey="id" :paginator="true"
                   :rows="10" :filters="filtros"
                   paginatorTemplate="FirstPageLink PrevPageLink PageLinks NextPageLink LastPageLink CurrentPageReport RowsPerPageDropdown"
                   :rowsPerPageOptions="[5,10,25]"
                   currentPageReportTemplate="Showing {first} to {last} of {totalRecords} produtos"
                   responsiveLayout="scroll">

          <template #header>
            <div class="flex flex-column md:flex-row md:justify-content-between md:align-items-center">
              <h5 class="m-0">Gerenciar Clientes</h5>
              <span class="block mt-2 md:mt-0 p-input-icon-left">
                <i class="pi pi-search"/>
                <InputText v-model="filtros['global'].value" placeholder="Search..."/>
              </span>
            </div>
          </template>

          <Column selectionMode="multiple" headerStyle="width: 3rem"></Column>
          <Column field="code" header="Código" :sortable="true" headerStyle="width:20%;">
            <template #body="slotProps">
              <span class="p-column-title">Código</span>
              {{ slotProps.data.code }}
            </template>
          </Column>

          <Column field="name" header="Nome" :sortable="true" headerStyle="width:40%;">
            <template #body="slotProps">
              <span class="p-column-title">Nome</span>
              {{ slotProps.data.name }}
            </template>
          </Column>

          <Column field="rating" header="Avaliações" :sortable="true" headerStyle="width:30%; min-width:10rem;">
            <template #body="slotProps">
              <span class="p-column-title">Avaliação</span>
              <Rating :modelValue="slotProps.data.rating" :readonly="true" :cancel="false"/>
            </template>
          </Column>

          <Column field="inventoryStatus" header="Status" :sortable="true" headerStyle="width:30%;">
            <template #body="slotProps">
              <span class="p-column-title">Status</span>
              <span
                  :class="'product-badge status-' + (slotProps.data.inventoryStatus ? slotProps.data.inventoryStatus.toLowerCase() : '')">{{
                  slotProps.data.inventoryStatus
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

        <Dialog v-model:visible="dialogProduto" :style="{width: '450px'}" header="Product Details" :modal="true"
                class="p-fluid">
          <img :src="'images/product/' + produto.image" :alt="produto.image" v-if="produto.image" width="150"
               class="mt-0 mx-auto mb-5 block shadow-2"/>
          <div class="field">
            <label for="name">Name</label>
            <InputText id="name" v-model.trim="produto.name" required="true" autofocus
                       :class="{'p-invalid': submetido && !produto.name}"/>
            <small class="p-invalid" v-if="submetido && !produto.name">Name is required.</small>
          </div>
          <div class="field">
            <label for="description">Description</label>
            <Textarea id="description" v-model="produto.description" required="true" rows="3" cols="20"/>
          </div>

          <div class="field">
            <label for="inventoryStatus" class="mb-3">Inventory Status</label>
            <Dropdown id="inventoryStatus" v-model="produto.inventoryStatus" :options="statuses" optionLabel="label"
                      placeholder="Select a Status">
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
          </div>

          <div class="field">
            <label class="mb-3">Category</label>
            <div class="formgrid grid">
              <div class="field-radiobutton col-6">
                <RadioButton id="category1" name="category" value="Accessories" v-model="produto.category"/>
                <label for="category1">Accessories</label>
              </div>
              <div class="field-radiobutton col-6">
                <RadioButton id="category2" name="category" value="Clothing" v-model="produto.category"/>
                <label for="category2">Clothing</label>
              </div>
              <div class="field-radiobutton col-6">
                <RadioButton id="category3" name="category" value="Electronics" v-model="produto.category"/>
                <label for="category3">Electronics</label>
              </div>
              <div class="field-radiobutton col-6">
                <RadioButton id="category4" name="category" value="Fitness" v-model="produto.category"/>
                <label for="category4">Fitness</label>
              </div>
            </div>
          </div>

          <template #footer>
            <Button label="Cancel" icon="pi pi-times" class="p-button-text" @click="hideDialog"/>
            <Button label="Save" icon="pi pi-check" class="p-button-text" @click="saveProduct"/>
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

export default {
  name: "ClientesIndex",

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

      statuses: [
        {label: 'ATIVO', value: 'instock'},
        {label: 'LOWSTOCK', value: 'lowstock'},
        {label: 'SUSPENSO', value: 'outofstock'}
      ]
    }
  },
  productService: null,
  created() {
    this.productService = new ProductService();
    this.initFilters();
  },
  mounted() {
    this.productService.getProducts().then(data => this.produtos = data);
  },

  methods: {
    formatCurrency(value) {
      if (value)
        return value.toLocaleString('en-US', {style: 'currency', currency: 'USD'});
      return;
    },

    openNew() {
      this.produto = {};
      this.submetido = false;
      this.dialogProduto = false;
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
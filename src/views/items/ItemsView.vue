<template>
  <div class="container mt-3">
    <h4 class="mb-3">Preencha os campos para cadastrar/editar um item</h4>
    <VeeForm @submit="onValidSubmit" v-slot="{ errors }" @invalid-submit="onInvalidSubmit"
      class="formCadastro animate__animated animate__fadeIn">
      <div class="row mb-1">
        <h3>Dados do item</h3>
        <hr />
        <div class="col-sm-12 col-md-6 col-lg-3">
          <label class="form-label">Cód. de Patrimônio <span>*</span></label>
          <input type="text" name="codPatrimonio" class="form-control" placeholder="Cód. automatico"
            v-model="form.codPatrimonio" disabled />
        </div>
        <div class="col-sm-12 col-md-6 col-lg-4">
          <label class="form-label">Título do item <span>*</span></label>
          <Veefield type="text" name="title" class="form-control" placeholder="Titulo do item" v-model.trim="form.title"
            required v-focus :class="{ 'is-invalid': errors.title }" :rules="required" maxlength="30" />
          <div class="invalid-feedback animate__animated animate__shakeX">{{ errors.title }}</div>
        </div>
        <div class="col-sm-12 col-md-6 col-lg-4">
          <label class="form-label">Categoria <span>*</span></label>
          <Veefield as="select" name="category" class="form-select" v-model="form.category" required
            :class="{ 'is-invalid': errors.category }" :rules="required">
            <option value="" disabled>Escolha a categoria</option>
            <option value="Computador">Computador</option>
            <option value="Periférico">Periférico</option>
            <option value="Monitor">Cadeira</option>
            <option value="Monitor">Outros</option>
          </Veefield>
          <div class="invalid-feedback animate__animated animate__shakeX">{{ errors.category }}</div>
        </div>
      </div>

      <h4 class="mt-3">Dados Complementares</h4>
      <hr>
      <div class="row mb-1">
        <div class=" col-sm-6 col-md-4 col-lg-3 ">
          <label class="form-label">Valor <span>*</span></label>
          <Veefield type="number" name="value" class="form-control" placeholder="Valor do item"
            v-model.trim="form.value" required :class="{ 'is-invalid': errors.value }" :rules="validateNumber" />
          <div class="invalid-feedback animate__animated animate__shakeX">{{ errors.value }}</div>
        </div>

        <div class="col-sm-12 col-md-4">
          <label class="form-label">Marca <span>*</span></label>
          <Veefield type="text" name="brand" class="form-control" placeholder="Marca do item" v-model.trim="form.brand"
            required :class="{ 'is-invalid': errors.brand }" :rules="required" maxlength="30" />
          <div class="invalid-feedback animate__animated animate__shakeX">{{ errors.brand }}</div>
        </div>
        <div class="col-sm-12 col-md-4 ">
          <label class="form-label">Modelo <span>*</span></label>
          <Veefield type="text" name="model" class="form-control" placeholder="Modelo do item" v-model.trim="form.model"
            required :class="{ 'is-invalid': errors.model }" :rules="required" maxlength="30" />
          <div class="invalid-feedback animate__animated animate__shakeX">{{ errors.model }}</div>
        </div>
      </div>

      <div class="row mt-3">
        <div class="col-sm-12 col-md-6">
          <label class="form-label">Url (imagem do produto) <span>*</span></label>
          <Veefield type="url" name="url" class="form-control" placeholder="caminho url da imagem" v-model="form.url"
            required :class="{ 'is-invalid': errors.url }" @focusout="imageFromUrl" :rules="required" />
          <div class="invalid-feedback animate__animated animate__shakeX">{{ errors.url }}</div>
          <img :src="url" name="imgUrl" class="img-fluid text-center" width="120">
        </div>
        <div class="col-sm-12 col-md-6">
          <label class="form-label">Descrição do item<span>*</span></label>
          <textarea name="description" rows="3" v-model="form.description" required
            placeholder="Digite as especificações do item" class="form-control" maxlength="180"></textarea>
        </div>
      </div>

      <div class="text-end">
        <button :type="infoById ? 'button' : 'reset'" @click="infoById ? cancelEdit() : ''"
          class="btn btn-secondary me-2 mt-2" v-text="infoById ? 'Cancelar' : 'Limpar'"></button>
        <button type="submit" class="mt-2 btn" :class="infoById ? 'btn-primary' : 'btn-success'"
          v-text="btnForm"></button>
      </div>
    </VeeForm>
  </div>
</template>

<script setup>
import { ref, computed } from "vue";
import { useStore } from "vuex";
import { useToast } from "vue-toastification";
import ToastNotification from "./components/ToastNotification.vue";
import { useRoute, useRouter } from "vue-router";
import { useLoading } from "vue-loading-overlay";
import { Form as VeeForm, Field as Veefield } from "vee-validate";
import { required, validateNumber } from "../../validators/validators";
import moment from "moment";

/* 
variaveis funcionais gerais 
*/
const store = useStore();
const route = useRoute();
const $loading = useLoading();
const toast = useToast();
const router = useRouter();
const content = {
  component: ToastNotification,
  listeners: {
    listItems: () => {
      toast.clear();
      router.push({ name: "listItems" });
    },
  },
};
//obtém dados do params da rota - define também a origem da edição (dashboard ou lista de itens)
const id = route.params.itemId ? Number(route.params.itemId.split('-')[0]) : null;
const origin = id ? route.params.itemId.split('-')[1] : null;

//atualização dos dados do item e seta página atual 
store.commit("itemsModule/UPDATE_ITEMS_LOCAL_STORAGE");
store.commit('configModule/SET_PAGE_NAME', 'Criação e edição de itens');

// variavel para verificar se o usuario esta editando ou criando um novo colaborador
const infoById = computed(() => {
  if (id) {
    return store.state.itemsModule.items.find(
      (item) => item.codPatrimonio === id
    );
  }
  return false;
});

// variável para retornar a contagem de itens cadastrados para novo codPatrimonio
const getCountItems = computed(() => {
  if (store.state.itemsModule.items.length > 0) {
    return store.state.itemsModule.items.length + 1;
  } else {
    return 1;
  }
});

// botão de submit do formulário
const btnForm = ref(infoById.value ? "Editar" : "Cadastrar");

// variaveis do formulários - reativas (data)
const form = ref({
  codPatrimonio: infoById.value ? infoById.value.codPatrimonio : null,
  title: infoById.value ? infoById.value.title : null,
  description: infoById.value ? infoById.value.description : null,
  category: infoById.value ? infoById.value.category : null,
  value: infoById.value ? infoById.value.value : null,
  url: infoById.value ? infoById.value.url : null,
  brand: infoById.value ? infoById.value.brand : null,
  model: infoById.value ? infoById.value.model : null,
  collaborator: infoById.value ? infoById.value.collaborator : null,
  createdAt: infoById.value ? infoById.value.createdAt : moment().format("llll"),
  updatedAt: infoById.value ? moment().format("llll") : "",
  loanAt: infoById.value ? infoById.value.loanAt : null,
});
const newForm = ref({})
const url = ref(null);

function imageFromUrl() {
  if (form.value.url && form.value.url !== newForm.value.url) {
    url.value = form.value.url;
  }
}

//função de exibição da imagem ao montar tela
imageFromUrl();

/* 
Funções para submit do formulário
*/
//função executada quando o formulário for submetido com sucesso
function onValidSubmit(values, actions) {
  newForm.value = { ...form.value }
  if (infoById.value) {
    editItem(actions);
    actions.resetForm();
  } else {
    newItem(actions);
    actions.resetForm();
  }
}

//função executada quando houver erros no formulário submetido
function onInvalidSubmit({ errors }) {
  for (let field in errors) {
    toast.error(errors[field], { timeout: 1500 });
  }
}

// função para registrar novo item
function newItem(actions) {
  const loader = $loading.show();
  setTimeout(() => {
    newForm.value.codPatrimonio = getCountItems.value;
    newForm.value.updatedAt = moment().format("llll");
    store.dispatch("itemsModule/registerItem", newForm.value);
    clearForm()
    loader.hide();
    toast(content, {
      position: "top-right",
      closeOnClick: false,
      pauseOnFocusLoss: false,
      pauseOnHover: false,
      draggable: false,
      draggablePercent: 0.6,
      showCloseButtonOnHover: true,
      closeButton: "button",
      icon: "fas fa-rocket",
      rtl: false,
    });
  }, 1000);
}

// Função para editar item
function editItem(actions) {
  const loader = $loading.show();
  setTimeout(() => {
    newForm.value.updatedAt = moment().format("llll");
    store.dispatch("itemsModule/editItem", newForm.value);
    loader.hide();
    toast.success("Item editado com sucesso!");
    if (origin === 'list') {
      router.push({ name: "listItems" });
    } else if (origin === 'dashboard') {
      router.push({ name: "dashboard" });
    }
  }, 2000);
}

// Função para limpar alguns dos campos do formulário não abrangidos pelo veevalidate
function clearForm() {
  form.value.codPatrimonio = '';
  form.value.description = '';
  url.value = null;
}

// Função para cancelar a edição
function cancelEdit() {
  toast.warning("Edição cancelada!", { timeout: 1000 });
  if (origin === 'list') {
    router.push({ name: "listItems" });
  } else if (origin === 'dashboard') {
    router.push({ name: "dashboard" });
  }
}
</script>

<style lang="scss" scoped>
.formCadastro {
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 5px;
  background-color: #fff;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);

  .form-label {
    font-size: 1.1rem;
    font-weight: bold;
    color: var(--color-dark);
  }

  span {
    color: var(--color-danger);
    font-size: 1rem;
    font-weight: bold;
  }
}
</style>

<template>
  <NavBar />
  <br>
  
  <div class="container">
    <div class="text has-text-black has-text-weight-bold has-text-centered is-size-4">
      Cadastro de Produtos
    </div>
    <br>
    <ProdutoForm @produto-cadastrado="atualizarListaProdutos" />
  </div>
  <br>
  <div class="container">
    <div class="columns">
      <div class="column is-2"></div>
      <div class="column">
        <ProdutoList :key="componentKey" />
      </div>
      <div class="column is-1"></div>
    </div>
  </div>
  <br>
  <FooterProduto class="rodape is-hidden-mobile" />
</template>

<script>
import NavBar from './components/NavBar.vue'
import ProdutoForm from './components/ProdutoForm.vue';
import ProdutoList from './components/ProdutoList.vue'
import FooterProduto from './components/FooterProduto.vue'
export default {
  name: 'App',
  components: {
    NavBar,
    ProdutoForm,
    ProdutoList,
    FooterProduto,
  },
  data() {
    return {
      componentKey: 0,
    };
  },
  methods: {
    atualizarListaProdutos() {
      this.componentKey += 1
    },
  },
}
</script>

<style>
#app {
  margin-top: 0px;
  min-height: 100vh;
  background-color: rgb(246, 251, 255);
}

.rodape {
  position: absolute;
  bottom: 0;
  padding: 1rem;
  width: 100%;
}
</style>

<template >
  <table class="table is-striped is-narrow is-hoverable is-fullwidth">
    <thead>
      <tr>
        <th>Código</th>
        <th>Nome</th>
        <th>Preço</th>
        <th>Descrição</th>
        <th></th>
        <th></th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="produto in produtos" :key="produto.codigo">
        <th>{{ produto.codigo }}</th>
        <td>{{ produto.nome }}</td>
        <td>{{ converterParaReais(produto.preco) }}</td>
        <td>{{ produto.descricao }}</td>
        <td><a @click="produtoEdit = produto, showModalEdit = !showModalEdit">edit</a></td>
        <td><button class="delete is-medium has-background-danger-dark"
            @click="ativarModal(), codigoDelete = produto.codigo"></button>
        </td>

      </tr>
    </tbody>
  </table>

  <div class="modal" :class="{ 'is-active': showModal }">
    <div class="modal-background"></div>
    <div class="modal-card">
      <header class="modal-card-head">
        <p class="modal-card-title has-text-weight-bold has-text-black">Deseja Remover?</p>
        <button class="delete" aria-label="close" @click="ativarModal(), codigoDelete = undefined"></button>
      </header>

      <footer class="modal-card-foot">
        <button class="button is-success has-text-weight-bold"
          @click="removerProduto(codigoDelete), ativarModal()">Sim</button>
        <button class="button is-danger has-text-weight-bold"
          @click="ativarModal(), codigoDelete = undefined">Não</button>
      </footer>
    </div>
  </div>

  <!-- Modal de Edição do Produto -->
  <div class="modal" :class="{ 'is-active': showModalEdit }">
    <div class="modal-background"></div>
    <div class="modal-card">
      <header class="modal-card-head">
        <p class="modal-card-title has-text-black has-text-weight-bold">Alterar Produto</p>
        <button class="delete" aria-label="close" @click="fecharModalEdit()"></button>
      </header>
      <section class="modal-card-body">
        <!-- Content ... -->
        <div class="columns">
          <div class="column is-2">
            <div class="field codigo">
              <label class="label">Código:</label>
              <div class="control">
                <input class="input" type="text" v-model="produtoEdit.codigo" disabled>
              </div>
            </div>
          </div>
          <div class="column">
            <div class="field">
              <label class="label">Nome:</label>
              <div class="control">
                <input class="input" type="text" placeholder="Nome do Produto" v-model="produtoEdit.nome">
              </div>
              <p class="help is-danger">{{ erros.nome }}</p>
            </div>
          </div>
        </div>
        <div class="columns">
          <div class="column is-4">
            <div class="field">
              <label class="label">Preço:</label>
              <div class="control">
                <input class="input" type="text" placeholder="R$" value="preço" v-model.lazy="produtoEdit.preco"
                  v-money="money">
              </div>
              <p class="help is-danger">{{ erros.preco }}</p>
            </div>
          </div>
          <div class="column">
            <div class="field">
              <label class="label">Descrição:</label>
              <div class="control">
                <textarea class="textarea has-fixed-size" placeholder="Descrição do Produto" rows="2"
                  v-model="produtoEdit.descricao"></textarea>
              </div>
              <p class="help is-danger">{{ erros.descricao }}</p>
            </div>
          </div>
        </div>
      </section>
      <footer class="modal-card-foot">
        <button class="button is-success has-text-weight-bold" @click="editarProduto()">Salvar</button>
        <button class="button is-danger has-text-weight-bold" @click="fecharModalEdit()">Desistir</button>
      </footer>
    </div>
  </div>
</template>
<script>
import axios from 'axios'
import { VMoney } from "v-money";
export default {
  directives: { money: VMoney },
  created() {
    axios.get(this.url + "/produtos",)
      .then(res => {
        this.produtos = res.data
      })
      .catch(err => {
        console.error(err);
      })
  },

  data() {
    return {
      url: process.env.VUE_APP_IP,
      produtos: [],
      showModal: false,
      showModalEdit: false,
      codigoDelete: undefined,
      produtoEdit: {
        codigo: undefined,
        nome: undefined,
        preco: undefined,
        descricao: undefined,
      },
      money: {
        decimal: ",",
        thousands: ".",
        prefix: "R$ ",
        suffix: "",
        precision: 2,
        masked: false,
      },
      erros: {
        codigo: undefined,
        nome: undefined,
        preco: undefined,
        descricao: undefined
      },
    }
  },

  methods: {
    removerProduto(codigo) {
      axios.delete(this.url + '/produto/' + codigo)
        .then(() => {
          axios.get(this.url + "/produtos",)
            .then(res => {
              this.produtos = res.data
            })
            .catch(err => {
              console.error(err);
            })
        })
        .catch(err => {
          console.error(err);
        })
    },
    ativarModal() {
      this.showModal = !this.showModal
    },
    converterParaReais(valor) {
      // Verifica se o valor é um número

      if (typeof valor !== 'number') {
        if (valor.indexOf('R$') >= 0) {
          return valor
        }
        valor = parseFloat(valor);
      }

      // Converte o número para o formato de moeda brasileira (Real)
      const valorFormatado = valor.toLocaleString('pt-BR', {
        style: 'currency',
        currency: 'BRL'
      });

      return valorFormatado;
    },
    editarProduto() {

      this.produtoEdit.preco = this.converterParaDecimal(this.produtoEdit.preco)

      if (this.formValido(this.produtoEdit)) {
        this.erros = {
          codigo: undefined,
          nome: undefined,
          preco: undefined,
          descricao: undefined
        }

        axios.put(this.url + '/produto', this.produtoEdit)
          .then(async () => {
            await this.buscarProdutosCadastrados()
            await this.fecharModalEdit()
          })
          .catch(async err => {
            console.error(err);
            this.erros = err.response.data
          })
      }

    },
    converterParaDecimal(valorEmReais) {
      // Remove o "R$" e substitui pontos por vazio, e vírgulas por pontos
      const valorSemSimbolo = valorEmReais.replace("R$", "").replace(/\./g, "").replace(",", ".");

      // Converte a string para um número decimal
      const valorDecimal = parseFloat(valorSemSimbolo);

      return valorDecimal;
    },
    formValido(produto) {
      var erros = {}

      if (!this.codigoValido(produto.codigo)) {
        erros.codigo = "O código é composto de até 4 números!"
      }

      if (!this.nomeValido(produto.nome)) {
        erros.nome = "O nome é composto por no mínimo 5 caracteres!"
      }

      if (!this.precoValido(produto.preco)) {
        erros.preco = "O valor do produto deve ser maior que 0!"
      }

      if (!this.descricaoValida(produto.descricao)) {
        erros.descricao = 'A descrição é composta por no mínimo 5 caracteres!'
      }

      if (Object.keys(erros).length === 0) {
        return true
      } else {
        this.erros = erros
        return false
      }
    },

    codigoValido(codigo) {
      if (codigo === undefined) {
        return false
      }

      // Verifica se a string tem exatamente 4 caracteres
      if (codigo.length !== 4) {
        return false;
      }

      // Verifica se todos os caracteres são números
      for (let i = 0; i < codigo.length; i++) {
        if (isNaN(parseInt(codigo[i]))) {
          return false;
        }
      }

      // Se passou pelas verificações acima, a string é válida
      return true;
    },

    nomeValido(nome) {
      if (nome === undefined) {
        return false
      }
      // Verifica se o nome não está vazio e tem mais de 5 caracteres
      if (nome && nome.trim().length > 5) {
        return true; // Retorna true se o nome for válido
      } else {
        return false; // Retorna false se o nome não for válido
      }
    },

    precoValido(preco) {
      if (preco === undefined) {
        return false
      }
      // Verifica se o preço é um número e se é maior que zero
      if (!isNaN(preco) && parseFloat(preco) > 0) {
        return true; // Retorna true se o preço for válido
      } else {
        return false; // Retorna false se o preço não for válido
      }
    },

    descricaoValida(descricao) {
      if (descricao === undefined) {
        return false
      }
      // Verifica se o nome não está vazio e tem mais de 5 caracteres
      if (descricao && descricao.trim().length > 5) {
        return true; // Retorna true se o nome for válido
      } else {
        return false; // Retorna false se o nome não for válido
      }
    },

    async buscarProdutosCadastrados() {
      await axios.get(this.url + "/produtos",)
        .then(res => {
          this.produtos = res.data
        })
        .catch(err => {
          console.error(err);
        })
    },

    async fecharModalEdit() {
      this.showModalEdit = false
    },

  },
}
</script>
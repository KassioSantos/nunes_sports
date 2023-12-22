<template>
    <div>
        <div class="columns">
            <div class="column is-1">
                <div class="field codigo">
                    <label class="label">Código:</label>
                    <div class="control">
                        <input class="input is-black" type="text" placeholder="####" maxlength="4" v-model="produtoForm.codigo"
                            v-maska data-maska="####">
                    </div>
                    <p class="help is-danger">{{ erros.codigo }}</p>
                </div>
            </div>

            <div class="column is-3">
                <div class="field">
                    <label class="label">Nome:</label>
                    <div class="control">
                        <input class="input is-black" type="text" placeholder="Nome do Produto" v-model="produtoForm.nome">
                    </div>
                    <p class="help is-danger">{{ erros.nome }}</p>
                </div>
            </div>

            <div class="column is-2">
                <div class="field">
                    <label class="label">Preço:</label>
                    <div class="control">
                        <input class="input is-black" type="text" placeholder="R$" value="preço" v-model.lazy="produtoForm.preco"
                            v-money="money">
                    </div>
                    <p class="help is-danger">{{ erros.preco }}</p>
                </div>
            </div>

            <div class="column is-5">
                <div class="field">
                    <label class="label">Descrição:</label>
                    <div class="control">
                        <textarea class="textarea is-black has-fixed-size" placeholder="Descrição do Produto" rows="1"
                            v-model="produtoForm.descricao"></textarea>
                    </div>
                    <p class="help is-danger">{{ erros.descricao }}</p>
                </div>
            </div>

            <div class="column">
                <div class="field">
                    <label class="label">&nbsp;</label>
                    <div class="control">
                        <button class="button is-black has-text-weight-bold" v-on:click="cadastrarProduto">Cadastrar</button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>
<script>
import axios from 'axios'
import { vMaska } from "maska"
import { VMoney } from "v-money";
import { mask } from "vue-the-mask"
export default {
    directives: { maska: vMaska, mask, money: VMoney },
    components: {
    },
    data() {
        return {
            url: process.env.VUE_APP_IP,
            produtoForm: {
                codigo: undefined,
                nome: undefined,
                preco: undefined,
                descricao: undefined
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
        };
    },
    methods: {
        cadastrarProduto() {
            this.produtoForm.preco = this.converterParaDecimal(this.produtoForm.preco)
            if (this.formValido(this.produtoForm)) {
                this.erros = {
                    codigo: undefined,
                    nome: undefined,
                    preco: undefined,
                    descricao: undefined
                }
                var res = axios.post(this.url + "/produto", this.produtoForm)
                    .then(() => {
                        return true
                    })
                    .catch(err => {
                        this.erros = err.response.data
                    })

                if (res) {
                    this.produtoForm = {
                        codigo: undefined,
                        nome: undefined,
                        preco: undefined,
                        descricao: undefined
                    }

                    setTimeout(() => this.$emit('produto-cadastrado'), 400);

                }
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

    },
}
</script>

<style scoped></style>
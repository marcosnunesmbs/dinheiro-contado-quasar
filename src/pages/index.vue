<template>
  <q-page>
    <div class="saldoBox bg-indigo-8 full-width">
      <div class="container">
        <div class="row justify-center relative-position saldo-principal" v-ripple>
          <div>
            <h2>{{formatPrice(total)}}</h2>
          </div>
        </div>

        <div class="row justify-center">
          <div class="col-5 ">
            <hr class="shadow-7">
            <div class="row justify-center valores relative-position" v-ripple @click="mudarLimiteModal()">
              <h4>Limite: {{formatPrice(limite)}}</h4>
            </div>
          </div>
          <div class="col-1">
            <div class="row justify-center">
              {{totalDeItens}}
            </div>
          </div>
          <div class="col-5">
            <hr class="shadow-7">
            <div class="row justify-center valores">
               <h4 v-bind:class="{ 'bg-negative': resta < 0 }">Resta: {{formatPrice(resta)}}</h4>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <div class="container shadow-up-4 itensList">
      <div class="flex flex-center text-negative" v-show="produtos.length === 0">
        <h4>Ainda não há itens cadastrados =(</h4>
      </div>
      <div class="row shadow-4 item-card-content" v-for="item in itens" :key="item.index">
        <div class="row full-width justify-between">
          <div class="col-4 self-center">
            <div>
              <span class="produto">{{item.produto}}</span>
              <sup>
                <div class="">
                  {{formatPrice(item.valor)}} x {{item.quantidade}}
                </div>
              </sup>
            </div>
          </div>
          <div class="self-center">
            <div class="valorTotal">
              {{formatPrice(item.quantidade * item.valor)}}
            </div>
          </div>
          <div>
            <q-btn icon="edit" round color="indigo" flat @click.prevent="editar(item)"></q-btn>
            <q-btn icon="delete" round color="negative" flat  @click.prevent="confirmarRemocao(item)"></q-btn>
          </div>
        </div>
      </div>
    </div>
    <q-fab
      class="fixed"
      style="right: 18px; bottom: 18px"
      color="primary"
      icon="keyboard_arrow_up"
      direction="up"
      push
    >
      <q-fab-action
        color="primary"
        icon="add"
        @click="openAddModal()"
      />
      <q-fab-action
        color="negative"
        icon="delete_sweep"
        @click="confirmarRemocaoCompleta"
        v-if="produtos.length > 0"
      />
    </q-fab>

    <q-modal v-model="addModal" minimized>
      <q-toolbar color="indigo-10">
        <q-toolbar-title>
          Adicionar Item
        </q-toolbar-title>
          </q-toolbar>
          <div class="container">
            <form @submit.prevent="adicionarItem()">
              <q-input ref="addItem" :before="[{icon: 'grade'}]" v-model="form.item" float-label="Nome do Item"/>
            <br/>
              <q-input ref="addQnt" type="number" :before="[{icon: 'list'}]" v-model="form.qnt" float-label="Quantidade"/>
            <br/>
              <q-input ref="addValor" type="number" :before="[{icon: 'attach money'}]" prefix="R$ " v-model="form.valor" float-label="Valor"/>
            <br/>
            <q-btn type="submit" label="Lançar Item" color="indigo" icon="done" class="full-width" @click.prevent="adicionarItem()"/>
          </form>
      </div>
    </q-modal>

    <q-modal v-model="editModal" minimized>
      <q-toolbar color="indigo-10">
        <q-toolbar-title>
          Alterar item
        </q-toolbar-title>
      </q-toolbar>
      <div class="container">
        <form @submit.prevent="adicionarItem()">
          <q-input ref="altItem" :before="[{icon: 'grade'}]" v-model="form.item" float-label="Nome do Item"/>
          <br/>
          <q-input ref="altQnt" type="number" :before="[{icon: 'list'}]" v-model="form.qnt" float-label="Quantidade"/>
          <br/>
          <q-input ref="altValor" type="number" :before="[{icon: 'attach money'}]" prefix="R$ " v-model="form.valor" float-label="Valor"/>
          <br/>
          <q-btn type="submit" label="Alterar Item" color="indigo" icon="done" class="full-width" @click.prevent="alterarItem()"/>
        </form>
      </div>
    </q-modal>

    <q-modal v-model="changeModal" minimized>
      <q-toolbar color="indigo-10">
        <q-toolbar-title>
          Alterar limite
        </q-toolbar-title>
      </q-toolbar>
      <div class="container">
        <form @submit.prevent="mudarLimite()">
          <q-input type="number" ref="altLimite" color="indigo" icon="attach money" prefix="R$ " v-model="limite" float-label="Valor"/>
          <q-btn type="submit" label="Lançar Item" color="indigo" icon="done" class="full-width" @click.prevent="mudarLimite()"/>
        </form>
      </div>
    </q-modal>

  </q-page>
</template>

<script>
import _ from 'lodash'
Array.prototype.remove = function(from, to) {
  var rest = this.slice((to || from) + 1 || this.length)
  this.length = from < 0 ? this.length + from : from
  return this.push.apply(this, rest)
}
export default {
  name: 'PageIndex',
  mounted () {
    if (!localStorage.getItem('produtos-dinheiro-contado')) {
      localStorage.setItem('produtos-dinheiro-contado', [])
    } else {
      this.produtos = JSON.parse(localStorage.getItem('produtos-dinheiro-contado'))
    }
    if (!localStorage.getItem('limite-dinheiro-contado')) {
      localStorage.setItem('limite-dinheiro-contado', [])
    } else {
      this.limite = localStorage.getItem('limite-dinheiro-contado')
    }
  },
  data () {
    return {
      produtos: [],
      limite: 0,
      addModal: false,
      changeModal: false,
      editModal: false,
      index: 0,
      form: {
        item: '',
        qnt: '',
        valor: ''
      }
    }
  },
  computed: {
    itens () {
      return this.produtos
    },
    total () {
      if (this.itens === []) {
        return 0
      }
      return _.sum( _.map(this.itens, function (i) { return i.quantidade * i.valor }))
    },
    resta () {
      return this.limite - this.total
    },
    totalDeItens () {
      return this.produtos.length
    }
  },
  methods: {
    mudarLimiteModal () {
      this.changeModal = true
      this.$refs.altLimite.focus()
    },
    openAddModal () {
      this.form = {
        item: '',
        qnt: '',
        valor: ''
      }
      this.addModal = true
      this.$refs.addItem.focus()     
    },
    formatPrice (value) {
      let val = (value / 1).toFixed(2).replace('.', ',')
      return 'R$ ' + val.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ".")
    },
    atualizarStorage () {
      localStorage.setItem('produtos-dinheiro-contado', JSON.stringify(this.produtos))
      localStorage.setItem('limite-dinheiro-contado', this.limite)
    },
    adicionarItem () {
      if (this.form.item === '' || this.form.qnt === '' || this.form.valor === '' || !this.form.item || !this.form.qnt || !this.form.valor ) {
        this.addModal = false
        return
      }
      this.produtos = _.concat(this.produtos, {produto: this.form.item, quantidade: this.form.qnt, valor: this.form.valor})
      this.form = {
        item: '',
        qnt: '',
        valor: ''
      }
      this.atualizarStorage()
      this.addModal = false
    },
    removerProduto (item) {
      let index = this.produtos.indexOf(item)
      this.produtos.remove(index)
      this.atualizarStorage()
    },
    removerTodosItens () {
      this.produtos = []
      this.atualizarStorage()
    },
    confirmarRemocaoCompleta () {
      this.$q.dialog({
        title: 'Atenção',
        message: 'Deseja remover todos os itens?',
        ok: 'Sim',
        cancel: {label: 'Não', color: 'negative', flat: true},
        color: 'indigo'
      }).then(() => {
        this.removerTodosItens()
      }).catch(() => {return})
    },
    confirmarRemocao (item) {
      this.$q.dialog({
        title: 'Atenção',
        message: 'Deseja remover este item?',
        ok: 'Sim',
        cancel: {label: 'Não', color: 'negative', flat: true},
        color: 'indigo'
      }).then(() => {
        this.removerProduto(item)
      }).catch(() => {return})
    },
    mudarLimite () {
      this.limite = this.limite
      this.form = {
        item: '',
        qnt: '',
        valor: ''
      }
      this.atualizarStorage()
      this.changeModal = false
    },
    alterarItem () {
      this.produtos[this.index] = {produto: this.form.item, quantidade: this.form.qnt, valor: this.form.valor}
      const novaLista = this.produtos
      this.produtos = []
      this.produtos = novaLista
      this.index = 0
      this.form = {
        item: '',
        qnt: '',
        valor: ''
      }
      this.atualizarStorage()
      this.editModal = false
    },
    editar (item) {
      this.form.item = item.produto
      this.form.qnt = item.quantidade
      this.form.valor = item.valor
      this.index = this.produtos.indexOf(item)
      this.editModal = true
      this.$refs.altItem.focus()
    }
  }
}
</script>
<style>
  .itensList {
    position: relative;
    top: 0;
    left: 0;
    overflow: auto;
    max-height: 65vh;
    padding-bottom: 50px !important;
  }
  .saldo-principal {
    border-radius: 100px;
  }
  .valorTotal {
    font-size: 20px;
  }
  .valores {
    margin-top: -10px;
    margin-bottom: -10px;
    border-radius: 100px;
  }
  .saldoBox {
    max-height: 25vh;
    color: white;
    position: relative;
  }
  .container {
    padding: 10px;
  }
  .item-card-content {
    padding: 7px;
    font-size: 16px;
    margin-bottom: 10px ;
  }
  .produto {
    font-size: 18px;
  }
  .quantidade{}
  h2 {
    margin-top: 10px !important;
    margin-bottom: 10px !important;
    font-weight: bold;
    font-size: 40px;
  }
  h4 {
    margin-top: 10px !important;
    margin-bottom: 10px !important;
    font-size: 16px;
  }
  h4.bg-negative {
    border-radius: 20px;
  }
</style>

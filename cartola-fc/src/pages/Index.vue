<template>
  <q-page padding>
    <!-- TABS JOGADORES/CLUBES -->
    <div class="row justify-center q-gutter-sm">
      <div class="col-xs-12 col-md-3">
        <q-tabs v-model="tab" inline-label class="bg-primary text-white shadow-2" >
          <q-tab @click="selecionadoJogadores" name='jogadores' glossy color="primary"  icon="sports_soccer" label="Jogadores" />
          <q-tab @click="SelecionadoClube" name='clube' glossy color="primary"  icon="ballot" label="Clube" />
        </q-tabs>
      </div>
    </div>
    <!-- TABELA JOGADORES -->
    <div v-if="jogadores" class="row justify-center q-gutter-sm q-ma-sm">
      <div class="col-xs-12 col-md-8">
        <q-table
          title="Jogadores"
          dense
          :data="dadosJogadores"
          :columns="columns"
          row-key="name"
          :filter="filter"
        >
          <template v-slot:top-left>
            <q-input
              v-model="filter"
              :clearable="true"
              placeholder="Pesquisar"
              size="xs"
            />
          </template>
          <template v-slot:top-right>
            <q-btn
              color="primary"
              icon-right="archive"
              label="Exportar csv"
              no-caps
              size="sm"
              @click="exportTable"
            />
          </template>
        </q-table>
      </div>
    </div>
    <!-- CARD CLUBES -->
    <div v-if="clube" class="row justify-center q-gutter-sm q-ma-sm">
      <div class="col-xs-6 col-md-3" v-for="cat in dadosClube" :key="cat.id">
        <q-card class="q-pa-md">
          <img style="width: 60px" :src="cat.escudos['60x60']">

          <q-card-section>
            <div class="text-h6">Nome: {{cat.nome}}</div>
            <div class="text-subtitle2">ID: {{cat.id}}</div>
            <div class="text-subtitle2">Nome Fantasia: {{cat.nome_fantasia}}</div>
            <div class="text-subtitle2">Abreviação: {{cat.abreviacao}}</div>
          </q-card-section>
        </q-card>
      </div>
    </div>
  </q-page>
</template>

<script>
import { exportFile } from 'quasar'

function wrapCsvValue (val, formatFn) {
  let formatted = formatFn !== void 0
    ? formatFn(val)
    : val

  formatted = formatted === void 0 || formatted === null
    ? ''
    : String(formatted)

  formatted = formatted.split('"').join('""')

  return `"${formatted}"`
}

export default {
  name: 'PageIndex',
  data() {
    return {
      filter: '',
      tab: 'jogadores',
      jogadores: true,
      clube: false,
      dadosJogadores: [],
      dadosClube: [],
      columns: [
        {
          name: 'atleta_id',
          required: true,
          label: 'ID Jogador',
          align: 'left',
          field: row => row.atleta_id,
          format: val => `${val}`,
          sortable: true
        },
        { name: 'nome', align: 'left', label: 'Nome', field: 'nome', sortable: true },
        { name: 'apelido', align: 'left', label: 'Apelido', field: 'apelido', sortable: true },
        { name: 'apelido_abreviado', align: 'left', label: 'Apelido abreviado', field: 'apelido_abreviado', sortable: true },
        { name: 'clube_id', align: 'left', label: 'Clube ID', field: 'clube_id', sortable: true },
        { name: 'jogos_num', align: 'left', label: 'Jogos', field: 'jogos_num', sortable: true },
        { name: 'media_num', align: 'left', label: 'Média', field: 'media_num', sortable: true },
        { name: 'rodada_num', align: 'left', label: 'Rodada', field: 'rodada_num', sortable: true },
        { name: 'slug', align: 'left', label: 'Slug', field: 'slug', sortable: true },
        { name: 'status_id', align: 'left', label: 'Status ID', field: 'status_id', sortable: true },
        { name: 'variacao_num', align: 'left', label: 'Variação', field: 'variacao_num', sortable: true },
      ],
      columnsClube: [
        {
          name: 'value',
          required: true,
          label: 'ID Clube',
          align: 'left',
          field: row => row.value,
          format: val => `${val}`,
          sortable: true
        },
        { name: 'nome', align: 'left', label: 'Nome', field: 'nome', sortable: true },
        { name: 'nome_fantasia', align: 'left', label: 'Nome Fantasia', field: 'nome_fantasia', sortable: true },
      ],
      teste: []
    }
  },
  mounted(){
    this.getJogadores()
    // this.getClube()
  },
  methods: {
    exportTable () {
      const content = [ this.columns.map(col => wrapCsvValue(col.label)) ].concat(
        this.dadosJogadores.map(row => this.columns.map(col => wrapCsvValue(
          typeof col.field === 'function'
            ? col.field(row)
            : row[col.field === void 0 ? col.name : col.field],
          col.format
        )).join(','))
      ).join('\r\n')

      const status = exportFile(
        'tabela-jogadores.csv',
        content,
        'text/csv'
      )

      if (status !== true) {
        this.$q.notify({
          message: 'Browser denied file download...',
          color: 'negative',
          icon: 'warning'
        })
      }
    },
    getJogadores(){
      const that = this
      // that.$axios.get('/cartola_jogadores')
      that.$axios.get(`https://cors-anywhere.herokuapp.com/api.cartolafc.globo.com/atletas/mercado`)
      .then((response) => {
        that.dadosJogadores = response.data.atletas
        that.dadosClube = response.data.clubes
      }).catch((error) => {
        console.log(error)
      })
    },
    getClube(){
      let that = this
      // that.$axios.get('/cartola_jogadores')
      that.$axios.get(`https://cors-anywhere.herokuapp.com/api.cartolafc.globo.com/clubes`)
      .then((response) => {
        that.dadosClube = response.data
        // console.log(that.dadosClube)
        // var map = new Map(Object.entries(that.dadosClube));
        // console.log(map)

        // Object.keys(that.dadosClube).forEach(function(item){
        //   // console.log(item + " - " + item.nome_fantasia);
        //   // console.log(item, that.dadosClube[item]);
        // });

        // for (var key in that.dadosClube) {
        //   var obj = that.dadosClube[key];
        //   for (var prop in obj) {
        //       that.teste = (prop + " = " + obj[prop]);
        //   }
        // }

        // for (var item in that.dadosClube){
        //   console.log(item + " - " + item.nome);
        // }
        // console.log(that.dadosClube)

        // Object.entries(that.dadosClube).forEach(([key, val]) => {
        //   that.teste = Object.keys(val).map(key => ({ key, value: val[key] }));
        //     console.log(that.teste);
        // });

        // that.teste.forEach(element => {
        //   // console.log(element)
        // });
      }).catch((error) => {
        console.log(error)
      })
    },
    selecionadoJogadores(){
      this.jogadores = true
      this.clube = false
    },
    SelecionadoClube(){
      this.jogadores = false
      this.clube = true
    }
  }
}
</script>
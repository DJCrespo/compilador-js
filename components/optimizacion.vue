<template>
  <div>
    <b-label class="title">Optimización</b-label>
    <div class="section">
      <div>
        <b-button type="is-success" :disabled="opCount == 0" @click="send">Optimizar</b-button>
      </div>
      <!--<b-table id="lexemas" :columns="columns" :data="triplos" :bordered="true"></b-table>-->
      <br>
      <!--<b-button type="is-danger is-light" icon-left="file-pdf-box" @click="exportTriplos()">Descargar PDF</b-button>-->
    </div>
  </div>
</template>

<script>
// eslint-disable-next-line
const tipoDato = /ent-|cad-|car-/g
// eslint-disable-next-line
const parentesis = /(\(|\))/g
// eslint-disable-next-line
const corchetes = /{|}/g

export default {
  name: 'optimizacionComponent',
  props: {
    text: {
      tpye: String,
      default: ''
    },
    triplos: {
      type: Array,
      default: null
    }
  },
  data () {
    return {
      textOp: '',
      triplosOp: [],
      arrayWorlds: null,
      filteredWorlds: null,
      finalWorlds: [],
      opCount: 0,
      finalArray: []
    }
  },
  watch: {
    text (newText) {
      this.finalArray = []
      this.opCount = 0
      this.triplosOp = []
      this.arrayWorlds = null
      this.filteredWorlds = null
      this.finalWorlds = []
      this.textOp = newText
      this.sliceWorlds(this.textOp)
      this.optimizitation()
    }
  },
  methods: {
    sliceWorlds (worlds) {
      this.arrayWorlds = worlds.split('\n')
      this.arrayWorlds.map(x => console.log(x))
    },
    send () {
      // console.log(this.finalArray)
      this.$emit('optimization', { array: this.finalArray, count: this.opCount })
    },
    optimizitation () {
      const outParents = []
      this.arrayWorlds.forEach((x, i) => {
        if (!x.match(parentesis) && !x.match(tipoDato) && !x.match(corchetes)) {
          this.finalArray.push(i)
          outParents.push({
            world: x.split(' = '),
            poss: i
          })
        } else {
          this.finalArray.push(x)
        }
      })
      console.log(outParents)
      const filter = outParents.map((x) => {
        x.filter = x.world[1]
        x.statusChanged = false
        return x
      })
      outParents.reverse().forEach((x) => {
        const sustitution = filter.find((y) => {
          console.log('filtrado: ', y)
          if (!x.statusChanged && x.poss !== y.poss && x.world[1] === y.filter) {
            if (x.world[0] !== y.world[0]) {
              this.opCount += 1
              x.world[1] = y.world[0]
              x.statusChanged = true
              y.statusChanged = true
              return x.world
            } else {
              return null
            }
          } else {
            return null
          }
        })
        console.log('sustitucion: ', sustitution)
        x.newWorld = typeof x.world[0] !== 'undefined' && typeof x.world[1] !== 'undefined' ? x.world[0] + ' = ' + x.world[1] : null
      })
      console.log('optimizacion: ', outParents)
      const newArray = this.finalArray.map((x, i) => {
        const world = outParents.find(y => y.poss === i)
        return typeof world !== 'undefined' ? (world.newWorld ? world.newWorld : world.world[0]) : (typeof x !== 'undefined' ? x : '')
      })
      this.finalArray = newArray
      console.log('final: ', this.finalArray)
      if (this.opCount !== 0) {
        this.$emit('optimization', { array: this.finalArray, count: this.opCount })
      }
    }
  }
}
</script>

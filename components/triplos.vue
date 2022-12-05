<template>
  <div>
    <b-label class="title">Triplos - posfijos</b-label>
    <div class="section">
      <b-table id="lexemas" :columns="columns" :data="triplos" :bordered="true"></b-table>
      <br>
      <b-button type="is-danger is-light" icon-left="file-pdf-box" @click="exportTriplos()">Descargar PDF</b-button>
    </div>
  </div>
</template>

<script>
// eslint-disable-next-line
import { jsPDF } from 'jspdf'
// eslint-disable-next-line
const variable = /PD9\d+\.\d+/g
// eslint-disable-next-line
const numeroEntero = /\-?\d+11/g
// eslint-disable-next-line
const numeroDecimal = /\-?\d+11.\d+/g
// eslint-disable-next-line
const operador = /[\+\-\*\/\|\&]/g
// eslint-disable-next-line
const tipoDato = /ent-|cad-|car-/g
// eslint-disable-next-line
const cadena = /[A-Za-z]/g
// eslint-disable-next-line
const op1 = /<|>|<=|>=|==|!=/g
// eslint-disable-next-line
const OpWhile = /(while11)/g
// eslint-disable-next-line
const op2 = /(\*|\/)/g
// eslint-disable-next-line
const op3 = /(\+|\-)/g
// eslint-disable-next-line
const parentesis = /(\(|\))/g
// eslint-disable-next-line
const andOr = /(\||\&)/g

export default {
  name: 'TriplosTable',
  props: {
    text: {
      type: String,
      default: ''
    }
  },
  data () {
    return {
      columns: [
        {
          field: 'line',
          label: 'Línea',
          centered: true
        },
        {
          field: 'DO',
          label: 'Dato objeto',
          centered: true
        },
        {
          field: 'DF',
          label: 'Dato fuente',
          centered: true
        },
        {
          field: 'OP',
          label: 'Operador',
          centered: true
        }
      ],
      triplos: [],
      linesAnalyze: null,
      tipo: '',
      i: 0,
      varCar: [],
      varEnt: [],
      varCad: [],
      operationsAfter: [],
      operations2: []
    }
  },
  watch: {
    text (newText) {
      this.triplos = []
      this.varCar = []
      this.varEnt = []
      this.varCad = []
      this.operationsAfter = []
      this.i = 0
      this.getTriplos(newText)
    }
  },
  methods: {
    getTriplos (text) {
      this.linesAnalyze = text.split('\n')
      this.linesAnalyze.forEach((line, linePosition) => {
        const words = line.split(' ')
        this.tipo = ''
        words.forEach((word, wordPosition) => {
          this.getVar(words, word, wordPosition, linePosition)
          // console.log(res)
        })
      })
    },
    getVar (words, word, wordPosition, linePosition) {
      let temporalWord
      // eslint-disable-next-line
      if(word.includes(';' || word.includes('\,'))) {
        temporalWord = word.slice(0, word.length - 1)
      } else {
        temporalWord = word
      }
      if (temporalWord.match(tipoDato)) {
        if (temporalWord !== this.tipo) {
          this.tipo = temporalWord
        }
      } else if (this.tipo === '') {
        if (temporalWord.match(OpWhile)) {
          console.log('hola while')
          console.log(temporalWord, 'While')
          const after2 = words.slice(wordPosition + 1, words.length - 1)
          // console.log(after2)
          // const parentesis = this.checkParentesis(after2)
          // console.log(parentesis)
          const andOr = this.checkAndOr(after2)
          this.getTriplosOp1(after2, andOr)
          // console.log(andOr)
        } else if (temporalWord.match('=')) {
          // console.log('hola =')
          const after = words.slice(wordPosition + 1, words.length)
          const before = words.slice(0, wordPosition)
          this.decomposeLine(before, after, linePosition)
          // eslint-disable-next-line
        } else if (temporalWord.match('\}')) {
          const index = this.triplos.findIndex(x => x.OP == null)
          // console.log(this.triplos[index])
          this.triplos.push({
            line: this.i + 1,
            DO: '',
            DF: 'JMP',
            OP: this.triplos[index].line
          })
          this.triplos.push({
            line: this.i + 2,
            DO: '',
            DF: '...',
            OP: ''
          })
          this.triplos[index].OP = this.i + 2
        }
      }
    },
    exportTriplos () {
      const columns = ['line', 'DO', 'DF', 'OP']
      // eslint-disable-next-line
      const doc = new jsPDF({ putOnlyUsedFonts: true, orientation: 'landscape' })
      doc.table(1, 1, this.triplos, columns, { autoSize: true })
      doc.save('triplos-posfijos.pdf')
    },
    decomposeLine (wordsBefore, wordsAfter) {
      this.checkOperations(wordsAfter)
      this.getTripo(this.operationsAfter, wordsAfter, wordsBefore)
    },
    checkOperations (words) {
      words.forEach((word, position) => {
        if (word.match(op2)) {
          console.log(word, 'operacion de grado 2')
          this.operationsAfter.push({
            val: 2,
            op: word,
            pos: position
          })
        } else if (word.match(op3)) {
          console.log(word, 'operacion de grado 3')
          this.operationsAfter.push({
            val: 3,
            op: word,
            pos: position
          })
        }
        // return this.operationsAfter
      })
    },
    checkParentesis (words) {
      const parentPositions = []
      words.forEach((word, position) => {
        if (word.match(parentesis)) {
          parentPositions.push({
            p: position,
            w: word
          })
        }
      })
      return parentPositions !== [] ? parentPositions : null
    },
    checkAndOr (words) {
      const andOrPositions = []
      words.forEach((word, position) => {
        if (word.match(andOr)) {
          andOrPositions.push({
            p: position,
            w: word
          })
        }
      })
      return andOrPositions !== [] ? andOrPositions : null
    },
    getTriplosOp1 (words, AndOr) {
      AndOr.forEach((x) => {
        const before = words.slice(1, x.p)
        const after = words.slice(x.p + 1, words.length - 1)
        before.forEach((z, y) => {
          if (z.match(op1)) {
            console.log(z, 'operacion de grado 1')
            this.triplos.push({
              line: this.i + 1,
              DO: 'T' + 1,
              DF: before[y - 1],
              OP: '='
            })
            this.i += 1
            this.triplos.push({
              line: this.i + 1,
              DO: 'T' + 1,
              DF: before[y + 1],
              OP: z
            })
            this.i += 1
            // eslint-disable-next-line
            if (x.w.match('\\|')) {
              console.log('Or')
              this.triplos.push({
                line: this.i + 1,
                DO: '',
                DF: 'TRUE',
                OP: this.i + 7
              })
              this.i += 1
              this.triplos.push({
                line: this.i + 1,
                DO: '',
                DF: 'FALSE',
                OP: this.i + 2
              })
              this.i += 1
              // eslint-disable-next-line
            } else if (x.w.match('\\&')) {
              console.log('AND')
              this.triplos.push({
                line: this.i + 1,
                DO: '',
                DF: 'TRUE',
                OP: this.i + 2
              })
              this.i += 1
              this.triplos.push({
                line: this.i + 1,
                DO: '',
                DF: 'FALSE',
                OP: null
              })
              this.i += 1
            }
          }
        })
        after.forEach((z, y) => {
          if (z.match(op1)) {
            console.log(z, 'operacion de grado 1')
            this.triplos.push({
              line: this.i + 1,
              DO: 'T' + 2,
              DF: after[y - 1],
              OP: '='
            })
            this.i += 1
            this.triplos.push({
              line: this.i + 1,
              DO: 'T' + 2,
              DF: after[y + 1],
              OP: z
            })
            this.i += 1
            // eslint-disable-next-line
            if (x.w.match('\\|')) {
              this.triplos.push({
                line: this.i + 1,
                DO: '',
                DF: 'TRUE',
                OP: this.i + 3
              })
              this.i += 1
              this.triplos.push({
                line: this.i + 1,
                DO: '',
                DF: 'FALSE',
                OP: null
              })
              // eslint-disable-next-line
            } else if (x.w.match('\\&')) {
              this.triplos.push({
                line: this.i + 1,
                DO: '',
                DF: 'TRUE',
                OP: this.i + 2
              })
              this.i += 1
              this.triplos.push({
                line: this.i + 1,
                DO: '',
                DF: 'FALSE',
                OP: null
              })
            }
          }
        })
      })
    },
    getTripo (op, words, wordsBefore) {
      let wordsAdaptative = words
      // eslint-disable-next-line
      const op2 = op.filter(x => x.val == 2)
      // Operaciones de nivel 2
      op2.forEach((x, y) => {
        this.triplos.push({
          line: this.i + 1,
          DO: 'T' + (y + 1),
          DF: wordsAdaptative[x.pos + 1],
          OP: '='
        })
        this.i += 1
        this.triplos.push({
          line: this.i + 1,
          DO: 'T' + (y + 1),
          DF: wordsAdaptative[x.pos - 1],
          OP: x.op
        })
        // this.i += 1
        wordsAdaptative[x.pos] = 'T' + (y + 1)
        wordsAdaptative[x.pos + 1] = null
        wordsAdaptative[x.pos - 1] = null
      })
      // Operaciones de nivel 3
      const filtered = wordsAdaptative.filter(y => y !== null)
      wordsAdaptative = filtered
      this.operationsAfter = []
      this.checkOperations(wordsAdaptative)
      const op3 = this.operationsAfter.filter(x => x.val === 3)
      op3.forEach((x, y) => {
        if (wordsAdaptative[x.pos - 1]) {
          this.triplos.push({
            line: this.i + 1,
            DO: 'T' + (y + 3),
            DF: wordsAdaptative[x.pos - 1],
            OP: '='
          })
          this.i += 1
          this.triplos.push({
            line: this.i + 1,
            DO: 'T' + (y + 3),
            DF: wordsAdaptative[x.pos + 1],
            OP: x.op
          })
          this.i += 1
          wordsAdaptative[x.pos] = 'T' + (y + 3)
          wordsAdaptative[x.pos + 1] = null
          wordsAdaptative[x.pos - 1] = null
        } else {
          this.triplos.push({
            line: this.i + 1,
            DO: 'T' + (y + 2),
            DF: wordsAdaptative[x.pos + 1],
            OP: x.op
          })
          this.i += 1
        }
      })
      this.i += 1
      // Final
      if (!wordsBefore[0].match(OpWhile)) {
        const ultimate = this.triplos.pop()
        this.triplos.push(ultimate)
        this.triplos.push({
          line: this.i,
          DO: wordsBefore[0],
          DF: ultimate.DO,
          OP: '='
        })
      }
    }
  }
}
</script>

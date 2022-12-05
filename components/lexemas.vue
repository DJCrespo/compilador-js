<template>
  <div>
    <b-label class="title">Lexemas</b-label>
    <div class="section">
      <b-table id="lexemas" :columns="columns" :data="tokens" :bordered="true"></b-table>
      <br>
      <b-button type="is-danger is-light" icon-left="file-pdf-box" @click="exportLexemas()">Descargar PDF</b-button>
    </div>
    <b-label class="title">Errores de lexemas</b-label>
    <div class="section">
      <b-table
        :columns="columnsError"
        :data="errors"
        :bordered="true"
      ></b-table>
      <br>
      <b-button type="is-danger is-light" icon-left="file-pdf-box" @click="exportErrorLexemas()">Descargar PDF</b-button>
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
const OpWhile = /(while11)/g

export default {
  name: 'LexemasTable',
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
          field: 'token',
          label: 'Tokens',
          centered: true
        },
        {
          field: 'lexema',
          label: 'Lexemas',
          centered: true
        },
        {
          field: 'tipoDato',
          label: 'Tipo de dato',
          centered: true
        }
      ],
      columnsError: [
        {
          field: 'tokenError',
          label: 'Token Error',
          centered: true
        },
        {
          field: 'lexema',
          label: 'Lexemas',
          centered: true
        },
        {
          field: 'linea',
          label: 'Línea',
          centered: true
        },
        {
          field: 'description',
          label: 'Descripción',
          centered: true
        }
      ],
      linesAnalyze: null,
      tokens: [],
      tipo: '',
      varEnt: [],
      varCar: [],
      varCad: [],
      errors: [],
      wordsInLine: [],
      variablesInOperation: []
    }
  },
  watch: {
    text (newText) {
      this.tokens = []
      this.errors = []
      this.varEnt = []
      this.varCad = []
      this.varCar = []
      this.variablesInOperation = []
      this.getLexemas(newText)
    }
  },
  methods: {
    exportLexemas () {
      // console.log(this.tokens)
      const columns = ['lexema', 'tipoDato', 'token']
      // eslint-disable-next-line
      const doc = new jsPDF({ putOnlyUsedFonts: true, orientation: 'landscape' })
      doc.table(1, 1, this.tokens, columns, { autoSize: true })
      doc.save('lexemas.pdf')
    },
    exportErrorLexemas () {
      // console.log(this.errors)
      const columns = ['description', 'lexema', 'linea', 'tokenError']
      // eslint-disable-next-line
      const doc = new jsPDF({ putOnlyUsedFonts: true, orientation: 'landscape' })
      doc.table(1, 1, this.errors, columns, { autoSize: true })
      doc.save('ErrorLexemas.pdf')
    },
    getLexemas (text) {
      this.linesAnalyze = text.split('\n')
      this.linesAnalyze.forEach((line, linePosition) => {
        const words = line.split(' ')
        this.tipo = ''
        words.forEach((word) => {
          const res = this.getVar(word, linePosition)
          if (word.includes(';') && this.tipo === '') {
            this.variablesInOperation = []
            this.operationsInLine(words, linePosition)
          } else if (!word.match(' ') && !word.match(',')) {
            this.wordsInLine.push(word)
          }
          if (res !== 'notMatch' && res !== 'setType') {
            this.tokens.push(res)
          }
        })
      })
    },
    getVar (word, linePosition) {
      let temporalWord
      // eslint-disable-next-line
      if (word.includes(';') || word.includes('\,')) {
        temporalWord = word.slice(0, word.length - 1)
      } else {
        temporalWord = word
      }
      if (temporalWord.match(tipoDato)) {
        if (temporalWord !== this.tipo) {
          this.tipo = temporalWord
          return {
            token: '',
            lexema: temporalWord,
            tipoDato: 'tipo de variable'
          }
        } else {
          return 'setType'
        }
      } else if (temporalWord.match(variable) && this.tipo !== '') {
        switch (this.tipo) {
          case 'ent-':
            if (!this.varEnt.includes(temporalWord)) {
              this.varEnt.push(temporalWord)
            }
            break
          case 'car-':
            if (!this.varCar.includes(temporalWord)) {
              this.varCar.push(temporalWord)
            }
            break
          case 'cad-':
            if (!this.varCad.includes(temporalWord)) {
              this.varCad.push(temporalWord)
            }
            break
          default:
            break
        }
        return {
          token: '',
          lexema: temporalWord,
          tipoDato: this.tipo
        }
      } else if (temporalWord.match(OpWhile)) {
        return {
          token: '',
          lexema: temporalWord,
          tipoDato: 'Operador especial'
        }
      } else if (!temporalWord.match(variable) && temporalWord.match(numeroDecimal)) {
        return {
          token: '',
          lexema: temporalWord,
          tipoDato: 'Número tipo car-'
        }
      } else if (!temporalWord.match(variable) && temporalWord.match(numeroEntero)) {
        return {
          token: '',
          lexema: temporalWord,
          tipoDato: 'Número tipo ent-'
        }
      } else if (!temporalWord.match(variable) && temporalWord.match(cadena)) {
        return {
          token: '',
          lexema: temporalWord,
          tipoDato: 'Cadena tipo cad-'
        }
      } else if (temporalWord.match(operador)) {
        // eslint-disable-next-line
        const comp = this.tokens.find((x) => x.lexema === temporalWord)
        console.log(comp)
        if (comp) {
          return 'notMatch'
        } else {
          return {
            token: '',
            lexema: temporalWord,
            tipoDato: 'Operación'
          }
        }
      } else {
        if (
          temporalWord.match(variable) &&
          !this.varEnt.includes(temporalWord) &&
          !this.varCad.includes(temporalWord) &&
          !this.varCar.includes(temporalWord) &&
          temporalWord !== ' ' &&
          temporalWord !== ';' &&
          temporalWord !== '\n'
        ) {
          this.errors.push({
            tokenError: 'ER' + (this.errors.length + 1),
            lexema: temporalWord,
            linea: (linePosition + 1).toString(),
            description: 'Variable no definida'
          })
          return {
            token: '',
            lexema: temporalWord,
            tipoDato: 'Variable indefinida'
          }
        } else {
          // console.log('Error con palabra: ' + temporalWord)
        }
        return 'notMatch'
      }
    },
    operationsInLine (words, linePosition) {
      words.forEach((word) => {
        let tempWord
        // eslint-disable-next-line
        if (word.includes(';') || word.includes('\,')) {
          tempWord = word.slice(0, word.length - 1)
        } else {
          tempWord = word
        }
        if (tempWord.match(variable)) {
          this.variablesInOperation.push(this.verifyVariable(tempWord))
        } else if (tempWord.match(numeroDecimal)) {
          this.variablesInOperation.push({
            type: 'car-',
            word: tempWord
          })
        } else if (tempWord.match(numeroEntero)) {
          this.variablesInOperation.push({
            type: 'ent-',
            word: tempWord
          })
        } else if (tempWord.match(operador)) {
          this.variablesInOperation.push({
            type: 'opa',
            word: tempWord
          })
        } else if (tempWord.match(cadena)) {
          if (!tempWord.match(variable) && !this.varEnt.includes(tempWord) && !this.varCar.includes(tempWord)) {
            this.variablesInOperation.push({
              type: 'cad-',
              word: tempWord
            })
          }
        } else if (tempWord.match('=')) {
          this.variablesInOperation.push('asign')
        }
      })
      this.verifyOperation(this.variablesInOperation, linePosition)
    },
    verifyVariable (palabra) {
      if (this.varEnt.includes(palabra)) {
        return {
          type: 'ent-',
          word: palabra
        }
      } else if (this.varCad.includes(palabra)) {
        return {
          type: 'cad-',
          word: palabra
        }
      } else if (this.varCar.includes(palabra)) {
        return {
          type: 'car-',
          word: palabra
        }
      } else {
        return 'notMatch'
      }
    },
    verifyOperation (operationsInLine, linePosition) {
      let asign = {
        type: null,
        word: null
      }
      let flagError = false
      operationsInLine.forEach((x, i) => {
        // console.log(x)
        if (asign.type === null && x === 'asign') {
          asign = {
            type: operationsInLine[i - 1].type,
            word: operationsInLine[i - 1].word
          }
          // console.log(asign)
        } else if (x.type === asign.type && !flagError) {
          // console.log('Operación con el mismo tipo de dato')
        } else if (asign.type === 'car-' && !flagError) {
          // console.log('Operacion con números reales')
        } else if (asign.type != null && ((asign.type === 'car-' && x.type === 'cad-') || (asign.type === 'ent-' && x.type === 'cad-') || x === 'notMatch')) {
          flagError = true
          this.errors.push({
            tokenError: 'ER' + (this.errors.length + 1),
            lexema: x.word,
            linea: (linePosition + 1).toString(),
            description: 'Incompatibilidad con tipo de dato de variable asignación: ' + asign.word
          })
          // console.log('Error variable no definida o incompatiblidad de datos')
          // console.log(x)
        }
      })
    }
  }
}
</script>

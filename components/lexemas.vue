<template>
  <div>
    <b-label class="title">Lexemas</b-label>
    <div class="section">
      <b-table :columns="columns" :data="tokens" :bordered="true"></b-table>
    </div>
    <b-label class="title">Errores de lexemas</b-label>
    <div class="section">
      <b-table
        :columns="columnsError"
        :data="errors"
        :bordered="true"
      ></b-table>
    </div>
  </div>
</template>

<script>
// eslint-disable-next-line
const variable = /PD9\d+\.\d+/g
// eslint-disable-next-line
const numeroEntero = /\-?\d+11/g
// eslint-disable-next-line
const numeroDecimal = /\-?\d+11.\d+/g
// eslint-disable-next-line
const operador = /[\+\-\*\/]/g
// eslint-disable-next-line
const tipoDato = /ent-|cad-|car-/g
// eslint-disable-next-line
const cadena = /[A-Za-z]/g

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
      this.getLexemas(newText)
    }
  },
  methods: {
    getLexemas (text) {
      this.linesAnalyze = text.split('\n')
      this.linesAnalyze.forEach((line, linePosition) => {
        const words = line.split(' ')
        this.tipo = ''
        words.forEach((word) => {
          const res = this.getVar(word, linePosition)
          if (word.includes(';') && this.tipo === '') {
            this.variablesInOperation = []
            this.operationsInLine(words)
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
      if (word.includes(';')) {
        temporalWord = word.slice(0, word.length - 1)
        console.log(temporalWord)
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
      } else if (temporalWord.match(numeroDecimal)) {
        return {
          token: '',
          lexema: temporalWord,
          tipoDato: 'Número tipo car-'
        }
      } else if (temporalWord.match(numeroEntero)) {
        return {
          token: '',
          lexema: temporalWord,
          tipoDato: 'Número tipo ent-'
        }
      } else if (temporalWord.match(operador)) {
        return {
          token: '',
          lexema: temporalWord,
          tipoDato: 'Número tipo operación'
        }
      } else if (temporalWord.match('while11')) {
        return {
          token: '',
          lexema: temporalWord,
          tipoDato: 'Operador especial'
        }
      } else if (temporalWord.match(cadena)) {
        return {
          token: '',
          lexema: temporalWord,
          tipoDato: 'Cadena tipo cad-'
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
            linea: linePosition + 1,
            description: 'Variable no definida'
          })
        } else {
          console.log('error 1')
        }
        return 'notMatch'
      }
    },
    operationsInLine (words) {
      words.forEach((word) => {
        let tempWord
        if (word.includes(';')) {
          tempWord = word.slice(0, word.length - 1)
        } else {
          tempWord = word
        }
        if (tempWord.match(variable)) {
          this.variablesInOperation.push(this.verifyVariable(tempWord))
        } else if (tempWord.match(numeroDecimal)) {
          this.variablesInOperation.push('car-')
        } else if (tempWord.match(numeroEntero)) {
          this.variablesInOperation.push('ent-')
        } else if (tempWord.match(operador)) {
          this.variablesInOperation.push('opa')
        } else if (tempWord.match(cadena)) {
          if (!tempWord.match(variable) && !this.varEnt.includes(tempWord) && !this.varCar.includes(tempWord)) {
            this.variablesInOperation.push('cad-')
          } else {
            console.log('variable matched')
          }
        } else if (tempWord.match('=')) {
          this.variablesInOperation.push('asign')
        }
        console.log(this.variablesInOperation)
      })
    },
    verifyVariable (word) {
      if (this.varEnt.includes(word)) {
        return 'ent-'
      } else if (this.varCad.includes(word)) {
        return 'cad-'
      } else if (this.varCar.includes(word)) {
        return 'car-'
      } else {
        return 'notMatch'
      }
    }
  }
}
</script>

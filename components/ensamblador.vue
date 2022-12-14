<template>
  <div>
    <b-label class="title">Generación de código ensamblador</b-label>
    <div class="section">
      <p class="subtitle">
        Previamente se pasará el tripo de posfijo a prefijo y luego a código
        ensamblador
      </p>
      <div class="section">
        <b-table
          :columns="columns"
          :data="ensamblador"
          :bordered="true"
        ></b-table>
        <br />
        <b-button
          type="is-danger is-light"
          icon-left="file-pdf-box"
          @click="exportTriplos()"
          >Descargar PDF</b-button
        >
      </div>
    </div>
  </div>
</template>

<script>
// eslint-disable-next-line
import { jsPDF } from 'jspdf'
// eslint-disable-next-line
const op1 = /<|>|<=|>=|==|!=/g

export default {
  name: 'ensambladorComponent',
  props: {
    triplos: {
      type: Array,
      default: null
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
          field: 'OP',
          label: 'Operador',
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
        }
      ],
      ensamblador: []
    }
  },
  watch: {
    triplos (newValue) {
      this.ensamblador = []
      this.getEnsamblador(newValue)
    }
  },
  methods: {
    exportTriplos () {
      const columns = ['line', 'OP', 'DO', 'DF']
      // eslint-disable-next-line
      const doc = new jsPDF({
        putOnlyUsedFonts: true,
        orientation: 'landscape'
      })
      doc.table(1, 1, this.ensamblador, columns, { autoSize: true })
      doc.save('triplos-posfijos.pdf')
    },
    getEnsamblador (value) {
      this.ensamblador = value.map((x, i) => {
        const objectTriplo = JSON.parse(JSON.stringify(x))
        switch (objectTriplo.OP) {
          case '=':
            objectTriplo.OP = 'MOV'
            if (objectTriplo.DO.match(/T[0-1]/g)) {
              objectTriplo.DO = 'AH'
            } else if (objectTriplo.DF.match(/T[0-1]/g)) {
              objectTriplo.DF = 'AH'
            }
            objectTriplo.flag = '='
            return objectTriplo
          case '+':
            objectTriplo.OP = 'ADD'
            if (objectTriplo.DO.match(/T[0-1]/g)) {
              objectTriplo.DO = 'AH'
            } else if (objectTriplo.DF.match(/T[0-1]/g)) {
              objectTriplo.DF = 'AH'
            }
            objectTriplo.flag = '+'
            return objectTriplo
          case '-':
            objectTriplo.OP = 'SUB'
            if (objectTriplo.DO.match(/T[0-1]/g)) {
              objectTriplo.DO = 'AH'
            } else if (objectTriplo.DF.match(/T[0-1]/g)) {
              objectTriplo.DF = 'AH'
            }
            objectTriplo.flag = '-'
            return objectTriplo
          case '*':
            objectTriplo.OP = 'MOV'
            if (objectTriplo.DO.match(/T[0-1]/g)) {
              objectTriplo.DO = 'BL'
            } else if (objectTriplo.DF.match(/T[0-1]/g)) {
              objectTriplo.DF = 'BL'
            }
            objectTriplo.flag = '*'
            return objectTriplo
          case '/':
            objectTriplo.OP = 'MOV'
            if (objectTriplo.DO.match(/T[0-1]/g)) {
              objectTriplo.DO = 'BL'
            } else if (objectTriplo.DF.match(/T[0-1]/g)) {
              objectTriplo.DF = 'BL'
            }
            objectTriplo.flag = '/'
            return objectTriplo
          case '%':
            objectTriplo.OP = 'MOV'
            if (objectTriplo.DO.match(/T[0-1]/g)) {
              objectTriplo.DO = 'BL'
            } else if (objectTriplo.DF.match(/T[0-1]/g)) {
              objectTriplo.DF = 'BL'
            }
            objectTriplo.flag = '%'
            return objectTriplo
          case '>':
            objectTriplo.OP = 'CMP'
            objectTriplo.flag = '>'
            return objectTriplo
          case '<':
            objectTriplo.OP = 'CMP'
            objectTriplo.flag = '<'
            return objectTriplo
          case '<=':
            objectTriplo.OP = 'CMP'
            objectTriplo.flag = '<='
            return objectTriplo
          case '>=':
            objectTriplo.OP = 'CMP'
            objectTriplo.flag = '>='
            return objectTriplo
          case '==':
            objectTriplo.OP = 'CMP'
            objectTriplo.flag = '=='
            return objectTriplo
          default:
            return objectTriplo
        }
      })
      this.ensamblador = this.ensamblador.map((x, i) => {
        if (x.DF === 'TRUE' || x.DF === 'FALSE') {
          const temporal = this.ensamblador[i - 1].flag
            ? this.ensamblador[i - 1]
            : this.ensamblador[i - 2]
          console.log(temporal)
          switch (temporal.flag) {
            case '==':
              console.log('flag: ' + temporal)
              return {
                OP: x.DF === 'TRUE' ? 'EQ' : 'JMP',
                DO: x.DF,
                DF: x.OP,
                line: i,
                flag: '=='
              }
            case '>':
              return {
                OP: x.DF === 'TRUE' ? 'GT' : 'JMP',
                DO: x.DF,
                DF: x.OP,
                line: i,
                flag: '>'
              }
            case '<':
              return {
                OP: x.DF === 'TRUE' ? 'GT' : 'JMP',
                DO: x.DF,
                DF: x.OP,
                line: i,
                flag: '<'
              }
            case '<=':
              return {
                OP: x.DF === 'TRUE' ? 'LE' : 'JMP',
                DO: x.DF,
                DF: x.OP,
                line: i,
                flag: '<='
              }
            case '>=':
              return {
                OP: x.DF === 'TRUE' ? 'LE' : 'JMP',
                DO: x.DF,
                DF: x.OP,
                line: i,
                flag: '>='
              }
            default:
              return x
          }
        } else {
          return x
        }
      })
      const indexs = this.ensamblador.map((x, i) => {
        return x.flag === '*' || x.flag === '/' || x.flag === '%'
          ? { index: i, flag: x.flag }
          : null
      })
      const cleanIndexs = indexs.filter(x => x)
      cleanIndexs.forEach((x) => {
        switch (x.flag) {
          case '*':
            this.ensamblador.splice(x.index, 0, {
              OP: 'MUL',
              DO: 'BL',
              DF: '',
              line: x.index + 1
            })
            this.ensamblador[x.index + 1].line += 1
            for (let i = x.index + 2; i < this.ensamblador.length; i++) {
              this.ensamblador[i].line += 2
            }
            break
          case '/':
            this.ensamblador.splice(x.index, 0, {
              OP: 'DIV',
              DO: 'BL',
              DF: '',
              line: x.index + 1
            })
            for (let i = x.index; i < this.ensamblador.length; i++) {
              this.ensamblador[i].line += 1
            }
            break
          case '%':
            this.ensamblador.splice(x.index, 0, {
              OP: 'DIV',
              DO: 'BL',
              DF: '',
              line: x.index + 1
            })
            for (let i = x.index; i < value.length; i++) {
              this.ensamblador[i].line += 1
            }
            break
        }
      })
      console.log(cleanIndexs)
      console.log('ensamblador: ', this.ensamblador)
    }
  }
}
</script>

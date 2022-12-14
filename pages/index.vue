<template>
  <section class="section has-text-centered">
    <div class="card">
      <div class="card-content">
        <h1 class="title">Compilador v2.0</h1>
        <h2 class="subtitle">Hecho por Didier Jesus Crespo Castilla</h2>
      </div>
    </div>

    <br />

    <div class="card">
      <div class="card-content">
        <div class="columns">
          <div class="column is-6 is-offset-3">
            <b-field label="Ingrese aquí el texto">
              <b-input v-model="message" type="textarea"></b-input>
            </b-field>
          </div>
        </div>
      </div>
    </div>

    <div class="card mt-3">
      <div class="card-content">
        <lexemas :text="message"></lexemas>
        <triplos :text="message" :optimization="optimizationAr" @triplos="getTriplos"></triplos>
        <optimizacion :text="message" @optimization="getValue"></optimizacion>
        <ensamblador :triplos="triplosAr"></ensamblador>
      </div>
    </div>
  </section>
</template>

<script>
export default {
  name: 'IndexPage',
  data () {
    return {
      message: null,
      optimizationAr: [],
      triplosAr: []
    }
  },
  methods: {
    getValue ({ array, count }) {
      this.$swal({
        title: 'Optimizaciones disponibles: ' + count,
        text: '¿Deseas aplicar la optimización?',
        showCancelButton: true,
        confirmButtonColor: '#3085d6',
        cancelButtonColor: '#d33',
        cancelButtonText: 'No',
        confirmButtonText: 'Si'
      }).then((result) => {
        if (result.isConfirmed) {
          this.optimizationAr = array
        }
      })
    },
    getTriplos (value) {
      this.triplosAr = value
    }
  }
}
</script>

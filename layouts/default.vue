<template>
  <div class="background">
    <section class="main-content columns">
      <div class="container column is-12">
        <Nuxt />
      </div>
    </section>
  </div>
</template>

<script>
export default {
  name: 'DefaultLayout'
}
</script>

<style>
.background {
  background-color: rgb(57, 57, 57);
}
</style>

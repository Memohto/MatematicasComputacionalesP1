<template>
  <div>
    <b-navbar toggleable="lg" type="dark" variant="success">
      <b-navbar-brand href="#">Remplazador de cadenas</b-navbar-brand>
    </b-navbar>
    <Form/>
  </div>
</template>

<script>
import Form from './components/Form.vue'

export default {
  name: 'App',
  components: {
    Form
  }
}
</script>

<style>
body {
  background-color: white;
}
</style>

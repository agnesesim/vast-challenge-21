<template>
  <b-navbar toggleable="lg" type="dark" variant="info">
    <b-navbar-brand href="#">Visual Analytics Project</b-navbar-brand>
    <b-navbar-toggle target="nav-collapse"></b-navbar-toggle>
    <b-collapse id="nav-collapse" is-nav>
      <b-navbar-nav>
        <b-nav-item href="#">Challenge 1</b-nav-item>
        <b-nav-item href="#">Challenge 2</b-nav-item>
      </b-navbar-nav>
    </b-collapse>
  </b-navbar>
</template>

<script>

export default {
    name: 'PageHeader',
}
</script>

<style scoped>
.bg-info {
    background-color: #0ac49f !important;
}
</style>

<template>
  <b-navbar class="is-dark" :fixed-top="true">
    <template #brand>
      <b-navbar-item href="https://www.shedloadofcode.com">
        <img
          src="../assets/logo.webp"
          alt="Site logo of shed with code inside it"
          width="61.08px"
          height="40px"
        />
      </b-navbar-item>
    </template>
    <template #start>
    </template>
  </b-navbar>
</template>

<script>
export default {
  components: {
    
  },
  data() {
    
  },
};
</script>

<style lang="scss">
.navbar {
  position: fixed;
  top: 0;
  width: 100%;
  -webkit-backface-visibility: hidden;
  backface-visibility: hidden;

  a.navbar-item:focus,
  a.navbar-item:focus-within,
  a.navbar-item:hover,
  a.navbar-item.is-active,
  .navbar-link:focus,
  .navbar-link:focus-within,
  .navbar-link:hover,
  .navbar-link.is-active {
    background-color: #fafafa;
    color: gray;
  }

  .navbar-link:not(.is-arrowless)::after {
    border-color: gray;
  }

  .navbar-item img {
    max-height: 2.5rem;
  }
}

@media only screen and (min-width: 768px) {
  .taginput .taginput-container.is-focusable,
  .textarea,
  .input {
    min-width: 400px;
  }
}

@media only screen and (max-width: 768px) {
  .navbar-menu {
    height: 100vh;
  }
}
</style>
<template>
  <div>
    <Header />
    <div>
      <slot name="Main"></slot>
    </div>
  </div>
</template>

<script>
import Header from "../components/Header.vue";

export default {
  name: "MainLayout",
  components: {
    Header,
  },
};
</script>

<template>
  <ToDo />
</template>

<script>
import { defineComponent } from "vue";

// Components
import ToDo from "../components/ToDo.vue";

export default defineComponent({
  name: "HomeView",

  components: {
    ToDo,
  },
});
</script>

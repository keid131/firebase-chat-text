<template>
  <div style="display:flex;">
    <div style="margin-right:10px">
      {{ userName }}:
    </div>
    <div>
      {{ text }}
    </div>
  </div>
</template>
<script lang="ts">
import { defineComponent } from "vue";

export default defineComponent({
  props: {
    userName: String,
    text: String,
  },
//   setup(props) {
//       return {}
//   },
});
</script>

<template>
  <q-page class="flex">
    <BlockList></BlockList>
  </q-page>
</template>

<script>
import { defineComponent } from 'vue';
import BlockList from "components/BlockList";

export default defineComponent({
  name: 'PageIndex',
  components: {BlockList}
})
</script>

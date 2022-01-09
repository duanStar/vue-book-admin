<template>
  <Detail :is-edit="false"/>
</template>

<script>
import Detail from './components/Detail.vue'
export default {
  name: 'Create',
  components: {
    Detail
  }
}
</script>

<style lang="scss" scoped>

</style>
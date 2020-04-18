<template>
  <div class="tool" >
    <div @click="event" >
      <i :class="iconClass"></i>
    </div>
  </div>
</template>

<script>
export default {
  name: 'tool',
  props: ['event', 'iconClass']
}

</script>

<style scoped lang="scss">
  @import "tool";
</style>

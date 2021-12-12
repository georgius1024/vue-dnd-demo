<template>
  <div @drop="onDrop" @dragover.prevent @dragenter.prevent>
    <slot></slot>
  </div>
  
</template>
<script>
export default {
  methods: {
    onDrop(event) {
      this.$emit('drop', event)
      console.log(event.target)
    }
  }
}
</script>
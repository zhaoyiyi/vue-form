<template>
  <label>
    {{label}}
    <input type="text" :value="value" @input="$emit('input', $event)" />
    <div v-if="error">{{error}}</div>
  </label>
</template>

<script>
export default {
  props: {
    value: String,
    label: String,
    error: String,
  }
}
</script>

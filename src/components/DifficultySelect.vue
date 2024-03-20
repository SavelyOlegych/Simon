<template>
  <select @change="changeDifficultyLevel">
    <option
      v-for="(level, name) in difficultyLevels"
      :key="name"
      :value="name"
      :selected="name === value"
    >
      {{ level.text }}
    </option>
  </select>
</template>

<script>
export default {
  name: "DifficultySelect",
  props: {
    value: String,
    difficultyLevels: Object,
  },
  methods: {
    changeDifficultyLevel(e) {
      this.$emit("input", 
        Array.from(e.target)
          .find(level => level.selected === true)
          .value
      );
    }
  }
};
</script>
<template>
  <select v-model="selectedLocale">
    <option
      v-for="locale in availableLocales"
      :key="locale.code"
      :value="locale.code"
    >
      {{ locale.flag }}
    </option>
  </select>
</template>

<script>
export default {
  data() {
    return {
      selectedLocale: this.$i18n.locale,
      availableLocales: [
        {
          code: "en",
          name: "English",
          flag: "🇬🇧",
          iso: "en-GB",
        },
        {
          code: "de",
          name: "Deutsch",
          flag: "🇩🇪",
          iso: "de-DE",
        },
      ],
    };
  },
  watch: {
    selectedLocale(newLocale) {
      this.$router.push(this.switchLocalePath(newLocale));
    },
  },
};
</script>

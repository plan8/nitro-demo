<template>
  <section class="section">
    <div class="container">
      <div class="info-footer">
        {{ footerText }}
      </div>
    </div>
  </section>
</template>

<script>
import { mapState } from "vuex";

export default {
  data() {
    return {};
  },
  computed: {
    ...mapState({
      item: (state) => state.player.item,
    }),
    footerText() {
      const year = new Date().getFullYear();
      return `Copyright Plan8 2019 - ${year}`;
    },
  },

  mounted() {},
  watch: {},
  beforeDestroy() {},
  methods: {},
};
</script>

<style lang="scss" scoped>
@import "@/assets/scss/mixins.scss";

.info-footer {
  font-size: 0.8rem;
  color: var(--textColor);

  @include breakpoint(sm) {
    font-size: 1rem;
  }
}
</style>

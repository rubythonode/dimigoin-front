<script>
export default {
  name: 'DimiNavbar'
}
</script>

<template>
  <div class="navbar">
    <div class="container">
      <div class="navbar__wrapper">
        <div class="navbar__brand">
          <slot name="brand" />
        </div>
        <div class="navbar__menu">
          <div class="navbar__start">
            <slot name="start" />
          </div>
          <div class="navbar__end">
            <slot name="end" />
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style lang="scss">
@import '../scss/vars';
@import '../scss/mixins';

.navbar {
  display: flex;
  width: 100%;
  min-height: 5rem;
  align-items: center;
  background-color: transparent;

  &__wrapper {
    display: flex;
  }

  &__brand {
    font-size: 24px;
    font-weight: $font-weight-extra-bold;
    justify-self: flex-start;
  }

  &__menu {
    display: flex;
    flex: 1 0;
    align-items: center;
    justify-content: space-between;

    @include until($tablet) {
      display: none;
    }
  }

  &__end {
    font-weight: $font-weight-bold;
  }
}
</style>

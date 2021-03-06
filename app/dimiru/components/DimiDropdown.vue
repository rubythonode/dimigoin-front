<script>
import DimiCard from './DimiCard.vue'
import DimiDivider from './DimiDivider.vue'

export default {
  name: 'DimiDropdown',
  components: { DimiCard, DimiDivider },

  props: {
    items: {
      type: Array,
      default: () => []
    },
    value: {
      type: Number,
      default: 0
    },

    dropup: {
      type: Boolean,
      default: false
    }
  },

  data: () => ({
    active: false,
    hovered: false
  }),

  computed: {
    classes () {
      return {
        'dropdown': true,
        'dropdown--active': this.active,
        'dropdown--dropup': this.dropup
      }
    }
  },

  mounted () {
    this.updateWidth()
    window.addEventListener('resize', () => this.updateWidth())
  },

  methods: {
    updateWidth () {
      if (!this.$refs.list || !this.$refs.items) return
      const { root, items, list } = this.$refs

      list.$el.style.display = 'block'
      const width = Math.max(...items.map(item => {
        const { display } = item.style
        item.style.display = '' // removes display: none;

        const { offsetWidth } = item
        item.style.display = display // restore

        return offsetWidth
      }))

      list.$el.style.display = ''
      root.style.width = list.$el.style.width = (width + 1) + 'px'
    },

    open () {
      this.active = true
    },

    closeDelayed () {
      if (this.timer) clearTimeout(this.timer)
      this.timer = setTimeout(() => (this.active = this.hovered), 500)
    },

    select (index) {
      this.active = false
      this.$emit('input', index)
    }
  }
}
</script>

<template>
  <div
    ref="root"
    :class="classes"
    @click="open"
    @mouseover="open(hovered = true)"
    @mouseout="closeDelayed(hovered = false)"
  >
    <div class="dropdown__view">
      {{ items[value] }}
      <span class="dropdown__view-icon icon-arrow-down" />
    </div>

    <dimi-card
      ref="list"
      class="dropdown__list"
      shadow
    >
      <template v-for="(item, index) in items">
        <p
          v-show="index !== value"
          ref="items"
          :key="`item-${index}`"
          class="dropdown__item"
          @click.stop="select(index)"
        >
          <span class="dropdown__item-name">
            {{ item }}
          </span>
        </p>

        <dimi-divider
          :key="`divider-${index}`"
          class="dropdown__item-divider"
        />
      </template>
    </dimi-card>
  </div>
</template>

<style lang="scss">
@import '../scss/vars';

.dropdown {
  position: relative;
  padding-top: 5px;
  padding-bottom: 5px;
  cursor: pointer;
  user-select: none;
  white-space: nowrap;

  &__view {
    display: flex;
    align-items: center;
    justify-content: center;
  }

  &--active &__view {
    color: $pink;
  }

  &__view-icon {
    margin-left: 0.5em;
    font-size: 50%;
  }

  &__list {
    position: absolute;
    z-index: 1;
    display: none;

    padding: 0;
    margin-top: 0.7em;
    cursor: pointer;
    user-select: none;
  }

  &--dropup &__list {
    bottom: 2.75em;
  }

  &--active &__list {
    display: block;
  }

  &__item {
    padding: 15px 24px;
    background-color: $white;
    text-align: center;
  }

  &__item:hover {
    background-color: $gray-lighten;
  }

  &__item-divider {
    position: relative !important;
    margin: 0;
  }

  &__item:last-child &__item-divider {
    display: none;
  }
}
</style>

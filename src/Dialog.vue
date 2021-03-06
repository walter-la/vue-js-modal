<template>
  <component 
    :is="$modal.context.componentName"
    name="dialog"
    height="auto"
    :classes="['v--modal', 'vue-dialog', this.params.class]"
    :width="width"
    :pivot-y="0.3"
    :adaptive="true"
    :clickToClose="clickToClose"
    :transition="transition"
    @before-open="beforeOpened"
    @before-close="beforeClosed"
    @opened="$emit('opened', $event)"
    @closed="$emit('closed', $event)"
  >
    <div class="dialog-content">
      <div
        class="dialog-c-title"
        v-if="params.title"
        v-html="params.title || ''" />
      <component
        v-if="params.component"
        v-bind="params.props"
        :is="params.component" />
      <div
        class="dialog-c-text"
        v-else
        v-html="params.text || ''" />
    </div>
    <div
      class="vue-dialog-buttons"
      v-if="buttons">
      <button
        v-for="(button, index) in buttons"
        :class="button.class || 'vue-dialog-button'"
        type="button"
        :style="buttonStyle"
        :key="index"
        v-html="button.title"
        @click.stop="click(index, $event)"
      >
        {{button.title}}
      </button>
    </div>
    <div v-else class="vue-dialog-buttons-none" />
  </component>
</template>
<script>
export default {
  name: 'VueJsDialog',
  props: {
    width: {
      type: [Number, String],
      default: 400
    },
    clickToClose: {
      type: Boolean,
      default: true
    },
    transition: {
      type: String,
      default: 'fade'
    }
  },
  data() {
    return {
      params: {},
      defaultButtons: [{ title: 'CLOSE' }]
    }
  },
  computed: {
    buttons() {
      return this.params.buttons || this.defaultButtons
    },
    /**
     * Returns FLEX style with correct width for arbitrary number of
     * buttons.
     */
    buttonStyle() {
      return {
        flex: `1 1 ${100 / this.buttons.length}%`
      }
    }
  },
  methods: {
    beforeOpened(event) {
      window.addEventListener('keyup', this.onKeyUp)

      this.params = event.params || {}
      this.$emit('before-opened', event)
    },

    beforeClosed(event) {
      window.removeEventListener('keyup', this.onKeyUp)

      this.params = {}
      this.$emit('before-closed', event)
    },

    click(buttonIndex, event, source = 'click') {
      const button = this.buttons[buttonIndex]

      if (button && typeof button.handler === 'function') {
        button.handler(buttonIndex, event, { source })
      } else {
        this.$modal.hide('dialog')
      }
    },

    onKeyUp(event) {
      if (event.which === 13 && this.buttons.length > 0) {
        const buttonIndex =
          this.buttons.length === 1
            ? 0
            : this.buttons.findIndex(button => button.default)

        if (buttonIndex !== -1) {
          this.click(buttonIndex, event, 'keypress')
        }
      }
    }
  }
}
</script>
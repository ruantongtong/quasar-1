<template>
  <q-modal
    minimized
    ref="dialog"
    @close="__dismiss()"
    :no-backdrop-dismiss="noBackdropDismiss"
    :no-esc-dismiss="noEscDismiss"
    :position="position"
  >
    <div v-if="title" class="modal-header" v-html="title"></div>
    <div v-if="message" class="modal-body modal-scroll" v-html="message"></div>

    <div v-if="form" class="modal-body modal-scroll">
      <template v-for="el in form">
        <h6 v-if="el.type === 'heading'" v-html="el.label"></h6>

        <q-input
          v-if="__isInputType(el.type)"
          :type="el.type"
          style="margin-bottom: 10px"
          v-model="el.model"
          :color="el.color"
          :placeholder="el.placeholder"
          :float-label="el.label"
          :no-pass-toggle="el.noPassToggle"
        ></q-input>

        <q-chips-input
          v-else-if="el.type === 'chips'"
          v-model="el.model"
          :color="el.color"
          :float-label="el.label"
        ></q-chips-input>

        <q-option-group
          v-else-if="['radio', 'checkbox', 'toggle'].includes(el.type)"
          :type="el.type"
          v-model="el.model"
          :color="el.color"
          :options="el.items"
          :inline="el.inline"
          :dark="el.dark"
          :keep-color="el.keepColor"
        ></q-option-group>

        <div v-if="el.type === 'slider' || el.type === 'range'" style="margin-top: 15px; margin-bottom: 10px">
          <label v-html="el.label + ' (' + (el.type === 'range' ? el.model.min + ' to ' + el.model.max : el.model) + ')'"></label>
          <component
            :is="'q-' + el.type"
            v-model="el.model"
            :min="el.min"
            :max="el.max"
            :step="el.step"
            :label="el.withLabel"
            :label-always="el.labelAlways"
            :markers="el.markers"
            :snap="el.snap"
            :square="el.square"
            :color="el.color"
            class="with-padding"
          ></component>
        </div>

        <div v-if="el.type === 'rating'" style="margin-bottom: 10px">
          <label class="block" v-html="el.label"></label>
          <q-rating v-model="el.model" :max="el.max" :icon="el.icon" :style="{fontSize: el.size || '2rem'}" :color="el.color"></q-rating>
        </div>
      </template>
    </div>
    <div v-if="progress" class="modal-body">
      <q-progress
        :percentage="progress.model"
        color="progress.color || primary"
        animate
        stripe
        :indeterminate="progress.indeterminate"
      ></q-progress>
      <span v-if="!progress.indeterminate">
        {{progress.model}} %
      </span>
    </div>

    <div
      v-if="buttons"
      class="modal-buttons"
      :class="{row: !stackButtons, column: stackButtons}"
    >
      <q-btn
        v-for="(button, index) in buttons"
        :key="index"
        @click="trigger(button.handler, button.preventClose)"
        :class="button.classes"
        :style="button.style"
        :color="button.color"
        :flat="button.flat || !button.raised && !button.push && !button.outline && !button.rounded"
        :push="button.push"
        :rounded="button.rounded"
        :outline="button.outline"
      >
        <span v-html="typeof button === 'string' ? button : button.label"></span>
      </q-btn>
    </div>
    <div class="modal-buttons row" v-if="!buttons && !noButtons">
      <q-btn flat @click="close()">OK</q-btn>
    </div>
  </q-modal>
</template>

<script>
import inputTypes from '../input/input-types'
import { QModal } from '../modal'
import { QInput } from '../input'
import { QChipsInput } from '../chips-input'
import { QOptionGroup } from '../option-group'
import { QSlider } from '../slider'
import { QRange } from '../range'
import { QRating } from '../rating'
import { QProgress } from '../progress'
import { QBtn } from '../btn'

export default {
  name: 'q-dialog',
  components: {
    QModal,
    QInput,
    QChipsInput,
    QOptionGroup,
    QSlider,
    QRange,
    QRating,
    QProgress,
    QBtn
  },
  props: {
    title: String,
    message: String,
    form: Object,
    stackButtons: Boolean,
    buttons: Array,
    noButtons: Boolean,
    progress: Object,
    onDismiss: Function,
    noBackdropDismiss: Boolean,
    noEscDismiss: Boolean,
    position: String
  },
  computed: {
    opened () {
      if (this.$refs.dialog) {
        return this.$refs.dialog.active
      }
    }
  },
  methods: {
    trigger (handler, preventClose) {
      if (typeof handler !== 'function') {
        this.close()
        return
      }

      const data = this.getFormData()

      if (preventClose) {
        handler(data, this.close)
      }
      else {
        this.close(() => { handler(data) })
      }
    },
    getFormData () {
      if (!this.form) {
        return
      }

      let data = {}

      Object.keys(this.form).forEach(name => {
        const el = this.form[name]
        if (el.type !== 'heading') {
          data[name] = el.model
        }
      })

      return data
    },
    close (fn) {
      if (!this.opened) {
        return
      }
      this.$refs.dialog.close(() => {
        if (typeof fn === 'function') {
          fn(this.getFormData())
        }
      })
    },
    __isInputType (type) {
      return inputTypes.includes(type)
    },
    __dismiss () {
      this.$root.$destroy()
      if (typeof this.onDismiss === 'function') {
        this.onDismiss(this.getFormData())
      }
    }
  },
  mounted () {
    this.$refs.dialog.open(() => {
      if (!this.$q.platform.is.desktop) {
        return
      }

      const node = this.$refs.dialog.$el.getElementsByTagName(this.form ? 'INPUT' : 'BUTTON')
      if (!node.length) {
        return
      }

      if (this.form) {
        node[0].focus()
      }
      else {
        node[node.length - 1].focus()
      }
    })
    this.$root.quasarClose = this.close
  }
}
</script>

<script>
import debounce from 'lodash/debounce'
import get from 'lodash/get'

export default {
  props: {
    component: [Function, String, Object, Function],
    value: [String, Number, Boolean],
    debounce: Number,
    // overrides form options
    options: Object,
  },

  inject: ['form'],

  methods: {
    handleInput(e) {
      // v-model
      this.$emit('input', e.target.value)

      this.form.setFieldValue(this.path, e.target.value)

      if (this.fieldOptions.validateOnInput) {
        this.validate()
      }
    },

    handleBlur() {
      if (this.fieldOptions.validateOnBlur) {
        this.validate()
      }
    },

    validate() {
      if (this.debounce) {
        this.debouncedValidateField(this.path)
      } else {
        this.form.validateField(this.path)
      }
    }
  },
  
  computed: {
    path() {
      return get(this.$vnode.data, 'model.expression', this.$el && this.$el.name)
    },
    fieldOptions() {
      return Object.assign({}, this.form.options, this.options)
    },
    debouncedValidateField() {
      return debounce(this.form.validateField, this.debounce)
    },
  },

  mounted() {
    const vModel = get(this.$vnode, 'data.model')
    // try get path from v-model, if v-model is not used then get from name
    const args = vModel ? [vModel.expression, vModel.value] : [this.$el.name, this.$el.value]

    this.form.initField(...args)
  },

  render() {
    const Component = this.component;

    const props = {
      on: {
        ...this.$listeners,
        input: this.handleInput,
        blur: this.handleBlur
      },
      attrs: this.$attrs,
    }


    return (
      <Component {...props} value={this.value} error={this.form.fields[this.path] ? this.form.fields[this.path].error : ''}>
        <this.$slots.default />
      </Component>
    )
  }

}
</script>

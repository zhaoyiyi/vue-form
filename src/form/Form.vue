<script>
import Vue from 'vue'
import set from 'lodash/set'
import get from 'lodash/get'

/**
 * form expose a way for fields to update central state of the form
 * for each field it should save states such as value, isTouched, isDirty, and error
 * form pass it down to field, and field uses callback to update the values
 */
export default {
  props: {
    schema: Object,
    onSubmit: Function,
    options: {
      type: Object,
      default: () => ({
        validateOnInput: false,
        validateOnBlur: false,
      })
    },
  },
  provide() {
    return {
      form: {
        initField: this.initField,
        setFieldValue: this.setFieldValue,
        validateField: this.validateField,
        options: this.options,
        fields: this.fields,
      },
    }
  },
  data() {
    return {
      fields: {},
      isValidating: false,
      isSubmitting: false,
    }
  },
  methods: {
    initField(path, value) {
      // use $set to make it reactive
      this.$set(this.fields, path, {
        value: value,
        error: '',
      })
    },

    setFieldValue(path, value) {
      this.fields[path].value = value
    },

    async validateField(path) {
      this.isValidating = true;

      try {
        await this.schema.validateAt(path, this.formData)
        

        this.fields[path].error = '';
      } catch (err) {
        this.fields[path].error = err.message
      }

      this.isValidating = false;

    },

    async validateForm() {
      this.isValidating = true;

      try {
        await this.schema.validate(this.formData, { abortEarly: false })

        Object.keys(this.fields).forEach((key) => {
          this.fields[key].error = '';
        })
      } catch(err) {
        if (!err.inner) {
          throw err
        }

        err.inner.forEach(({ path, message }) => {
          if (this.fields[path]) {
            this.fields[path].error = message;
          } else {
            throw new Error(`unable to find path ${path}`)
          }
        })
      }

      this.isValidating = false;
    },

    async handleSubmit(e) {
      e.preventDefault();
      e.stopPropagation();

      this.isSubmitting = true;

      try {
        await this.validateForm();
        if (this.isValid) {
          await this.onSubmit(this.formData)
        }
      } catch(err) {
        throw err;
      }

      this.isSubmitting = false;

      this.$emit('submit', this.formData)
    },

  },
  computed: {
    formData() {
      return Object.keys(this.fields).reduce((obj, key) => {

        set(obj, key, this.fields[key].value)

        return obj
      }, {})
    },

    isValid() {
      return Object.values(this.fields).every(({error}) => !error)
    }
  }
}
</script>

<template>
  <form @submit="handleSubmit" >
    <pre>{{fields}}</pre>
    <slot v-bind="{ formData, isValidating, isSubmitting, isValid }" />
  </form>
</template>


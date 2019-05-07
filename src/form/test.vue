<script>
import * as yup from "yup";
import Field from "./Field";
import Form from "./Form.vue";
import Input from './Input.vue'

export default {
  components: {
    Field,
    Form,
  },

  schema: yup.object().shape({
    firstname: yup.string().required("firstname is requried"),
    lastname: yup.string().required(),
    user: yup.object().shape({
      name: yup
        .string()
        .required()
        .test("name-test", "name async error", username => {
          return new Promise(resolve => {
            setTimeout(() => {
              if (username === "aaa") {
                resolve(true);
              } else {
                resolve(false);
              }
            }, 1000);
          });
        }),
    })
  }),

  methods: {
    handleSubmit(form) {
      console.log(form);
    }
  },

  data() {
    return {
      firstname: "123",
      lastname: "",
      user: {
        name: "",
        password: ''
      }
    };
  },

  computed: {
    input() {
      return Input;
    }
  }
};
</script>


<template>
  <div>
    <Form
      :schema="$options.schema"
      :on-submit="handleSubmit"
      v-slot="{ formData, isSubmitting, isValid }"
    >
      <pre>{{formData}}</pre>
      <pre>submitting: {{isSubmitting}}</pre>
      <pre>valid: {{isValid}}</pre>
      <label>
        firstname
        <Field
          v-model="firstname"
          component="input"
          type="text"
          name="firstname"
        />
      </label>

      <br />

        <Field
          v-model="lastname"
          :component="input"
          label="last name"
        />

      <br />

      <label>
        username
        <Field
          v-model="user.name"
          component="input"
          :debounce="500"
          :options="{ validateOnInput: true }"
        />
      </label>

      <br />

      <button>submit</button>
    </Form>
  </div>
</template>

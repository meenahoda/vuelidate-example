<template>
  <q-page class="flex flex-center">
    <q-card dark color="tertiary">
      <q-card-title>Vuelidate Examples</q-card-title>
      <q-card-main>
        <q-field
          :error="$v.form.fullName.$error"
          error-label="Must contain letters only. Max Length 50."
          dark
        >
          <q-input
            float-label="Full Name"
            v-model="form.fullName"
            color="secondary"
            dark
          />
        </q-field>

        <q-field
          :error="$v.form.username.$error"
          error-label="Must contain alphanumeric characters only. Between 5-20 characters."
        >
          <q-input
            float-label="Username"
            v-model="form.username"
            color="secondary"
            dark
          />
        </q-field>

        <q-field
          class="q-mt-md"
          label="What is your preferred method of contact?"
        >
          <q-btn-toggle
            v-model="form.preferredContact"
            toggle-color="secondary"
            :options="[
              {label: 'Mobile', value: 'MOBILE'},
              {label: 'Email', value: 'EMAIL'}
            ]"
          />
        </q-field>

        <q-field
          v-if="form.preferredContact === 'MOBILE'"
          :error="$v.form.mobile.$error"
        >
          <q-input
            float-label="Mobile Number"
            v-model="form.mobile"
            color="secondary"
            dark
            type="tel"
          />
        </q-field>

        <q-field
          v-if="form.preferredContact === 'EMAIL'"
          :error="$v.form.email.$error"
        >
          <q-input
            float-label="Email Address"
            v-model="form.email"
            color="secondary"
            dark
            type="email"
          />
        </q-field>

        <q-field
          :error="$v.form.age.$error"
          error-label="You must be between 18 and 60."
        >
          <q-input
            float-label="Age"
            v-model="form.age"
            color="secondary"
            dark
            type="number"
          />
        </q-field>

        <q-field
          :error="$v.form.dob.$error"
          error-label="Does not match age entered."
        >
          <q-datetime
            float-label="Date of Birth"
            v-model="form.dob"
            color="secondary"
            dark
            type="date"
            :max="yesterday"
          />
        </q-field>
      </q-card-main>
      <q-card-actions align="end">
        <q-btn label="submit" color="secondary" @click="submit" />
      </q-card-actions>
    </q-card>
  </q-page>
</template>

<script>
import { date } from 'quasar'
import {
  required,
  requiredIf,
  email,
  minLength,
  maxLength,
  between,
  alpha,
  alphaNum,
  integer
} from 'vuelidate/lib/validators'

const today = new Date()
const { subtractFromDate } = date

const mobile = value => value ? value.startsWith('07') && value.length === 11 && /^\d+$/.test(value) : false
const ageMatchesDOB = (dob, model) => {
  if (model.age && dob) {
    const birth = new Date(dob)
    const month = today.getMonth() - birth.getMonth()

    let age = today.getFullYear() - birth.getFullYear()

    if (month < 0 || (month === 0 && today.getDate() < birth.getDate())) {
      age--
    }

    return age === model.age
  } else {
    return false
  }
}

export default {
  name: 'PageIndex',
  data () {
    return {
      yesterday: subtractFromDate(today, { days: 1 }),
      form: {
        fullName: null,
        username: null,
        preferredContact: null,
        mobile: null,
        email: null,
        age: null,
        dob: null
      }
    }
  },
  validations: {
    form: {
      fullName: { required, alpha, maxLength: maxLength(50) },
      username: { required, alphaNum, minLength: minLength(5), maxLength: maxLength(20) },
      mobile: {
        required: requiredIf(model => model.preferredContact === 'MOBILE'),
        integer,
        mobile
      },
      email: {
        required: requiredIf(model => model.preferredContact === 'EMAIL'),
        email
      },
      age: { required, integer, between: between(18, 60) },
      dob: { ageMatchesDOB }
    }
  },
  methods: {
    submit () {
      this.$v.form.$touch()

      if (this.$v.form.$error) {
        this.$q.notify({
          type: 'warning',
          message: 'Please review the fields.',
          position: 'top'
        })
      } else {
        this.$q.notify({
          type: 'positive',
          message: 'Successful!',
          position: 'top'
        })
      }
    }
  }
}
</script>

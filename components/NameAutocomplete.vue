<template>
  <div>
    <v-autocomplete
      :loading="loading"
      :items="items"
      :search-input.sync="search"
      cache-items
      hide-no-data
      hide-details
      label="Enter Dash Name"
      solo
      :value="value"
      auto-select-first
      class="mx-auto"
      style="
        font-size: 1.6rem;
        font-weight: 700;
        padding-top: 0.5em;
        padding-bottom: 0.5em;
      "
      outlined
      @input="$emit('input', $event)"
    ></v-autocomplete>
    <v-text-field
      v-if="value.length > 1"
      v-model="loginPin"
      style="
        font-size: 1.6rem;
        font-weight: 700;
        padding-top: 0.5em;
        padding-bottom: 0.5em;
      "
      outlined
      label="DAPP PIN"
      :rules="loginPinRules"
    />
    <v-btn
      style="
        border-top-left-radius: 13px;
        border-bottom-right-radius: 13px;
        border-top-right-radius: 1px;
        border-bottom-left-radius: 1px;
      "
      large
      class="ml-2 mb-4"
      color="primary"
    >
      Log In
    </v-btn>
  </div>
</template>
<script lang="ts">
import Vue from 'vue'
import { mapActions } from 'vuex'

export default Vue.extend({
  props: { value: { type: String, default: '' } },
  data() {
    return {
      loading: false,
      items: [],
      search: null,
    }
  },
  watch: {
    async search(val) {
      console.log('val :>> ', val)
      console.log('this.value :>> ', this.value)
      console.log('this.querySelections(val) :>> ', this.querySelections(val))
      const curVal = this.value ? this.value.split(':')[0] : null
      console.log('curVal :>> ', curVal)
      val && val !== curVal && (await this.querySelections(val))
    },
  },
  methods: {
    ...mapActions(['searchDashNames']),
    async querySelections(v: string) {
      this.loading = true
      const dashNames = await this.searchDashNames(v)
      this.items = dashNames.map((name: any) => {
        return { text: name.data.label, value: `${name.data.label}:${name.id}` }
      })
      console.log('items :>> ', this.items)
      this.loading = false
    },
  },
})
</script>

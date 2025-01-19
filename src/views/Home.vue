<script lang="ts" setup>
import { z } from 'zod'
import { ref, computed, type UnwrapRef } from 'vue'
import { useZodRegle } from '@regle/zod'

// TODO can we infer this? or if keep any, infer the references in the template?
const data = ref<any>({})
const schema = computed(() => {
  return z.object({
    condition: z.boolean(),
    ...(data.value.condition === true && {
      nested: z.object({
        list: z
          .array(
            z.object({
              name: z.string(),
            }),
          )
          .default([]),
      }),
    }),
  })
})
type Schema = UnwrapRef<typeof schema>

const rewardEarly = ref(true)
const { r$ } = useZodRegle<z.infer<Schema>, Schema>(data, schema, { rewardEarly })
</script>

<template>
  <label>
    <input type="radio" v-model="r$.$fields.condition.$value" :value="true" />
    yes
  </label>

  <label>
    <input type="radio" v-model="r$.$fields.condition.$value" :value="false" />
    no
  </label>

  <div v-if="r$.$fields.condition.$value">
    <div v-for="(field, ix) in r$.$fields.nested?.$fields.list.$each">
      <input v-model="field.$fields.name.$value" :key="field.$id" />
    </div>

    <button
      @click="
        () => {
          if (r$.$fields.nested) {
            ;(r$.$fields.nested.$fields.list.$value ??= []).push({ name: 'John' })
          }
        }
      "
    >
      add ({{ r$.$fields.nested?.$fields.list?.$value?.length }})
    </button>
  </div>
</template>

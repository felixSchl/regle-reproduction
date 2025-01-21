<script setup lang="ts">
import Drawer from '@/components/Drawer.vue'
import { ref } from 'vue'
import JSONViewer from '@/components/JSONViewer.vue'
import { useRegleSchema } from '@regle/schemas'
import { z } from 'zod'

// this schema only works with mode: 'schema'
const schema1 = z.discriminatedUnion('cond', [
  z.object({
    cond: z.literal(false),
  }),
  z.object({
    cond: z.literal(true),
    timestamp: z.string(),
  }),
])

// this schema only works with mode: 'rules'
const schema2 = z.object({
  party1: z.object({ name: z.string().nonempty() }),
  party2: z.object({ name: z.string().nonempty() }),
})

const schema = schema2

const { r$ } = useRegleSchema<z.infer<typeof schema>, typeof schema>(
  // XXX this actually fails type-checking because I should at least provide `{ "cond": ... }`
  //     but it doesn't have appear to have a bearing on whether 'timestamp' is
  //     available regardless if mode is "rules." Ideally I can just leave this
  //     empty as in the beginning there's simply no data.
  // @ts-ignore
  {},
  schema,
  {
    // mode: 'rules',
    mode: 'schema',
  },
)
</script>

<template>
  <div class="px-6 text-gray-900 antialiased">
    <div class="mx-auto max-w-xl py-12 md:max-w-4xl">
      <h2 class="text-2xl mb-8">Regle reproduction</h2>

      <!-- testing schema2 (only works with mode: 'rules') -->
      <template v-if="schema === schema2">
        <div class="flex flex-col gap-2">
          <template v-if="r$.$fields.party1?.$fields?.name">
            <input
              type="text"
              v-model="r$.$fields.party1.$fields.name.$value"
              class="border border-gray-300 rounded-md p-1"
              placeholder="Party 1 Name"
            />
            <div class="text-xs text-red-500" v-if="r$.$fields.party1.$fields.name.$error">
              {{ r$.$fields.party1.$fields.name.$errors[0] }}
            </div>
          </template>

          <template v-if="r$.$fields.party2?.$fields?.name">
            <input
              type="text"
              v-model="r$.$fields.party2.$fields.name.$value"
              class="border border-gray-300 rounded-md p-1 mt-4"
              placeholder="Party 2 Name"
            />
            <div class="text-xs text-red-500" v-if="r$.$fields.party2.$fields.name.$error">
              {{ r$.$fields.party2.$fields.name.$errors[0] }}
            </div>
          </template>
        </div>
      </template>

      <!-- testing schema1 (only works with mode: 'schema') -->
      <template v-else>
        <div class="flex flex-col">
          <h1 class="text-2xl mb-2">Condition:</h1>

          <!-- XXX this won't show if mode = 'rules' -->
          <select
            v-if="r$.$fields.cond"
            class="border border-gray-300 rounded-md p-1 w-fit min-w-[100px]"
            :value="r$.$fields.cond.$value"
            @change="
              (event) => {
                if (r$.$fields.cond == null) return
                r$.$fields.cond.$value = (event?.target as HTMLSelectElement)?.value === 'true'
              }
            "
          >
            <option value="true">TRUE</option>
            <option value="false">FALSE</option>
          </select>

          <div class="mt-4 flex flex-col gap-2" v-if="r$.$fields.cond?.$value === true">
            <h2>Select a Date:</h2>
            <input type="text" v-model="r$.$fields.timestamp.$value" placeholder="Pick Date" />
          </div>

          <button class="border p-2 rounded-md font-bold mt-8" @click="r$.$validate()">
            validate
          </button>
        </div>
      </template>

      <Drawer>
        <JSONViewer :data="r$" />
      </Drawer>
    </div>
  </div>
</template>

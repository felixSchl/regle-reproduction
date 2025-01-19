<script lang="ts" setup>
import { z } from 'zod';
import {ref,computed} from 'vue';
import { useZodRegle} from '@regle/zod'
import JSONViewer from '../components/JSONViewer.vue'
import Drawer from '../components/Drawer.vue'

const data = ref<
  Partial<{
    condition: boolean;
    nested: {
      list: {
        name: string;
      }[];
    };
  }>
>({});

const schema = computed(() => {
  if (data.value.condition === true) {
    return z.object({
      condition: z.boolean(),
      nested: z.object({
        list: z.array(
          z.object({
            name: z.string(),
          }),
        ),
      }),
    });
  }
  return z.object({
    condition: z.boolean(),
  });
});

const rewardEarly = ref(true);
const { r$ } = useZodRegle(data, schema, { rewardEarly });
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
          const list = r$.$fields.nested?.$fields.list?.$value ?? [];
          r$.$fields.nested.$fields.list.$value = list;
          list.push({ name: 'john' });
        }
      "
    >
      add ({{ r$.$fields.nested?.$fields.list?.$value?.length }})
    </button>
  
    <!--
    <Drawer>
      <JSONViewer :data="r$" />
    </Drawer>
    -->
  </div>
</template>


<template>
  <q-page>
    <div class="q-pa-md">
      <q-list bordered separator>
        <q-slide-item
          @right="onEntrySlideRight($event, entry)"
          v-for="entry in entries"
          :key="entry.id"
          left-color="positive"
          right-color="negative"
        >
          <!--<template v-slot:left>
            <q-icon name="done" />
          </template>-->

          <template v-slot:right>
            <q-icon name="delete" />
          </template>

          <q-item>
            <q-item-section
              class="text-weight-bold"
              :class="useAmountColorClass(entry.amount)"
            >
              {{ entry.name }}
            </q-item-section>

            <q-item-section
              class="text-weight-bold"
              :class="useAmountColorClass(entry.amount)"
              side
            >
              {{ useCurrencify(entry.amount) }}
            </q-item-section>
          </q-item>
        </q-slide-item>
      </q-list>
    </div>

    <q-footer class="bg-transparent">
      <div class="row q-mb-sm q-px-md q-py-sm shadow-up-3">
        <div class="col text-grey-7 text-h6">
          Balance:
        </div>

        <div :class="useAmountColorClass(balance)" class="col text-h6 text-right">
          {{ useCurrencify(balance) }}
        </div>
      </div>

      <q-form @submit.prevent="addEntry" class="row q-px-sm q-pb-sm q-col-gutter-sm bg-primary">
        <div class="col">
          <q-input
            v-model="addEntryForm.name"
            ref="nameRef"
            input-class="text-right"
            placeholder="Name"
            bg-color="white"
            outlined
            dense
          />
        </div>
        <div class="col">
          <q-input
            v-model.number="addEntryForm.amount"
            placeholder="Amount"
            bg-color="white"
            type="number"
            step="0.01"
            outlined
            dense
          />
        </div>
        <div class="col col-auto">
          <q-btn
            type="submit"
            color="primary"
            icon="add"
            round
          />
        </div>
      </q-form>
    </q-footer>
  </q-page>
</template>

<script setup>
import { ref, computed, reactive } from 'vue';
import { uid, useQuasar } from 'quasar';
import { useCurrencify } from 'src/use/useCurrencify';
import { useAmountColorClass } from 'src/use/useAmountColorClass';

const $q = useQuasar();

const entries = ref([
  { id: 'id1', name: 'Salary', amount: 4999.99 },
  { id: 'id2', name: 'Rent', amount: -999 },
  { id: 'id3', name: 'Phone', amount: -14.99 },
  { id: 'id4', name: 'Unknown', amount: 0 },
]);

/* Balance Calculation */
const balance = computed(() => {
  return entries.value.reduce((accumulator, { amount }) => accumulator + amount, 0);
});

/* Add entry form */
const nameRef = ref(null);

const addEntryFormDefault = {
  name: '',
  amount: null,
};

const addEntryForm = reactive({ ...addEntryFormDefault });

const addEntryFormReset = () => {
  Object.assign(addEntryForm, addEntryFormDefault);
  nameRef.value.focus();
};

const addEntry = () => {
  const newEntry = { ...addEntryForm, id: uid() };
  entries.value.push(newEntry);
  addEntryFormReset();
};

/* Handle right slide actions */
const onEntrySlideRight = ({ reset }, entry) => {
  $q.dialog({
  title: 'Delete entry',
  message: `
    Delete this entry?
    <div class="text-weight-bold ${useAmountColorClass(entry.amount)}">
      ${entry.name} : ${useCurrencify(entry.amount)}
    </div>
  `,
    cancel: true,
    persistent: true,
    html: true,
    ok: {
      label: 'Delete',
      color: 'negative',
      noCaps: true,
    },
    cancel: {
      label: 'Cancel',
      color: 'primary',
      noCaps: true,
    },
  })
    .onOk(() => deleteEntry(entry.id))
    .onCancel(() => reset());
};

/* Delete entry */
const deleteEntry = (entryId) => {
  const index = entries.value.findIndex((entry) => entry.id === entryId);
  entries.value.splice(index, 1);
  $q.notify({
    message: 'Entry deleted',
    position: 'top'

  })


};
</script>

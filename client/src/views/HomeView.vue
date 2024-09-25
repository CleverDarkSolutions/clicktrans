<template>
  <div class="container mx-auto p-6">
    <div v-if="!isSubmitted" class="grid grid-cols-1 gap-4 lg:grid-cols-2">
      <form @submit.prevent="handleSubmit" class="space-y-6">
        <div class="col-span-2">
          <label for="description" class="block text-sm font-medium text-gray-700">Description</label>
          <Textarea
              id="description"
              v-model="form.description"
              :maxlength="255"
              rows="3"
              autoResize
              @input="checkDescriptionLength"
              class="w-full border rounded-md shadow-sm focus:ring-indigo-500 focus:border-indigo-500"
          />
          <div class="text-sm text-gray-500">
            Characters left: {{ 255 - form.description.length }}
          </div>
          <small v-if="errors.description" class="text-red-600">{{ errors.description }}</small>
        </div>
        <div class="col-span-2">
          <label class="block text-sm font-medium text-gray-700">Send confirmation</label>
          <div class="flex space-x-4">
            <div class="flex items-center">
              <RadioButton inputId="yes" v-model="form.confirmation" value="yes" />
              <label for="yes" class="ml-2 text-sm">Yes</label>
            </div>
            <div class="flex items-center">
              <RadioButton inputId="no" v-model="form.confirmation" value="no" />
              <label for="no" class="ml-2 text-sm">No</label>
            </div>
          </div>
          <small v-if="errors.confirmation" class="text-red-600">{{ errors.confirmation }}</small>
        </div>

        <div class="col-span-1">
          <label for="vat" class="block text-sm font-medium text-gray-700">VAT</label>
          <Dropdown
              id="vat"
              v-model="form.vat"
              :options="vatOptions"
              optionLabel="label"
              placeholder="Choose VAT"
              @change="enableNettoInput"
              class="w-full border rounded-md shadow-sm focus:ring-indigo-500 focus:border-indigo-500"
          />
          <small v-if="errors.vat" class="text-red-600">{{ errors.vat }}</small>
        </div>

        <div class="col-span-1">
          <label for="priceNetto" class="block text-sm font-medium text-gray-700">Price Netto EUR</label>
          <InputText
              id="priceNetto"
              v-model="form.priceNetto"
              :disabled="!vatSelected"
              placeholder="Enter price netto"
              @input="validateNettoPrice"
              class="w-full border rounded-md shadow-sm focus:ring-indigo-500 focus:border-indigo-500"
          />
          <small v-if="errors.priceNetto" class="text-red-600">{{ errors.priceNetto }}</small>
        </div>

        <div class="col-span-1">
          <label for="priceBrutto" class="block text-sm font-medium text-gray-700">Price Brutto EUR</label>
          <InputText
              id="priceBrutto"
              v-model="priceBrutto"
              disabled
              class="w-full border rounded-md shadow-sm focus:ring-indigo-500 focus:border-indigo-500"
          />
        </div>

        <div class="col-span-2">
          <Button label="Submit" icon="pi pi-check" type="submit" class="w-full bg-indigo-600 text-white font-medium py-2 px-4 rounded-md hover:bg-indigo-700" />
        </div>
      </form>

      <small v-if="errors.submit" class="text-red-600 col-span-2">{{ errors.submit }}</small>
    </div>
    <div v-else class="success-message text-center p-4 bg-green-100 text-green-700 rounded-lg">
      <h2>Congratulations! Your form has been submitted successfully.</h2>
    </div>
  </div>
</template>

<script setup lang="ts">
import { reactive, ref } from 'vue';
import InputText from 'primevue/inputtext';
import Textarea from 'primevue/textarea';
import RadioButton from 'primevue/radiobutton';
import Dropdown from 'primevue/dropdown';
import Button from 'primevue/button';

const form = reactive({
  description: '',
  confirmation: '',
  vat: '',
  priceNetto: '',
});

const errors = reactive({
  description: '',
  confirmation: '',
  vat: '',
  priceNetto: '',
  submit: '',
});

const vatSelected = ref(false);
const priceBrutto = ref('');
const isSubmitted = ref(false);

const vatOptions = [
  { label: '19%', value: '19' },
  { label: '21%', value: '21' },
  { label: '23%', value: '23' },
  { label: '25%', value: '25' }
];

function checkDescriptionLength() {
  if (!form.description) {
    errors.description = 'Text is required';
  } else if (form.description.length > 255) {
    errors.description = 'You can’t enter more than 255 characters';
  } else {
    errors.description = '';
  }
}

function enableNettoInput() {
  vatSelected.value = !!form.vat;
  updateBruttoPrice();
}

function validateNettoPrice() {
  const price = form.priceNetto.replace(',', '.');
  if (isNaN(parseFloat(price)) || price === '') {
    errors.priceNetto = 'Please, input number';
  } else {
    errors.priceNetto = '';
    updateBruttoPrice();
  }
}

function updateBruttoPrice() {
  const priceNetto = parseFloat(form.priceNetto.replace(',', '.'));
  const vat = parseFloat(form.vat);
  if (!isNaN(priceNetto) && vat) {
    priceBrutto.value = (priceNetto * (1 + vat / 100)).toFixed(2);
  } else {
    priceBrutto.value = '';
  }
}

function validateForm(): boolean {
  let isValid = true;

  if (!form.description) {
    errors.description = 'Text is required';
    isValid = false;
  }

  if (!form.confirmation) {
    errors.confirmation = 'Text is required';
    isValid = false;
  }

  if (!form.vat) {
    errors.vat = 'Text is required';
    isValid = false;
  }

  if (!form.priceNetto || isNaN(parseFloat(form.priceNetto.replace(',', '.')))) {
    errors.priceNetto = 'Please, input number';
    isValid = false;
  }

  return isValid;
}

async function handleSubmit() {
  if (validateForm()) {
    try {
      const response = await fetch('https://jsonplaceholder.typicode.com/posts', {
        method: 'POST',
        body: JSON.stringify(form),
      });

      if (!response.ok) {
        throw new Error('Submission failed');
      }

      isSubmitted.value = true;
    } catch (error) {
      errors.submit = 'Error submitting the form, please try again.';
    }
  }
}
</script>

<style scoped>
.success-message {
  color: green;
}
</style>

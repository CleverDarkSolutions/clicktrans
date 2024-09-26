<template>
  <div class="container mx-auto p-6">
    <div v-if="!isSubmitted" class="grid grid-cols-1 gap-4 lg:grid-cols-2">
      <form @submit.prevent="handleSubmit" class="space-y-6">
        <div class="col-span-2">
          <label for="description" class="block text-sm font-medium text-gray-700">Description</label>
          <Textarea
              id="description"
              v-model="description"
              :maxlength="255"
              rows="3"
              autoResize
              @input="checkDescription"
              class="w-full border rounded-md shadow-sm focus:ring-indigo-500 focus:border-indigo-500"
          />
          <div class="text-sm text-gray-500">
            Characters left: {{ 255 - description.length }}
          </div>
          <small v-if="errors.description" class="text-red-600">{{ errors.description }}</small>
        </div>
        <div class="col-span-2">
          <label class="block text-sm font-medium text-gray-700">Send confirmation</label>
          <div class="flex space-x-4">
            <div class="flex items-center" @click="checkConfirmation">
              <RadioButton inputId="yes" v-model="confirmation" value="yes" />
              <label for="yes" class="ml-2 text-sm">Yes</label>
            </div>
            <div class="flex items-center" @click="checkConfirmation">
              <RadioButton inputId="no" v-model="confirmation" value="no" />
              <label for="no" class="ml-2 text-sm">No</label>
            </div>
          </div>
          <small v-if="errors.confirmation" class="text-red-600">{{ errors.confirmation }}</small>
        </div>

        <div class="col-span-1">
          <label for="vat" class="block text-sm font-medium text-gray-700">VAT</label>
          <Dropdown
              id="vat"
              v-model="vat"
              :options="vatOptions"
              optionLabel="label"
              placeholder="Choose VAT"
              @change="checkVat"
              class="w-full border rounded-md shadow-sm focus:ring-indigo-500 focus:border-indigo-500"
          />
          <small v-if="errors.vat" class="text-red-600">{{ errors.vat }}</small>
        </div>

        <div class="col-span-1">
          <label for="priceNetto" class="block text-sm font-medium text-gray-700">Price Netto EUR</label>
          <InputText
              id="priceNetto"
              v-model="priceNetto"
              @input="checkNettoPrice"
              :disabled="!vatSelected"
              placeholder="Enter price netto"
              class="w-full border rounded-md shadow-sm focus:ring-indigo-500 focus:border-indigo-500"
          />
          <small v-if="errors.priceNetto" class="text-red-600">{{ errors.priceNetto }}</small>
        </div>

        <div class="col-span-1">
          <label for="priceBrutto" class="block text-sm font-medium text-gray-700">Price Brutto EUR</label>
          <InputText
              id="priceBrutto"
              v-model="computedPriceBrutto"
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
import { ref, computed } from 'vue';
import Textarea from 'primevue/textarea';
import RadioButton from 'primevue/radiobutton';
import Dropdown from 'primevue/dropdown';
import Button from 'primevue/button';
import InputText from 'primevue/inputtext';

const description = ref('');
const confirmation = ref('');
const vat = ref(0);
const priceNetto = ref('');
const errors = ref({
  description: '',
  confirmation: '',
  vat: '',
  priceNetto: '',
  submit: '',
});
const vatSelected = ref(false);
const isSubmitted = ref(false);

const vatOptions = [
  { label: '19%', value: 19 },
  { label: '21%', value: 21 },
  { label: '23%', value: 23 },
  { label: '25%', value: 25 }
];

const computedPriceBrutto = computed(() => {
  console.log(vat.value);
  console.log(priceNetto.value)
  const price = Number(priceNetto.value) * (1 + (vat.value.value / 100));
  if(isNaN(price)){
    return 0
  }
  else{
    return price.toString();
  }
});

function checkDescription() {
  if (!description.value) {
    errors.value.description = 'Text is required';
  } else if (description.value.length > 255) {
    errors.value.description = 'You can’t enter more than 255 characters';
  } else {
    errors.value.description = '';
  }
}

function checkConfirmation(){
  console.log(confirmation.value);
  if (!confirmation.value) {
    errors.value.confirmation = 'Selection is required';
  }
  else {
    errors.value.confirmation = '';
  }
}

function checkVat() {
  console.log(vat.value)
  if (!vat.value) {
    errors.value.vat = 'Text is required';
  }
  else{
    errors.value.vat = '';
  }
  vatSelected.value = !!vat.value;
}

function checkNettoPrice() {
  const numberPattern = /^[0-9]*[.,]?[0-9]+$/;
  if (!numberPattern.test(priceNetto.value)) {
    errors.value.priceNetto = 'Please, input number';
  } else {
    errors.value.priceNetto = '';
  }
}

function updateAssist() {

}

function validateForm(){
  checkDescription()
  checkConfirmation()
  checkVat()
  checkNettoPrice()
  console.log(description.value !== '')
  console.log(confirmation.value !== '')
  console.log(vat.value.value > 0)
  console.log(priceNetto.value !== '')
  if(description.value !== '' && confirmation.value !== '' && vat.value.value > 0 && priceNetto.value !== ''){
    return true;
  }
  else {
    console.log('Failed', 'Desc: ',description.value,'Conf: ',confirmation.value,'VAT: ',vat.value.value, 'netto: ',priceNetto.value, 'Brutto: ', computedPriceBrutto.value)
  }
}

async function handleSubmit() {
  if (validateForm()) {
    try {
      const response = await fetch('https://jsonplaceholder.typicode.com/posts', {
        method: 'POST',
        body: JSON.stringify({
          description: description.value,
          confirmation: confirmation.value,
          vat: vat.value.value,
          priceNetto: priceNetto.value,
          priceBrutto: computedPriceBrutto.value
        }),
      });

      if (!response.ok) {
        throw new Error('Submission failed');
      }

      isSubmitted.value = true;
    } catch (error) {
      console.log(error)
      errors.value.submit = 'Error submitting the form, please try again.';
    }
  }
}
</script>

<style scoped>
.success-message {
  color: green;
}
</style>

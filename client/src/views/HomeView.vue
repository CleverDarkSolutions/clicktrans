<template>
  <div class="container mx-auto p-6">
    <div v-if="!isSubmitted">
      <form @submit.prevent="handleSubmit" class="space-y-6">

        <div class="relative col-span-2 flex items-center">
          <label for="description" class="block text-sm font-medium text-gray-700">Description</label>
          <div class="flex-1 ml-4">
            <Textarea
                id="description"
                v-model="description"
                :maxlength="255"
                rows="3"
                autoResize
                @input="checkDescription"
                class="w-full border rounded-md shadow-sm focus:ring-indigo-500 focus:border-indigo-500"
            />
          </div>
          <small v-if="errors.description" class="text-red-600 absolute right-[-140px]">{{ errors.description }}</small>
        </div>

        <div class="relative col-span-2 flex items-center">
          <label class="block text-sm font-medium text-gray-700">Send confirmation</label>
          <div class="flex space-x-4 ml-4">
            <div class="flex items-center">
              <RadioButton
                  @change="checkConfirmation"
                  inputId="yes"
                  v-model="confirmation"
                  value="yes"
              />
              <label for="yes" class="ml-2 text-sm">Yes</label>
            </div>
            <div class="flex items-center">
              <RadioButton
                  @change="checkConfirmation"
                  inputId="no"
                  v-model="confirmation"
                  value="no"
              />
              <label for="no" class="ml-2 text-sm">No</label>
            </div>
          </div>
          <small v-if="errors.confirmation" class="text-red-600 absolute right-[-140px]">{{ errors.confirmation }}</small>
        </div>

        <div class="relative col-span-1 flex items-center">
          <label for="vat" class="block text-sm font-medium text-gray-700">VAT</label>
          <div class="flex-1 ml-4">
            <Dropdown
                id="vat"
                v-model="vat"
                :options="vatOptions"
                optionLabel="label"
                placeholder="Choose VAT"
                @change="checkVat"
                class="w-full border rounded-md shadow-sm focus:ring-indigo-500 focus:border-indigo-500"
            />
          </div>
          <small v-if="errors.vat" class="text-red-600 absolute right-[-140px]">{{ errors.vat }}</small>
        </div>

        <div class="relative col-span-1 flex items-center">
          <label for="priceNetto" class="block text-sm font-medium text-gray-700">Price Netto EUR</label>
          <div class="flex-1 ml-4">
            <InputText
                id="priceNetto"
                v-model="priceNetto"
                @input="checkNettoPrice"
                :disabled="!vatSelected"
                placeholder="Enter price netto"
                class="w-full border rounded-md shadow-sm focus:ring-indigo-500 focus:border-indigo-500"
            />
          </div>
          <small v-if="errors.priceNetto" class="text-red-600 absolute right-[-140px]">{{ errors.priceNetto }}</small>
        </div>

        <div class="col-span-1 flex items-center">
          <label for="priceBrutto" class="block text-sm font-medium text-gray-700">Price Brutto EUR</label>
          <div class="flex-1 ml-4">
            <InputText
                id="priceBrutto"
                v-model="computedPriceBrutto"
                disabled
                class="w-full border rounded-md shadow-sm focus:ring-indigo-500 focus:border-indigo-500"
            />
          </div>
        </div>

        <div>
          <Button label="Submit" icon="pi pi-check" type="submit" class="w-full bg-indigo-600 text-white font-medium py-2 px-4 rounded-md hover:bg-indigo-700" />
          <small v-if="errors.submit" class="text-red-600 col-span-2 right-[-140px]">{{ errors.submit }}</small>
        </div>
      </form>
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
  const normalizedNetto = priceNetto.value.replace(',', '.');
  const price = Number(normalizedNetto) * (1 + (vat.value.value / 100));
  if (isNaN(price)) {
    return 0;
  } else {
    return price.toFixed(2);
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
  if (!confirmation.value) {
    errors.value.confirmation = 'Text is required';
  }
  else {
    errors.value.confirmation = '';
  }
}

function checkVat() {
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
  const normalizedPrice = priceNetto.value.replace(',', '.');
  if (!numberPattern.test(priceNetto.value) || isNaN(parseFloat(normalizedPrice))) {
    errors.value.priceNetto = 'Please, input number';
  } else {
    errors.value.priceNetto = '';
  }
}
function validateForm(){
  checkDescription()
  checkConfirmation()
  checkVat()
  checkNettoPrice()
  if(description.value !== '' && confirmation.value !== '' && vat.value.value > 0 && priceNetto.value !== ''){
    return true;
  }
  else {
    errors.value.submit = 'Error submitting the form, please try again.';
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

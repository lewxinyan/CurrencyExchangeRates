<template>
    <v-select
        :id="id"
        v-model="selectedCurrency"
        :items="currencies"
        item-title="name"
        item-value="code"
        :label="label"
        return-object
    >
        <template v-slot:selection="{item}">
            {{ item.raw.code }} - {{ item.raw.name }}
        </template>
        <template v-slot:item="{ props, item}">
            <v-list-item v-bind="props" :title="`${item.raw.code} - ${item.raw.name}`"></v-list-item>
        </template>
    </v-select>
</template>

<script>
import {ref, watch} from 'vue';

export default {
    name: 'CurrencySelect',
    props:{
        id:{
            type:String,
            required: true
        },
        label:{
            type:String,
            default: 'Select a currency'
        },
        modelValue: {
            type:Object,
            default:null
        },
    },
    emits:['update:modelValue'],

    setup(props, {emit}) {
        const currencies = [
            {code: 'USD', name: 'US Dollar'},
            {code: 'EUR', name: 'Euro'},
            {code: 'GBP', name: 'British Pound'},
            {code: 'JPY', name: 'Japanese Yen'},
            {code: 'CHF', name: 'Swiss Franc'},
            {code: 'AUD', name: 'Australian Dollar'},
            {code: 'CAD', name: 'Canadian Dollar'},
            {code: 'NZD', name: 'New Zealand Dollar'},
            {code: 'CNY', name: 'Chinese Renminbi'},
            {code: 'INR', name: 'Indian Rupee'},
            {code: 'MYR', name: 'Malaysian Ringgit'},
            {code: 'SGD', name: 'Singapore Dollar'},
            {code: 'BRL', name: 'Brazilian Real'},
            {code: 'RUB', name: 'Russian Ruble'},
            {code: 'ZAR', name: 'South African Rand'},
            {code: 'MXN', name: 'Mexican Peso'},
            {code: 'TRY', name: 'Turkish Lira'},
            {code: 'KRW', name: 'South Korean Won'},
            {code: 'SEK', name: 'Swedish Krona'},
            {code: 'NOK', name: 'Norwegian Krone'},
            {code: 'DKK', name: 'Danish Krone'},
        ];

        const selectedCurrency = ref(props.modelValue);

        watch(selectedCurrency,(newValue) => {
            emit('update:modelValue', newValue);
        });

        return{
            currencies,
            selectedCurrency,
        };
    },
};
</script>
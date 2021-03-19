<template>
<div>
    <span class="axis-header text-xl font-semibold block">
        Данные по габаритам
    </span>

    <div class="grid grid-cols-3 border-b-2">
        <span class="col-span-1 text-center font-semibold">
            Измерение
        </span>
        <span class="col-span-1 text-center font-semibold">
            Максимальное значение
        </span>
        <span class="col-span-1 text-center font-semibold">
            Фактическое значение
        </span>
    </div>

    <div class="grid grid-cols-3 gap-x-3 mt-2" v-for="(val,key) in dimensionData" :key="key">
        <div class="col-span-1 text-semibold">
            {{ val.title }}
        </div>
        <div class="col-span-1 text-center text-semibold">
            <input type="number" :class="inputClass" v-model.number="dimensionData[key].max" @input="updateValue">
        </div>
        <div class="col-span-1 text-center text-semibold">
            <input type="number" :class="inputClass" v-model.number="dimensionData[key].has" @input="updateValue">
        </div>
    </div>
</div>
</template>

<script>
export default {
    props: ['value'],

    data() {
        return {
            dimensionData: {
                length: {
                    title: "Длинна",
                    max: 0,
                    has: 0
                },
                width: {
                    title: "Ширина",
                    max: 0,
                    has: 0
                },
                height: {
                    title: "Высота",
                    max: 0,
                    has: 0
                },
            }
        }
    },

    computed: {
        inputClass () {
            return 'focus:ring-indigo-500 focus:border-indigo-500 flex-1 block rounded-md sm:text-sm border-gray-300 border-2 p-1 w-full'
        }
    },

    methods: {
        updateValue () {
            this.$emit("input", this.dimensionData)
        }
    },
}
</script>
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

    <div class="grid grid-cols-3 gap-x-3 mt-2" v-for="(title,key) in loopKeys" :key="key">
        <div class="col-span-1 text-semibold">
            {{ title }}
        </div>
        <div class="col-span-1 text-center text-semibold">
            <input type="number" class="focus:ring-indigo-500 focus:border-indigo-500 flex-1 block rounded-md sm:text-sm border-gray-300 border-2 p-1 w-full" v-model.number="dimensionData.max[key]" @input="updateValue">
        </div>
        <div class="col-span-1 text-center text-semibold">
            <input type="number" class="focus:ring-indigo-500 focus:border-indigo-500 flex-1 block rounded-md sm:text-sm border-gray-300 border-2 p-1 w-full" v-model.number="dimensionData.has[key]" @input="updateValue">
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
                max: {
                    widht: 0,
                    height: 0,
                    length: 0,
                },
                has: {
                    widht: 0,
                    height: 0,
                    length: 0,
                }
            }
        }
    },

    computed: {
        loopKeys () {
            return {
                "length": "Длинна",
                "height": "Высота",
                "width": "Ширина",
            }
        }
    },

    methods: {
        updateValue () {
            this.$emit("input", this.dimensionData)
        }
    },
}
</script>
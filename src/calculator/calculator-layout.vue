<template>
<div class="block w-full max-w-screen-xl p-3">
    <div class="no-print-1">
        <axis-data v-model="axisData" :rules="rules" />

        <hr class="my-3">

        <dimension-data v-model="dimensionData" />

        <hr class="my-3">

        <div class="grid grid-cols-2 gap-3">
            <div class="col-span-1">
                <label for="cargoWeight">Вес груза</label>
                <input type="number" name="cargoWeight" v-model.number="cargoWeight"
                    class="focus:ring-indigo-500 focus:border-indigo-500 flex-1 w-full rounded-md sm:text-sm border-gray-300 border-2 p-1"
                >
            </div>

            <div class="col-span-1">
                <label for="distance">Расстояние</label>
                <input type="number" name="distance" v-model.number="distance"
                    class="focus:ring-indigo-500 focus:border-indigo-500 flex-1 w-full rounded-md sm:text-sm border-gray-300 border-2 p-1"
                >
            </div>
        </div>

        <hr class="my-3">
    </div>

    <calculation 
        :rules="rules"
        :axis-data="axisData"
        :dimension-data="dimensionData"
        :cargo-weight="cargoWeight"
        :distance="distance"
    />
</div>
</template>

<script>

import AxisData from "./axis-data"
import DimensionData from "./dimensions-data"
import Calculation from "./calculation"

export default {
    components: { AxisData, DimensionData, Calculation },

    props: {
        rules: {
            type: Object,
            default: () => ({
                permissibleLoads: {
                    1: {
                        title: "Одиночная ось",
                        weight: 11
                    },
                    2: {
                        title: "Сдвоенная ось",
                        weight: 16
                    },
                    3: {
                        title: "Строенная ось",
                        weight: 23
                    }
                },
                axisTypeDiff: 2.5,

                cargoWeightExcess: {
                    overLimitStep: 10,
                    overLimitPayment: 0.78,

                    gradiation: {
                        40: 0,
                        44: 0.1,
                        52: 0.2,
                        60: 0.27,
                    }
                },

                axisWeightExcess: {
                    overLimitStep: 5,
                    overLimitPayment: 0.15,

                    gradiation: {
                        5: 0.05,
                        10: 0.1,
                        20: 0.27
                    }
                },

                coefficient: {
                    gradiation: {
                        10: 2,
                        40: 3
                    },

                    higher: 5
                },

                dimensionExcess: 0.3,
            })
        }
    },

    data() {
        return {
            axisData: [],

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
            },

            cargoWeight: 0,
            distance: 0
        }
    },

    computed: {
        dataCorrect () {
            if (this.axisData.spacings.filter(el => !!el).length != this.axisData.spacings.length) return false
            if (this.axisData.weight.filter(el => !!el).length != this.axisData.weight.length) return false

            return true
        }
    },
}
</script>

<style>
    .print-only{
        display: none;
    }

    @media print {
        .no-print {
            display: none;
        }

        .print-only{
            display: block;
        }
}
</style>
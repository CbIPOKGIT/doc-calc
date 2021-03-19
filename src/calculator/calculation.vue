<template>
<div>
    <div class="overflow-hidden rounded-md bg-gray-800">
        <table class="table-auto border-green-800 calculation-table">
            <thead>
                <tr class="text-gray-50 font-semibold text-center text-xs">
                    <th class="p-2">№</th>
                    <th class="p-2" colspan="2">Фактические весовые и(или) габаритные параметры ТЗ</th>
                    <th class="p-2">Параметры на которые насчитывается плата за проезд</th>
                    <th class="p-2">Плата за проезд с превышением нормативных параметров</th>
                    <th class="p-2">Превышение параметров от норматива</th>
                </tr>
            </thead>

            <tbody class="bg-white">
                <tr>
                    <td class="text-center">1</td>
                    <td class="">{{ weightOverData.title }}</td>
                    <td class="text-center">{{ weightOverData.has }}т</td>
                    <td class="text-center">{{ weightOverData.over }}</td>
                    <td class="text-center">{{ weightOverData.payment }}</td>
                    <td class="text-center">{{ weightOverData.percent }}</td>
                </tr>

                <!-- Нагрузка на оси -->
                <tr>
                    <td class="text-center">2</td>
                    <td class="" colspan="5">Нагрузка на оси</td>
                </tr>

                <tr v-for="axis,indexAxis in axisOverData" :key="indexAxis + 'axis'">
                    <td class="text-center">
                        2.{{ axis.axis_count }}.{{ indexAxis + 1 }}
                    </td>
                    <td>{{ axis.title }}</td>
                    <td class="text-center">{{ axis.has }}</td>
                    <td class="text-center">{{ axis.over }}</td>
                    <td class="text-center">{{ axis.payment }}</td>
                    <td class="text-center">{{ axis.percent }}</td>
                </tr>

                <!-- Габариты -->
                <tr>
                    <td class="text-center">3</td>
                    <td class="" colspan="5">Габариты</td>
                </tr>

                <tr v-for="dim,indexDim in dimensionsOverData" :key="indexDim + 'dim'">
                    <td class="text-center">
                        3.{{ indexDim + 1 }}
                    </td>
                    <td>{{ dim.title }}</td>
                    <td class="text-center">{{ dim.has }}</td>
                    <td class="text-center">{{ dim.over }}</td>
                    <td class="text-center">{{ dim.payment }}</td>
                    <td class="text-center">{{ dim.percent }}</td>
                </tr>

                <!-- Итого -->
                <tr>
                    <td colspan="4"><div class="pl-3">Итого за 1 км проезда</div></td>
                    <td class="text-center">{{totalPayment}}</td>
                    <td />
                </tr>

                <!-- Итого -->
                <tr>
                    <td colspan="6">
                        <div class="px-3 block text-right font-semibold text-lg">
                            Итого: 
                            <span class="text-red-800">
                                {{ totalPrice }}
                            </span>
                            euro
                        </div>
                    </td>
                </tr>

            </tbody>
        </table>
    </div>
</div>
</template>

<script>

/**
 * {
 *      Заголовок,
 *      факт,
 *      over,
 *      плата за проезд,
 *      процент превышения
 * }
 */
const getEmptyData = () => ({
    title: null,
    has: null,
    over: null,
    payment: null,
    percent: null
})

const sortedObjectKeys = obj => Object.keys(obj)
                                    .map(el => parseInt(el))
                                    .sort((a,b) => a-b)

export default {
    props: ['rules', 'axisData', 'dimensionData', 'cargoWeight', 'distance'],

    computed: {
        weightOverData () {
            const data = getEmptyData()
            data.title = "Общая масса"
            data.has = this.cargoWeight

            const weights = sortedObjectKeys(this.rules.cargoWeightExcess.gradiation)

            if (this.cargoWeight <= weights[0]) {
                return data
            } else {
                data.over = Math.round( (this.cargoWeight - weights[0]) * 100  ) / 100
                data.percent = Math.round( data.over * 10000 / weights[0] ) / 100
            }

            //Вычисляем в каком диапазоне находится вес
            let gradiate = weights.filter(el => el >= this.cargoWeight).shift()

            gradiate = !gradiate ? [...weights].pop() : gradiate

            let payment = this.rules.cargoWeightExcess.gradiation[ gradiate ]

            if (gradiate < this.cargoWeight) {
                payment += Math.ceil( ( this.cargoWeight - gradiate ) / this.rules.cargoWeightExcess.overLimitStep )
                            * this.rules.cargoWeightExcess.overLimitPayment
            }

            data.payment = payment

            return data
        },

        axisOverData () {
            if (this.axisData.length == 0) {
                return []
            }

            return this.axisData.map(axis => this.recalculateAxisOverData( axis )).filter(data => !!data)
        },

        dimensionsOverData () {
            return Object.values(this.dimensionData).map(data => this.recalculateDimensionOverData(data))
        },

        allOverData () {
            return [this.weightOverData, ...this.dimensionsOverData, ...this.axisOverData]
        },

        totalPayment () {
            return Math.round( this.allOverData.reduce((acc, el) => acc + el.payment || 0, 0) * 100 ) / 100
        },

        koef () {
            const perc = this.allOverData.filter(el => el.percent > 0).map(el => el.percent)
                    .sort((a,b) => a-b).pop() || 0

            const gradiation = sortedObjectKeys(this.rules.coefficient.gradiation)

            if (perc == 0) {
                return 1
            }

            const gradiatiant = gradiation.filter(el => el >= perc).shift()

            if (!gradiatiant) {
                return this.rules.coefficient.higher
            } else {
                return this.rules.coefficient.gradiation[ gradiatiant ]
            }
        },

        totalPrice () {
            return Math.round(this.distance * this.koef * this.totalPayment * 100) / 100
        }
    },

    methods: {
        recalculateAxisOverData (axis) {
            if (axis == undefined) return

            const data = getEmptyData()
            data.title = this.generateAxisTitle(axis.axis_count)
            data.has = axis.total_weight
            data.axis_count = axis.axis_count

            if (axis.total_weight <= axis.permissible_weight) {
                return data
            }

            data.over = Math.round( (axis.total_weight - axis.permissible_weight) * 100 ) / 100
            data.percent = Math.round( data.over * 10000 / axis.permissible_weight ) / 100

            const gradiations = sortedObjectKeys(this.rules.axisWeightExcess.gradiation)

            let gradiate = gradiations.filter(el => el > data.percent).shift()
            gradiate = !gradiate ? [...gradiations].pop() : gradiate

            let payment = this.rules.axisWeightExcess.gradiation[ gradiate ]

            if (gradiate < data.percent) {
                payment += Math.ceil( (data.percent - gradiate) / this.rules.axisWeightExcess.overLimitStep )
                            * this.rules.axisWeightExcess.overLimitPayment
            }

            if (axis.axis_count > 2 && axis.single_tier) {
                payment *= 2
            }

            data.payment = Math.round( payment * 100 ) / 100

            return data
        },

        generateAxisTitle (axisCount) {
            const axis = this.rules.permissibleLoads[ axisCount ]

            return !axis ? "Мультиось" : axis.title
        },

        recalculateDimensionOverData (dimensionData) {
            const data = getEmptyData()
            data.title = dimensionData.title
            data.has = dimensionData.has

            if (dimensionData.has <= dimensionData.max) {
                return data
            }

            data.over = dimensionData.has - dimensionData.max
            data.payment = 0.3
            
            data.percent = Math.round( (dimensionData.has - dimensionData.max) * 10000 / dimensionData.max ) / 100

            return data
        }
    },
}
</script>

<style>
    .calculation-table td {
        border-color: #e5e7eb;
        border-width: 1px;
        padding: 1px 3px;
    }

    .calculation-table th {
        border-color: #fefeff;
        border-width: 1px;
    }
</style>
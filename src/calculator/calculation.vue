<template>
<div>
    <div v-if="!!rules.debug">
        Вес: {{ weightPayments }} <br>
        Ось: {{ axisPayments }} <br>
        Габариты: {{ dimensionPayments }} <br>
        Коэфициент: {{ coefficient }} <br>
    </div>
    
    <div>
        Сумма: {{ total }}
    </div>
</div>
</template>

<script>
export default {
    props: ['rules', 'axisData', 'dimensionData', 'cargoWeight', 'distance'],

    computed: {
        weightPayments () {
            const limits = Object.keys(this.rules.cargoWeightExcess.gradiation)
                                .map(el => parseInt(el)).sort((a,b) => a-b)

            if (this.cargoWeight < limits[0]) {
                return {
                    payment: 0,
                    over: 0
                }
            }

            let graduate = limits.filter(el => el >= this.cargoWeight).shift()

            const minLevel = limits[0]

            const over = this.cargoWeight * 100 / minLevel - 100

            if (!graduate) {
                graduate = limits.pop()

                const tax = this.rules.cargoWeightExcess.gradiation[ graduate ]

                const oversize = Math.ceil( (this.cargoWeight - graduate) / this.rules.cargoWeightExcess.overLimitStep )
                
                return {
                    payment: tax + oversize * this.rules.cargoWeightExcess.overLimitPayment,
                    over
                }
            }

            return {
                payment: this.rules.cargoWeightExcess.gradiation[ graduate ],
                over
            }
        },

        axisPayments () {
            const data = []

            for (let i = 0; i < this.axisData.weight.length; i++) {
                if (this.axisData.permissible_weight[i] == 0) continue
                if (this.axisData.permissible_weight[i] >= this.axisData.weight[i]) continue

                data.push({
                    permissible: this.axisData.permissible_weight[i],
                    weight: this.axisData.weight[i]
                })
            }

            if (!data.length) {
                return {
                    payment: 0,
                    over: 0
                }
            }

            const axis = data.sort((a,b) => a.weight / a.permissible - b.weight / b.permissible).pop()
            
            const over = axis.weight * 100 / axis.permissible - 100
            
            const limits = Object.keys(this.rules.axisWeightExcess.gradiation)
                                    .map(el => parseInt(el)).sort((a,b) => a-b)

            let graduate = limits.filter(el => el >= over).shift()

            if (!graduate) {
                graduate = limits.pop()

                const tax = this.rules.axisWeightExcess.gradiation[ graduate ]

                const oversize = Math.ceil( (over - graduate) / this.rules.axisWeightExcess.overLimitStep )
                
                return {
                    payment: tax + oversize * this.rules.axisWeightExcess.overLimitPayment,
                    over
                }
            }

            return {
                payment: this.rules.axisWeightExcess.gradiation[ graduate ],
                over
            }
        },

        dimensionPayments () {
            const oversizes = Object.keys(this.dimensionData.has)
                                .filter(d => this.dimensionData.max[d] > 0)
                                .map(d => ({
                                    oversize: this.dimensionData.has[d] - this.dimensionData.max[d],
                                    percent: this.dimensionData.has[d] * 100 / this.dimensionData.max[d] - 100
                                })).filter(d => d.oversize > 0 && d.percent > 0)

            return {
                payment: this.rules.dimensionExcess * oversizes.length,
                over: oversizes.map(el => el.percent).sort((a,b) => b-a).shift() || 0
            }
        },

        coefficient () {
            const highest = [this.weightPayments.over, this.axisPayments.over, this.dimensionPayments.over]
                            .filter(el => el > 0)
                            .sort((a,b) => a-b)
                            .pop()

            if (!highest) return 1

            const gradient = Object.keys(this.rules.coefficient.gradiation).map(el => parseInt(el))
                                .sort((a,b) => a-b)
                                .filter(el => el >= highest)
                                .shift()

            if (!gradient) return this.rules.coefficient.higher

            return this.rules.coefficient.gradiation[ gradient ]
        },

        total () {
            console.log()
            return (this.weightPayments.payment + this.axisPayments.payment + this.dimensionPayments.payment)
                    * this.coefficient * this.distance
        }
    },
}
</script>
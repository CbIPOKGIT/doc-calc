<template>
<div>
    <span class="axis-header text-xl font-semibold block">
        Данные по осям
    </span>

    <div class="flex justify-between items-center mb-2">
        <div>
            <div>Spacings</div>
            
            <div class="spacings-list pl-7">
                <div class="inline-block mr-1" v-for="(space, indexSpace) in axisData.spacings" :key="indexSpace">
                    <input type="number" v-model.number="axisData.spacings[indexSpace]"
                        class="focus:ring-indigo-500 focus:border-indigo-500 flex-1 block w-16 rounded-md sm:text-sm border-gray-300 border-2 p-1"
                        @input="updateValue"
                    >
                </div>
            </div>
        </div>

        <div>
            <button class="border border-transparent text-base font-medium rounded-full text-white bg-indigo-600 hover:bg-indigo-700 h-8 w-8" @click="addSpacing">+</button>

            <button class="border border-transparent text-base font-medium rounded-full text-white bg-red-600 hover:bg-red-700 h-8 w-8 ml-1" @click="removeSpacing">-</button>
        </div>
    </div>

    <div class="block weight-block">
        <div>Допустимая нагрузка на ось</div>
        <div class="inline-block mr-1" v-for="(weight, indexPWeight) in axisData.permissible_weight" :key="indexPWeight">
            <input type="number" :value="axisData.permissible_weight[indexPWeight]"
                class="focus:ring-indigo-500 focus:border-indigo-500 flex-1 block w-16 rounded-md sm:text-sm border-gray-300 border-2 p-1"
                readonly
            >
        </div>
    </div>

    <div class="block weight-block">
        <div>Фактическая нагрузка на ось</div>
        <div class="inline-block mr-1" v-for="(weight, indexWeight) in axisData.weight" :key="indexWeight">
            <input type="number" v-model.number="axisData.weight[indexWeight]"
                class="focus:ring-indigo-500 focus:border-indigo-500 flex-1 block w-16 rounded-md sm:text-sm border-gray-300 border-2 p-1"
            >
        </div>
    </div>
</div>
</template>

<script>
export default {
    props: ['value','rules'],

    data() {
        return {
            axisData: {
                spacings: [0],
                permissible_weight: [0,0],
                weight: [0,0]
            }
        }
    },

    methods: {
        addSpacing () {
            this.axisData.spacings.push(0)
            this.axisData.weight.push(0)
        },

        removeSpacing () {
            if (this.axisData.spacings.length > 1) {
                this.axisData.spacings.splice(-1,1)
                this.axisData.weight.splice(-1,1)
                this.recalculateWeights()
            }
        },

        updateValue () {
            this.recalculateWeights()

            this.$emit("input", this.axisData)
        },

        recalculateWeights () {
            const blocks = []
            let block = [0]

            for (let i = 0; i < this.axisData.spacings.length; i++) {
                if (this.axisData.spacings[i] > this.rules.axisTypeDiff) {
                    blocks.push([...block])
                    block = []
                }
                block.push(i + 1)
            }
            blocks.push(block)

            this.axisData.permissible_weight.splice(0)
            for (let i = 0; i < this.axisData.spacings.length + 1; i++) this.axisData.permissible_weight.push(0)

            for (const block of blocks) {
                for (const index of block) {
                    this.axisData.permissible_weight[index] = this.rules.permissibleLoads[ block.length ]
                }
            }
        }
    },
}
</script>
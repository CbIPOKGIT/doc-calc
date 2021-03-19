<template>
<div>
    <span class="axis-header text-xl font-semibold block">
        Данные по осям
    </span>

    <div class="flex justify-between items-center mb-2">
        <div>
            <h6 >Расстояние между осями</h6>
            
            <div class="spacings-list pl-7">
                <div class="inline-block mr-1" v-for="(space, indexSpace) in spacings" :key="indexSpace">
                    <input type="number" v-model.number="spacings[indexSpace]"
                        :class="inputClass" class="w-16 "
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

    <div v-if="blocks.length > 0">
        <h5 class="font-semibold">Допустимая нагрузка</h5>
        <div class="flex gap-x-3">
            <div v-for="pWeight,indexP in permissible_weights" :key="indexP" class="flex-1">
                <div class="flex justify-between">
                    <h6 >{{ axisTitle(indexP) }}</h6>

                    <div class="flex items-center" v-if="isTrippleAxis(indexP)">
                        <input type="checkbox" class="h-4 w-4 text-indigo-600 focus:ring-indigo-500 border-gray-300 rounded" v-model="singleTiers[indexP]" @change="emitUpdate()">
                        <label class="ml-2 block text-sm text-gray-900" v-text="`Одиночная покрышка`" />
                    </div>
                </div>

                <input type="number" v-model.number="permissible_weights[indexP]"
                    :class="inputClass" class="w-full"
                    @input="emitUpdate()"
                >
            </div>
        </div>

        <h5 class="font-semibold">Фактическая нагрузка</h5>
        <div class="flex gap-x-3">
            <div v-for="rWeight,indexR in real_weights" :key="indexR" class="flex-1" 
                :class="indexR < real_weights.length - 1 ? 'with-border-right' : ''"
            >
                <div class="flex gap-x-2">
                    <div class="flex-grow" v-for="w,indexW in rWeight" :key="indexW">
                        <input type="number" v-model.number="real_weights[indexR][indexW]"
                            :class="inputClass" class="w-full"
                            @input="emitUpdate()"
                        >
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
</template>

<script>
export default {
    props: ['value','rules'],

    data() {
        return {
            blocks: [],
            spacings: [],
            permissible_weights: [],
            real_weights: [],

            singleTiers: []
        }
    },

    computed: {
        inputClass () {
            return `focus:ring-indigo-500 focus:border-indigo-500 flex-1 block rounded-md sm:text-sm border-gray-300 border-2 p-1`
        },

        finalValue () {
            const values = []

            for (let i = 0; i < this.blocks.length; i++) {
                const block = this.blocks[i]

                values.push({
                    axis_count: block.length,
                    single_tier: block.length >= 3 ? this.singleTiers[i] : false,
                    total_weight: this.real_weights[i].reduce((acc, value) => acc + value),
                    permissible_weight: this.permissible_weights[i]
                })
            }

            return values.sort((a,b) => a.axis_count - b.axis_count)
        }
    },

    methods: {
        axisTitle (axisIndex) {
            const section = this.rules.permissibleLoads[ this.blocks[axisIndex].length ]

            return !!section
                    ? section.title
                    : "Мультиось"
        },

        isTrippleAxis (axisIndex) {
            return this.blocks[axisIndex].length >= 3
        },

        emitUpdate () {
            this.$emit("input", this.finalValue)
        },

        addSpacing () {
            this.spacings.push(1)
            this.recalculateAxis()
            this.recalculateWeightsBlocks()

            this.emitUpdate()
        },

        removeSpacing () {
            this.spacings.splice(-1,1)
            this.recalculateAxis()
            this.recalculateWeightsBlocks()

            this.emitUpdate()
        },

        updateValue () {
            this.recalculateAxis()
            this.recalculateWeightsBlocks()

            this.emitUpdate()
        },

        recalculateWeightsBlocks () {
            this.permissible_weights.splice(0)
            this.blocks.forEach(block => {
                const section = this.rules.permissibleLoads[ block.length ]
                this.permissible_weights.push( !!section ? section.weight : 0 )
            })

            if (this.real_weights.length > this.blocks.length) {
                const diff = this.blocks.length - this.real_weights.length
                this.real_weights.splice(diff, Math.abs(diff))
            } else {
                while (this.real_weights.length < this.blocks.length) {
                    this.real_weights.push([])
                }
            }

            for (let i = 0; i < this.real_weights.length; i++) {
                const el = this.real_weights[i]
                const block = this.blocks[i]

                if (el.length > block.length) {
                    const diff = block.length - el.length
                    el.splice(diff, Math.abs(diff))
                } else {
                    while (el.length < block.length) {
                        el.push(0)
                    }
                }
            }

            for (let i = 0; i < this.blocks.length; i++) {
                if (this.singleTiers.length < i + 1) {
                    this.singleTiers.push(false)
                }
            }
            this.singleTiers.splice(this.blocks.length)
        },

        recalculateAxis () {
            this.blocks.splice(0)

            if (this.spacings.length > 0) {
                let block = [0]

                for (let i = 0; i < this.spacings.length; i++) {
                    if (this.spacings[i] > this.rules.axisTypeDiff) {
                        this.blocks.push([...block])
                        block = []
                    }
                    block.push(i + 1)
                }
                this.blocks.push(block)
            }
            
        }
    },
}
</script>

<style>
    .with-border-right {
        position: relative;
    }

    .with-border-right::after {
        content: "";
        position: absolute;
        top: 0;
        left: 100%;
        width: 5px;
        height: 100%;
        transform: translateX(4px);
        background-color: rgba(184, 184, 184, 0.2);
        border-radius: 2px;
    }
</style>
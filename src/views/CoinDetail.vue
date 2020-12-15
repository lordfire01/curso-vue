<template>
     <div class="flex-col my-40">
         <div class="flex justify-center">
             <bounce-loader :loading="isLoading" :color="'#68d391'" :size="100" />
         </div>
         <template v-if="!isLoading">
             <div class="flex flex-col sm:flex-row justify-around items-center">
                 <div class="flex flex-col items-center">
                     <img :src="`https://static.coincap.io/assets/icons/${row.symbol.toLowerCase()}@2x.png`" :alt="row.name" class="w-20 h-20 mr-5">
                     <h1 class="text-5xl">
                         {{ row.name }}
                         <small class="sm:mr-2 text-gray-500">{{ row.symbol }}</small>
                     </h1>
                 </div>

                 <div class="my10 flex flex-col">
                     <ul>
                         <li class="flex justify-between">
                             <b class="text-gray-600 mr-10 uppercase">
                                 Ranking
                             </b>
                             <span>{{ row.rank }}</span>
                         </li>
                         <li class="flex justify-between">
                             <b class="text-gray-600 mr-10 uppercase">
                                 Precio Actual
                             </b>
                             <span>{{ row.priceUsd | dollar }}</span>
                         </li>
                         <li class="flex justify-between">
                             <b class="text-gray-600 mr-10 uppercase">
                                 Precio más bajo
                             </b>
                             <span>{{ min | dollar }}</span>
                         </li>
                         <li class="flex justify-between">
                             <b class="text-gray-600 mr-10 uppercase">
                                 Precio más alto
                             </b>
                             <span>{{ max | dollar }}</span>
                         </li>
                         <li class="flex justify-between">
                             <b class="text-gray-600 mr-10 uppercase">
                                 Precio Promedio
                             </b>
                             <span>{{ avg | dollar }}</span>
                         </li>
                         <li class="flex justify-between">
                             <b class="text-gray-600 mr-10 uppercase">
                                 Variacion 24hrs
                             </b>
                             <span>{{ row.changePercent24Hr | percent }}</span>
                         </li>
                     </ul>
                 </div>

                 <div class="my-10 sm:mt-0 flex flex-col justify-center text-center">
                     <button 
                        @click="toggleConverter"
                        class="bg-green-500 hover:bg-green-700 text-white font-bold py-2 px-4 rounded"
                     >{{ fromUsd ? `USD a ${row.symbol}` : `${row.symbol} a USD`}}</button>

                     <div class="flex flex-row my-5">
                         <label for="convertValue" class="w-full">
                             <input 
                                v-model="convertValue"
                                type="number" 
                                id="convertValue" 
                                class="text-center bg-white focus:outline-none focus:shadow-outline boreder"
                                :placeholder="`Valor en ${fromUsd ? 'USD' : row.symbol}`"
                             >
                         </label>
                     </div>
                     <span class="text-xl">{{ convertResult }} {{ fromUsd ? row.symbol : 'USD' }}</span>
                 </div>
             </div>

             <line-chart class="my-10"
                :colors="['orange ']" 
                :min="min"
                :max="max"
                :data="history.map(h => [h.date, parseFloat(h.priceUsd).toFixed(2)])"
             />

             <h3 class="text-xl my-10">Mejores Ofertas de Cambio</h3>
             <table>
                 <tr v-for="m in markets" :key="`${m.exchangeId}-${m.priceUsd}`" class="border-b">
                     <td>
                         <b>{{ m.exchangeId }}</b>
                     </td>
                     <td>{{ m.priceUsd | dollar }}</td>
                     <td>{{ m.baseSymbol }} / {{ m.quoteSymbol }}</td>
                     <td>
                         <botones 
                            :is-loading="m.isLoading || false" 
                            v-if="!m.url" 
                            @custom-click="getWebSite(m)"
                         >
                             <slot>Obtener Link</slot>
                         </botones>
                         <a v-else class="hover:underline text-green-600" target="_blank">{{ m.url }}</a>
                     </td>
                 </tr>
             </table>
         </template>
     </div>
</template>

<script>
import api from '@/api'
import Botones from '@/components/Botones'
export default {
    name: "CoinDetail",

    components: {Botones},

    data () {
        return {
            isLoading: false,
            row: {},
            history: [],
            markets: [],
            fromUsd: true,
            convertValue: null
        }
    },

    created() {
        this.getCoin()
    },

    computed: {
        convertResult() {
            if(!this.convertValue) {
                return 0;
            }

            const result = this.fromUsd ? this.convertValue / this.row.priceUsd : this.convertValue * this.row.priceUsd

            return result.toFixed(4)
        },
        min() {
            return Math.min.apply(Math,
                this.history.map(h => parseFloat(h.priceUsd).toFixed(2))
            )
        },
        max() {
            return Math.max.apply(Math,
                this.history.map(h => parseFloat(h.priceUsd).toFixed(2))
            )
        },
        avg() {
            return Math.abs.apply(Math,
                this.history.map(h => parseFloat(h.priceUsd).toFixed(2))
            )
        }
    },

    watch: {
        $route () {
            this.getCoin()
        }
    },
    
    methods: {
        getCoin() {
            const id = this.$route.params.id
            this.isLoading = true

            Promise.all([
                api.getRow(id),
                api.rowHistory(id),
                api.getMarkets(id)
            ])
            .then(
                ([row, history, markets]) => {
                    this.row = row,
                    this.history = history
                    this.markets = markets
                }
            )
            .finally(() => this.isLoading = false)
        },
        getWebSite(exchange) {
            this.$set(exchange, 'isLoading', true)

            return api.getExchange(exchange.exchangeId).then(res => {
                this.$set(exchange, 'url', res.exchangeUrl)

            }).finally(() => {
                this.$set(exchange, 'isLoading', false)
            })
        },
        toggleConverter() {
            this.fromUsd = !this.fromUsd
        }
    }
}
</script>

<style scoped>
td {
    padding: 10px;
    text-align: center;
}
</style>
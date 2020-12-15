<template>
    <table>
        <thead>
            <tr class="bg-gray-100 border-b-2 border-gray-400">
                <th></th>
                <th :class="{up: this.sortOrder == 1, down: this.sortOrder == -1}">
                    <span class="underline cursor-pointer" @click="changeSortOrder">Ranking</span>
                </th>
                <th>Nombre</th>
                <th>Precio</th>
                <th>Cap. de Mercado</th>
                <th>Variacion 24hrs</th>
                <th class="hidden sm:block">
                    <input
                        id="filter"
                        class="bg-gray-100 focus:outline-none border-b border-gray-400 py-2 px-4 block"
                        placeholder="Buscar..."
                        type="text"
                        v-model="filter"
                    />
                </th>
            </tr>
        </thead>
        <tbody>
            <tr 
            v-for="r in filtered"
            :key="r.id"
            class="border-b border-gray-200 hover:bg-gray-100 hover:bg-orange-100">
                <td>
                    <img 
                    class="w-6 h-6"
                    :src="`https://static.coincap.io/assets/icons/${r.symbol.toLowerCase()}@2x.png`" :alt="r.name">
                </td>
                <td>
                    <b># {{ r.rank }}</b>
                </td>
                <td>
                    <router-link 
                    class="hover:underline text-green-600"
                    :to="{name: 'coin-detail', params: {id: r.id}}">
                        {{ r.name }}
                    </router-link>
                    <small class="ml-1 text-green-500">{{ r.symbol }}</small>
                </td>
                <td>{{ r.priceUsd | dollar }}</td>
                <td>{{ r.marketCapUsd | dollar }}</td>
                <td :class="r.changePercent24Hr.includes('-') ? 'text-red-600' : 'text-green-600'">
                    {{ r.changePercent24Hr | percent }}
                </td>
                <td class="hidden sm:block">
                    <botones @custom-click="goToCoin(r.id)">
                        <span>Detalle</span>
                    </botones>
                </td>
            </tr>
        </tbody>
    </table>
</template>

<script>
import Botones from '@/components/Botones'
export default {
    name: 'MyTable',

    components: {Botones},
    
    data() {
        return {
            filter: '',
            sortOrder: 1
        }
    },

    props: {
        registros: {
            type: Array,
            default: () => []
        }
    },

    computed: {
        filtered() {

            const altOrder = this.sortOrder == 1 ? -1 : 1

            return this.registros.filter(r => 
                r.symbol.toLowerCase().includes(this.filter.toLowerCase()) ||
                r.name.toLowerCase().includes(this.filter.toLowerCase())
            ).sort((a, b) => {
                if(parseInt(a.rank) > parseInt(b.rank)) {
                    return this.sortOrder
                }

                return altOrder
            })
        }
    },

    methods: {
        goToCoin(coin) {
            this.$router.push({ name:"coin-detail", params: { id: coin } })
        },
        changeSortOrder() {
            this.sortOrder = this.sortOrder == 1 ? -1 : 1
        }
    }
    
}
</script>

<style scoped>
    .up::before {
        content: '\21D1';
    }

    .down::before {
        content: '\21D3';
    }

    td {
        padding: 20px 0px;
        font-size: 0.6rem;
        text-align: center;
    }

    th {
        padding: 5px;
        font-size: 0.6rem;
    }

    @media (min-width: 640px) {
        td, th {
            padding: 20px;
            font-size: 1rem;
        }

        th {
            padding: 12px;
        }
    }
</style>
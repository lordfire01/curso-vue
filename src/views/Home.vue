<template>
    <div>
        <bounce-loader :loading="isLoading" :color="'#68d391'" :size="100" />
        <my-table v-if="!isLoading" v-bind:registros="registros" />
    </div>
</template>

<script>
import api from '@/api'
import MyTable from '@/components/MyTable';

export default {
    name: "Home",
    components: { MyTable },
    data () {
        return {
            isLoading: false,
            registros: []
        }
    },

    created () {
        this.isLoading = true;
        api.getData()
        .then(registros => (this.registros = registros))
        .finally(() => this.isLoading = false)
    }
}
</script> 
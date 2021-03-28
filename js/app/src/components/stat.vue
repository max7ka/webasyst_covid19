<template>
    <div class="block fields">
        <div class="field-group">
            <waField name="Последнее обновление" :value="formatDate(data.updated)"/>
            <waField name="Сделано тестов" :value="formatNumber(data.tests)"/>
            <waField name="Случаи" :value="formatNumber(data.cases)"/>
            <waField name="Умерло" :value="formatNumber(data.deaths)"/>
            <waField name="Состоят на лечении" :value="formatNumber(data.active)"/>
            <waField name="Выздоровевшие" :value="formatNumber(data.recovered)"/>
            <waField name="Сегодня умерло" :value="formatNumber(data.todayDeaths)"/>
            <waField name="Сегодня выздоровело" :value="formatNumber(data.todayRecovered)"/>
        </div>
    </div>  
</template>

<script>
import waField from './waField.vue';
export default {
    name: 'stat',
    components:{
        waField,
    },
    props: ['data'],
    methods:{
        formatDate(val){
            return new Date(val).toLocaleString("ru", {day:'numeric',month:'long',year:'numeric',hour:'numeric',minute:'numeric'})
        },
        formatNumber(val){
            return new Intl.NumberFormat('ru-RU').format(val)
        },        
    }
}
</script>

<style>

</style>
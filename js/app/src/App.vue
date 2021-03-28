<template>
  <div class="block main-content">

    <div class="flex flex-end" >
      <a href="#" class="button green" style="height:20px" @click="updateData">Обновить</a>
    </div>

    <transition name="fade">
    <div class="block" v-if="ChatAllDayAgo.stat.error==false && ChatContinent.error==false">
      <ul class="tabs">
          <li v-for="(item,index) in tabs.items" :key="`tab${index}`" :class="{selected: index==tabs.selectedIndex}">
            <a href="#" @click="tabClick(tabs,index)">{{item.name}}</a>
          </li>
      </ul>
      <div class="tab-content">
          <transition name="slide-fade">
          <div class="block flex row" v-if="tabs.selectedIndex==0" style="position:absolute">
            <div class="flex column">
              <div style="text-align: center;padding:10px"><strong>Статистика за один день</strong></div>
              <div class="flex row" style="height:70vh;width:95vw">
                <stat :data="ChatAllDayAgo.stat" class="grow1"/>
                <apexchart type="donut" class="grow1" width="100%" height="100%" :options="ChatAllDayAgo.chartOptions" :series="ChatAllDayAgo.series"></apexchart>
              </div>
            </div>
          </div>
          </transition>
          <transition name="slide-fade">
          <div  class="block flex column" v-if="tabs.selectedIndex==1" style="position:absolute">
            <div style="text-align: center;padding:10px"><strong>Число случаев за день по континентам</strong></div>
            <div class="block flex row" style="width:95vw;">
              <apexchart ref="refChatContinent" class="grow1" type="treemap" width="100%" height="100%" :options="ChatContinent.chartOptions" :series="ChatContinent.series"></apexchart>
              <div class="block flex column grow1">
                <div>{{ChatContinent.continent}}</div>
                <stat :data="ChatContinent.stat"/>
                <div class="">Страны</div>
                <div class="countries small">
                  <span style="padding:10px 10px 0 0" v-for="(item,inde) in ChatContinent.countries" :key="`co${inde}`">{{item}}</span>
                </div>
              </div>
            </div>
          </div>
          </transition>
      </div>
    </div>
    </transition>

  </div>
</template>

<script>
import stat from './components/stat.vue';

export default {
  components: {
    stat,
  },
  created(){
    this.updateData()
  },
  data(){
    return {
      tabs : {
        items: [
          {name:'Диаграмма'},
          {name:'Карта мира'},
        ],
        selectedIndex:0,
      },
      ChatAllDayAgo: {
        stat:{
          updated: -1,
          cases: -1,//случаи
          deaths: -1,
          active: -1,//состоят на лечении
          recovered: -1,//выздоровевшие
          todayDeaths: -1,
          todayRecovered: -1,
          tests: -1,//сделано тестов
          error:true
        },
        series: [1,1,1,1],
        chartOptions: {
          labels: [/*'Сделано тестов', */'Случаи', 'Умерло', 'Состоят на лечении', 'Выздоровевшие'/*,'Сегодня умерло','Сегодня выздоровело'*/],
        },
      },

      ChatContinent: {
        error: true,
        continent: '',
        stat:{},
        countries: [],
        data: {},
        series: [
          {
            data: [
              {x: '1',y: 1},
              {x: '2',y: 1},
              {x: '3',y: 1},
              {x: '4',y: 1},
              {x: '5',y: 1},
              {x: '6',y: 1},
            ]
          }
        ],
        chartOptions: {
          legend: {
            show: false
          },
          chart: {
            height: 350,
            type: 'treemap',
            events: {
              dataPointMouseEnter: this.ChatContinent_dataPointMouseEnter
            }              
          },
          /*title: {
            text: 'Число случаев за день по континентам',
            align: 'center'
          },*/
          colors: [
            '#3B93A5',
            '#F7B844',
            '#ADD8C7',
            '#EC3C65',
            '#CDD7B6',
            '#C1F666',
          ],
          plotOptions: {
            treemap: {
              distributed: true,
              enableShades: false
            }
          }
        },     
      }
    }
  },
  methods:{
    tabClick(tabs,index){
      tabs.selectedIndex = index
    },
    async updateData(){
      try {
        const response = await this.axios.get('https://disease.sh/v3/covid-19/all')
        this.ChatAllDayAgo.stat = this.createStat(response.data)
        this.ChatAllDayAgo.series=
        [
          //this.ChatAllDayAgo.stat.tests,
          this.ChatAllDayAgo.stat.cases,
          this.ChatAllDayAgo.stat.deaths,
          this.ChatAllDayAgo.stat.active,
          this.ChatAllDayAgo.stat.recovered,
          //this.ChatAllDayAgo.stat.todayDeaths,
          //this.ChatAllDayAgo.stat.todayRecovered,
        ]
        this.ChatAllDayAgo.stat.error = false
      } catch (error) {
        this.ChatAllDayAgo.stat.error = true
      }

      try {
        const responce2 = await this.axios.get('https://disease.sh/v3/covid-19/continents')
        this.ChatContinent.data = responce2.data
        //this.ChatContinent.stat = responce2.data
        var data=[]
        this.ChatContinent.data.forEach(item => {
          data.push({x: item.continent,y:item.cases})
        });
        this.ChatContinent.stat = this.createStat(this.ChatContinent.data[0])
        this.ChatContinent.series = [{data:data}]
        this.ChatContinent.error = false
        this.ChatContinent.countries = this.ChatContinent.data[0].countries
        this.ChatContinent.continent = this.ChatContinent.data[0].continent
      } catch (error) {
        this.ChatContinent.error = true
      }
    },
    ChatContinent_dataPointMouseEnter (event, chartContext, config) {
      //console.log(config.dataPointIndex);
      this.ChatContinent.stat = this.createStat(this.ChatContinent.data[config.dataPointIndex])
      this.ChatContinent.countries = this.ChatContinent.data[config.dataPointIndex].countries
      this.ChatContinent.continent = this.ChatContinent.data[config.dataPointIndex].continent
    },
    createStat(resp){
      return {
        updated: resp.updated,
        cases: resp.cases,
        deaths: resp.deaths,
        active: resp.active,
        recovered: resp.recovered,
        todayDeaths: resp.todayDeaths,
        todayRecovered: resp.todayRecovered,
        tests: resp.tests,
      }      
    }
  }
}
</script>

<style lang="scss" scoped>
  .flex{
    display: flex !important;
  }
  .flex-end {
    justify-content: flex-end;
  }  
  .grow1 {
    flex-grow: 1;
  }
  .wrap {
    display: flex !important;
    flex-wrap: wrap;
  }   
  .row{
    flex-direction: row
  }
  .column{
    flex-direction: column
  }  

  .countries{
    display: flex;
    flex-direction: row;
    flex-wrap: wrap;
    width: 40vw;
  }
  
  .label{
    width: 200px;
  }

  .tab-content{
    background-color: rgba(254, 254, 254, 0.6);
    height:90vh;
  }
  ul.tabs li{
    background-color: rgba(254, 254, 254, 0.8);
  }
  .main-content{
    background: rgb(228,245,252); /* Old browsers */
    background: -moz-linear-gradient(-45deg,  rgba(228,245,252,1) 0%, rgba(191,232,249,1) 37%, rgba(159,216,239,1) 61%, rgba(42,176,237,1) 100%); /* FF3.6-15 */
    background: -webkit-linear-gradient(-45deg,  rgba(228,245,252,1) 0%,rgba(191,232,249,1) 37%,rgba(159,216,239,1) 61%,rgba(42,176,237,1) 100%); /* Chrome10-25,Safari5.1-6 */
    background: linear-gradient(135deg,  rgba(228,245,252,1) 0%,rgba(191,232,249,1) 37%,rgba(159,216,239,1) 61%,rgba(42,176,237,1) 100%); /* W3C, IE10+, FF16+, Chrome26+, Opera12+, Safari7+ */
    filter: progid:DXImageTransform.Microsoft.gradient( startColorstr='#e4f5fc', endColorstr='#2ab0ed',GradientType=1 ); /* IE6-9 fallback on horizontal gradient */    
    height:120vh
  }

  .fade-enter-active, .fade-leave-active {
    transition: all .4s ease;
  }
  .fade-enter, .fade-leave-to {
    transform: translateX(20px);
    opacity: 0;
  }

  .slide-fade-enter-active {
    transition: all .8s ease;
  }
  .slide-fade-leave-active {
    transition: all .3s cubic-bezier(1.0, 0.5, 0.8, 1.0);
  }
  .slide-fade-enter, .slide-fade-leave-to{
    transform: translateX(300px);
    opacity: 0;
  }  
</style>

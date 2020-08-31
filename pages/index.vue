<template>
  <div>   
    <v-container>
      <h1 class='text-center'>Отчет «Источники, сводка». Посетители из Санкт-Петербурга</h1>
      <canvas ref='canvas' class="mt-5"></canvas>
      <div class="mt-15 text-center">
        <div class="mt-5 mb-5">
          <v-select 
            v-model='input'  
                  :items="metrics"
                  item-text="text"
                  item-value="name"
                  dense
                  outlined
                  color='deep-purple accent-5'
                  @input='from(input)'
                ></v-select>
        </div>
        <h2 class="d-block mt-5 mb-5 text-left" v-if='input.length <=0'>{{'Из поисковых систем'}}</h2>
      </div>
      <v-row class='mt-5 text-center'>
        <v-col :cols='my_colz' v-for='item,index in defa'>
          <div v-if='index == 0'>
            <p style='color:#ffcc00;'>Визиты</p>
            <p>{{item}}</p>
          </div>
          <div v-if='index == 1'>
            <p style='color:#ffcc00;'>Посетители</p>
            <p>{{item}}</p>
          </div>
          <div v-if='index == 2'>
            <p style='color:#ffcc00;'>Отказы</p>
            <p>{{item}}</p>
          </div>
          <div v-if='index == 3'>
            <p style='color:#ffcc00;'>Глубина просмотра</p>
            <p>{{item}}</p>
          </div>
          <div v-if='index == 4'>
            <p style='color:#ffcc00;'>Время на сайте</p>
            <p>{{item}}</p>
          </div>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>
<script>
import {Pie} from 'vue-chartjs'
  export default{
    extends: Pie,
    data(){
      return{
        input:[],
        metrics:[{name:'from_search',text:'Из поисковых систем'},{name:'from_links',text:'Переходы по ссылкам на сайтах'},
        {name:'from_zahodi',text:'Прямые заходы'}],
        from_links:[],
        from_search:[],
        from_zahodi:[],

        defa:[],

        my_colz:2.4,
      }
    },
    async mounted(){
      if(window.innerWidth <400){
        this.my_colz = 12
      }
      //ЗАПРОС//
      var f = await fetch('https://api-metrika.yandex.net/stat/v1/data?preset=sources_summary&filters=ym%3As%3AregionCityName%3D%3D%27%D0%A1%D0%B0%D0%BD%D0%BA%D1%82-%D0%9F%D0%B5%D1%82%D0%B5%D1%80%D0%B1%D1%83%D1%80%D0%B3%27&id=44147844&lang=ru').then((res)=>{
        return res.json()
      }, (err)=>{
        console.log('Ошибка в запросе')
      })
      //console.log(f)

      //показатели из разных источников (Переходы из выдачи гугл,яндекс, по ссылкам и т.д)//
      var my_data = []
      for(var i=0;i<f.data.length;i++){
        let new_obj = {} //на каждой итерации создаем новый объект и в него кладем ключ со значением
          new_obj[f.data[i].dimensions[0].name /* + " " + f.data[i].dimensions[1].name */] = f.data[i].metrics
        my_data.push(new_obj)//пушим этот объект в массив
      }

      //для объединения некоторых показателей, например по ссылками с разных сайтов объединяем в один объект //
      var double_metrics_links = []
      var double_metrics_search_pages= []
      var pryamie_zahodi = []

      for(var q=0;q<my_data.length;q++){

        if('Переходы по ссылкам на сайтах' in my_data[q]){
          double_metrics_links.push(my_data[q])
        }
        if('Переходы из поисковых систем' in my_data[q]){
          double_metrics_search_pages.push(my_data[q])
        }
        if('Прямые заходы' in my_data[q]){
          pryamie_zahodi.push(my_data[q])
        }
      }

      // VUE переменные для всей статистики//
      this.from_search = this.cicle(double_metrics_search_pages)
      this.from_links = this.cicle(double_metrics_links)
      this.from_zahodi = this.cicle(pryamie_zahodi)

      this.defa = this.from_search 
      //console.log(double_metrics_one)
      console.log(this.cicle(pryamie_zahodi))
      //console.log(this.cicle(double_metrics_two))

      console.log(my_data)

      //ГРАФИК //
      this.renderChart({labels: /*['Визиты', ' Посетители', 'Отказы', 'Глубина просмотра', ' Время на сайте']*/['Переходы из поисковых систем', 'Переходы по ссылкам на сайтах', 'Прямые заходы'],
              datasets: [{
                  label: '# of Votes',
                  data: [this.cicle(double_metrics_search_pages)[1],this.cicle(double_metrics_links)[1],this.cicle(pryamie_zahodi)[1]],//f.totals,// массив данных равный по кол-ву элементов полям labels
                  backgroundColor: [
                      'rgba(255, 99, 132, 0.5)',
                      'rgba(54, 162, 235, 0.5)',
                       'rgba(255, 206, 86, 0.5)',
                      // 'rgba(75, 192, 192, 0.5)',
                      // 'rgba(153, 102, 255, 0.5)',
                  ],
                  borderColor: [
                      'rgba(255, 99, 132, 1)',
                      'rgba(54, 162, 235, 1)',
                       'rgba(255, 206, 86, 1)',
                      // 'rgba(75, 192, 192, 1)',
                      // 'rgba(153, 102, 255, 1)',
                  ],
                  borderWidth: 1
              }]
          })
    },
    methods:{
      cicle(datas,counter){//функция для сложения одинаковых метрик в массивах
        var base = Object.values(datas[0])[0] //берем первый элемент что бы к нему прибавлять остальные

        for(var counter=0; counter<datas.length;counter++){//сначала перебираем переданный массив с объектами
          //console.log(Object.values(datas[counter])[0])

          // через Object.values получаем значения каждого объекта
          Object.values(datas[counter])[0].forEach(function(item,index){
            if(counter != 0){//так как значение первого объекта у нас взято за основу, его пропускаем
              
              base[index] += item //т.к. индексы одинаковые-соответсвуют проходимся по каждому и складываем
              
            }
          })  
          
        }

        var res = []
        base.forEach(function(item,index){//для выведения среднего арифмитического для показателей которые не должны превышать более 100(например отказы)
          if(index <=1){//первые два значеня пропускаем, они нам не нужны
            res.push(item)
          }else{// в остальных выводим среднее арифметическое
            item = item / datas.length
            res.push(item)
          }
        })
        return res
      },
      from(data){
        if(data == 'from_links'){
          this.defa = this.from_links
        }
        if(data == 'from_search'){
          this.defa = this.from_search
        }
        if(data == 'from_zahodi'){
          this.defa = this.from_zahodi
        }
      }
    }
  };
</script>
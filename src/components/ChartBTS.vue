<script setup>
import { ref, onMounted } from "vue";
import io from "socket.io-client"
import VueApexCharts from "vue3-apexcharts";
import dayjs from "dayjs";

class Queue {
  constructor() {
    this._arr = [];
  }
  length() {
    return this._arr.length;
  }
  enqueue(item) {
    this._arr.push(item);
  }
  dequeue() {
    return this._arr.shift();
  }
  get_arr() {
    return this._arr;
  }
}

const props = defineProps({
  msg: String,
})

const times = ref([]);
const temperatures = ref([]);
const humidities = ref([]);
const pressures = ref([]);

const velocity = ref([]);
const options1 = ref({});
const series1 = ref([]);
const options2 = ref({});
const series2 = ref([]);
// const options3 = ref({});
// const series3 = ref([]);
const chardata_q = new Queue();

const socket = io("http://" + import.meta.env.VITE_HOST + ":" + import.meta.env.VITE_BACKEND_PORT);

//onMounted(()=>{//업데이트 될 때마다
socket.on("kfc", (arg) => {
  times.value = arg.map((x) => dayjs(x.time).format("HH:mm:ss"));
  temperatures.value = arg.map((x) => x.temp);
  humidities.value = arg.map((x) => x.humid);
  pressures.value = arg.map((x) => x.pressure);
 
  velocity.value = arg.map((x)=>x.velo);

  chardata_q.enqueue({
    time: times.value[0],
    temp: temperatures.value[0],
    humi: humidities.value[0],
    pres: pressures.value[0]
  })

  if (chardata_q.length() > 15)
    chardata_q.dequeue()
  console.log(chardata_q)
  console.log(chardata_q.length())
  console.log(chardata_q.get_arr())
  //console.log(chardata_q.time)
  const timeArray = chardata_q.get_arr().map(item => item.time);
  const temper = chardata_q.get_arr().map(item => item.temp);
  const hum = chardata_q.get_arr().map(item => item.humi);
  const press = chardata_q.get_arr().map(item => item.pres);
  console.log(timeArray);

  options1.value = {
    xaxis: {
      categories: timeArray,
      //categories: times.value,
    },
    title:{
      text: "온도 습도 대기압",
      align: 'center',
      style:{
        fontWeight:'bold',
        fontSize: '20px'
      }
    }
  };

  series1.value = [
    // {
    //   name: "온도",
    //   //data: temperatures.value,
    //   data: temper,
    // },
    {
      name: "습도",
      //data: humidities.value,
      data: hum,
    },
    // {
    //   name: "대기압",
    //   //data: pressures.value,
    //   data: press,
    // },
  ];

//
  options2.value = {
    //series: velocity.value,
    series: velocity.value.map(value => value * 200),//
    //series: [50],
    chart: {
      height: 350,
      type: 'radialBar',
      offsetY: -10
    },
    title:{
      text: "속도",
      align: 'center',
      style:{
        fontWeight:'bold',
        fontSize: '20px'
      }
    },
    plotOptions: {
      radialBar: {
        startAngle: -135,
        endAngle: 135,
        dataLabels: {
          name: {
            pontSize: '16px',
            color: undefined,
            offsetY: 120
          },
          value: {
            offsetY: 76, 
            fontSize: '22px',
            color: undefined,
            formatter: function (val) {
              return val + "m/s";//
            }
          }
        }
      }
    },

    fill: {
      type: 'gradient',
      gradient: {
        shade: 'dark',
        shadeIntensity: 0.15,
        inverseColors: false,
        opacityFrom: 1,
        opacityTo: 1,
        stops: [0, 50, 65, 91]
      },
    },
    stroke: {
      dashArray: 4
    },
    labels: ['Velocity'],
  };

  series2.value = velocity.value;//속도 값
  //

  // options3.value = {
  //   chart: {
  //     height: 350,
  //     type: 'radar',
  //   },
  //   title: {
  //     text: 'Basic Radar Chart'
  //   },
  //   xaxis: {
  //     categories: ['January', 'February', 'March', 'April', 'May', 'June']
  //   },
  // };

  // series3.value = [{
  //   name: 'Series 1',
  //   data: [80, 50, 30, 40, 100, 20],
  // }];

  //

});
//});

socket.emit("bbq", "is soso");

</script>


<template>
  <!-- <div class="BTS"> -->
  <div class = "overall">
    <h1 class = "title">{{ msg }}</h1>
    
    <!-- <p class = "title">
      온도 습도 대기압
    </p> -->
    <VueApexCharts width="500" type="line" :options="options1" :series="series1" />
  </div>
  <div>
    <p></p><p></p>
  </div>
  <div id="chart">
    <!-- <p>
      속도
    </p> -->
      <VueApexCharts width="500" type="radialBar" :options="options2" :series="series2" />
  </div>

  <!-- <div id="chart">
    <p>
      최창환 바보
    </p>
      <VueApexCharts width="500" type="radar" :options="options3" :series="series3" />
  </div> -->
</template>
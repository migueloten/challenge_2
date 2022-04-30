<template>
  <div>
    <div class="title">
      <b-navbar>
        <b-navbar-brand>
          <img src="../assets/logo.png" alt="Transport logo" height="40">
          <b>{{ appName }}</b>
        </b-navbar-brand>
      </b-navbar>
    </div>

    <div class="content">
      <div class="parameters">
        <h1 class="internalTitle">Como é o frete que você precisa?</h1>
        <form action='' id="infos" onsubmit="return false">
          <label>Destino</label>
          <br>
          <select required id="destination" v-model="destination" dat="S">
            <option value="0" disabled>Selecione aqui o destino do frete</option>
            <option v-for= "option in destinations" :value="option.value" :key="option.value"> {{ option.text }} </option>
          </select>
          <br>
          <label for="weight">Peso</label>
          <br>
          <input required id="weight" v-model="weight" type="number" min="0" step=".1" name="weight" placeholder="Insira aqui o peso da carga em Kg">
          <br>
          <div class="button">
            <button class="analyze" @click="resultOptions">Analisar</button>
          </div>
        </form>
      </div>

      <div class="results d-none">
        <h1 class="internalTitle">Estas são as melhores alternativas de frete que encontramos para você</h1>
        <div class="cheap">
          <p> 
            <b-icon icon="cash-coin"></b-icon>
            Frete mais barato: 
            <b class="cheaper"></b>
          </p>
        </div>
        <div class="fast">
          <p> 
            <b-icon icon="stopwatch"></b-icon>
            Frete mais rápido: 
            <b class="faster"></b>
          </p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import {
  BNavbar,
  BNavbarBrand,
} from 'bootstrap-vue'
import axios from 'axios'

export default {
  components: {
    BNavbar,
    BNavbarBrand,
  },
  data() {
    let info = ""
    const appName = ''
    let destinations = []
    let destination = 0
    let weight
    return {
      appName,
      info,
      destinations,
      destination,
      weight
    }
  },
  created() {
    this.getData();
    this.appName = 'Melhor Frete'
  },
  methods: {
    getData(){
      axios
      .get('http://localhost:3000/transport')
      .then(response => (this.options(response.data)))
    },
    // Implemente aqui os metodos utilizados na pagina
    options: function(info) {
      this.info = info
      var options = []
      info.forEach(element => {
        options.push(element.city)
      })
      options = [...new Set(options)];
      options.forEach(element => {
        var option = {}
        option.text = element
        option.value = element
        this.destinations.push(option)
      })
    },

    resultOptions: function(){
      let fretes = []
      let cheaper = {cost_transport_light: 1000, cost_transport_heavy: 1000}
      let faster = {lead_time: 1000}
      let bCheaper = document.getElementsByClassName('cheaper')[0]
      let bFaster = document.getElementsByClassName('faster')[0]

      this.info.forEach(element => {
        if(element.city == this.destination){
          fretes.push(element)
        }
      })

      let type = ''
      if(this.weight <= 100){
        type = 'cost_transport_light'
      }
      else {
        type = 'cost_transport_heavy'
      }
      fretes.forEach(element => {
        let price
        try{
          price = element[type].replace("R$ ", '')
        }
        catch(err){
          price = element[type]
        }
        element[type] = parseFloat(price)
        if(element[type] < cheaper[type]){
          cheaper = element
        }
      })

      fretes.forEach(element => {
        let time
        try{
          time = element.lead_time.replace("h", '')
        }
        catch(err){
          time = element.lead_time
        }
        element.lead_time = parseInt(time)
        if(element.lead_time < faster.lead_time){
          faster = element
        }
      })
      
      bCheaper.innerHTML = cheaper.name + " - R$ " + (cheaper[type]*this.weight).toFixed(2) + " - " + cheaper.lead_time + "h"
      bFaster.innerHTML = faster.name + " - R$ " + (faster[type]*this.weight).toFixed(2) + " - " + faster.lead_time + "h"
      if (cheaper.name){
        document.getElementsByClassName('results')[0].classList.remove('d-none')
      }
    }
  },
}
</script>

<style scoped>
/*---------------------------------NavBar--------------------------------*/
.title .navbar {
  background-color: #93c47d !important;
}

.title .navbar-brand {
  margin-left: 20px;
  color: #000;
}

.navbar-brand img{
  margin-right: 10px;
}

/*------------------------------Conteúdo----------------------------------*/
.content{
  padding: 0 40px
}

.internalTitle {
  font-weight: 600;
  font-size: 14px;
  color: #000;
  border-bottom: 1px solid #000;
  max-width: 410px;
  padding: 0 0 5px 10px;
  margin: 0;
}

.parameters, .results{
  max-width: 800px;
  margin-top: 30px;
}

/*-------------Seção de adição de parametros para a busca----------------*/

.parameters form{
  padding: 0 0 0 20px;
  max-width: 500px;
}

label{
  margin-top: 25px;
}

input, select{
  font-size: 12px;
  width: 100%;
  padding: 10px;
  border: 1px solid #000;
  background-color: #cfe2f3
}

.button{
  text-align: center;
}

.analyze {
  background-color: #93c47d;
  color: #000;
  font-weight: 600;
  border: #000 1px solid;
  border-radius: 5px;
  padding: 5px 25px;
  margin-top: 20px;  
}

/*------------------Seção de visualização do resultado--------------------*/
.cheap, .fast{
  margin-top: 15px;
  border: 2px dashed;
  border-radius: 5px;
  max-width: 600px;
  font-size: 14px;
}

.cheap{
  background-color: #d9ead3;
  border-color: #9ebc91;
  padding: 7px;
}

.fast{
  background-color: #c9daf8;
  border-color: #83a7e6;
  padding: 7px;
}

.results p{
  margin: 0
}


</style>

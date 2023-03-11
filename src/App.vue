<script setup>
import { inject } from "@vercel/analytics";

import { ref } from "vue";

inject(); //vercel analytics
const inputText = ref("");

const numeroDeDiasDePre = ref("0");
const numeroDeCaracteres = ref("0");
const numeroDeHoras = ref("00h00");

//experimental
const dataDeEntregaCopiado = ref("content_copy");
const dataDeEntrega = ref("");
//experimental

const diasCopiado = ref("content_copy");
const caracteresCopiado = ref("content_copy");
const horasCopiado = ref("content_copy");

const copiado = ref("");

let numeroDeCaracteresCalculado = "";

const calcCaracter = () => {
  numeroDeCaracteresCalculado = inputText.value.length;
  numeroDeCaracteres.value = numeroDeCaracteresCalculado.toLocaleString();
  inputText.value = "";
};

const calcHoras = () => {
  // ja que 800 carcteres por minuto
  // equivale a 48000 por hora

  const totalMinutes = Math.round(numeroDeCaracteresCalculado / 800);
  const minutes = totalMinutes % 60;
  const hours = Math.floor(totalMinutes / 60);

  numeroDeHoras.value = `${String(hours).padStart(2, "0")}h${String(
    minutes
  ).padStart(2, "0")}`;
};

const calcPre = () => {
  numeroDeDiasDePre.value = Math.ceil(numeroDeCaracteresCalculado / 76000);
};

//experimental

const weekendDataBase = [
  {
    date: "4/7/23",
    name: "Sexta-Feira Santa",
    level: "Feriado Nacional",
    nationalDate: '7/4/23'
  },
  {
    date: "4/21/23",
    name: "Dia de Tiradentes",
    level: "Feriado Nacional",
    nationalDate: '21/4/23'
  },
  {
    date: "5/1/23",
    name: "Dia do Trabalho",
    level: "Feriado Nacional",
    nationalDate: '1/5/23'
  },
  {
    date: "6/8/23",
    name: "Corpus Christi",
    level: "Feriado municipal",
    nationalDate: '8/6/23'
  },
  {
    date: "7/9/23",
    name: "Revolução Constitucionalista",
    level: "Feriado Estadual",
    nationalDate: '9/7/23'
  },
  {
    date: "9/7/23",
    name: "Independência do Brasil",
    level: "Feriado Nacional",
    nationalDate: '7/9/23'
  },
  {
    date: "10/12/23",
    name: "Nossa Senhora Aparecida",
    level: "Feriado Nacional",
    nationalDate: '12/10/23'
  },
  {
    date: "11/2/23",
    name: "Dia de Finados",
    level: "Feriado Nacional",
    nationalDate: '2/11/23'
  },
  {
    date: "11/15/23",
    name: "Proclamação da República",
    level: "Feriado Nacional",
    nationalDate: '15/11/23'
  },
  {
    date: "11/20/23",
    name: "Dia da Consciência Negra",
    level: "Feriado Municipal",
    nationalDate: '20/11/23'
  },
];
const printWeekends = ref([{
  name: 'Exemplo',
  nationalDate: '1/1/23'
}]); 

const compensarFeriados = (a, future) => {
  const toDay = a.getTime();

  const tester = (n) => {
    const time = new Date(n["date"]).getTime();
    if (toDay < time && future > time) return true;
  };


  const daysFiltered = weekendDataBase.filter(tester);

  printWeekends.value= daysFiltered
  return daysFiltered.length;
};



const finalDate = (future, count, daysOff) => {
  const toDayForFutureCount = new Date();
  const futureDateForTester = toDayForFutureCount.setDate(
    new Date(future).getDate() + daysOff + count
  );

  const weekDay = new Date(futureDateForTester).getDay();

  if (weekDay == 7) {
    return toDayForFutureCount.setDate(
      new Date(future).getDate() + daysOff + count + 1
    );
  } else if (weekDay == 6) {
    console.log("rodei");
    return toDayForFutureCount.setDate(
      new Date(future).getDate() + daysOff + count + 2
    );
  }
 
  return futureDateForTester;
};



const calcDate = () => {
  const days = numeroDeDiasDePre.value;
  const daysForWeekends = Math.floor(days / 5);
  const weekendsCounted = daysForWeekends * 2 + days - 1;

  const toDay = new Date();
  const toDayForFutureCount = new Date();

  const futureDateForTester = toDayForFutureCount.setDate(
    toDay.getDate() + weekendsCounted
  );

  const setDaysOff = compensarFeriados(toDay, futureDateForTester);

  const resultOfDaysOff = finalDate(
    futureDateForTester,
    weekendsCounted,
    setDaysOff
  );
  const formated = new Intl.DateTimeFormat("pt-BR", {
    day: "numeric",
    weekday: "long",
    month: "short",
    year: "numeric",
  }).format(resultOfDaysOff);
  dataDeEntrega.value = formated;

};


//experimental

const imprimirCalc = () => {
  if (inputText.value == "") return;
  calcCaracter();
  calcHoras();
  calcPre();
  calcDate();
  diasCopiado.value = "content_copy";
  caracteresCopiado.value = "content_copy";
  horasCopiado.value = "content_copy";
  dataDeEntregaCopiado.value = "content_copy";
};

const copiar = (n, m) => {
  navigator.clipboard
    .writeText(n)
    .then(() => console.log("Texto copiado com sucesso!"))
    .catch((err) => console.error("Falha ao copiar o texto:", err));

  const funcoesDeSimbolos = [
    function () {
      diasCopiado.value = "file_copy";
    },
    function () {
      caracteresCopiado.value = "file_copy";
    },
    function () {
      horasCopiado.value = "file_copy";
    },
    function () {
      dataDeEntregaCopiado.value = "file_copy";
    },
  ];
  funcoesDeSimbolos[m]();
  copiado.value = n;
};
</script>

<template>
  <div class="card">
    <p class="header">
      Copie e cole todo o texto do PDF, depois use o botão 'Calcular' ou tecle
      'Enter'
    </p>
    <div class="subCard">
      <textarea v-model="inputText" @keyup.enter="imprimirCalc"> </textarea>
      <button @click="imprimirCalc">Calcular</button>
    </div>

    <div class="subCard">
      <p>Dias de pré necessários (sem revisão): {{ numeroDeDiasDePre }}</p>
      <button @click="copiar(numeroDeDiasDePre, 0)">
        <span class="material-symbols-outlined"> {{ diasCopiado }} </span>
      </button>
    </div>
    <div class="subCard">
      <p>
        Número de caracteres:
        {{ numeroDeCaracteres }}
      </p>
      <button @click="copiar(numeroDeCaracteres, 1)">
        <span class="material-symbols-outlined"> {{ caracteresCopiado }} </span>
      </button>
    </div>
    <div class="subCard">
      <p>Número de horas: {{ numeroDeHoras }}</p>
      <button @click="copiar(numeroDeHoras, 2)">
        <span class="material-symbols-outlined"> {{ horasCopiado }} </span>
      </button>
    </div>

    <div class="subCard">
      <p>Data de entrega: {{ dataDeEntrega }}</p>
      <button @click="copiar(dataDeEntrega, 3)">
        <span class="material-symbols-outlined">
          {{ dataDeEntregaCopiado }}
        </span>
      </button>
    </div>
  <div class='date'>
  
    <p> Se houver feriados entre os dias de pré você os verá aqui! </p>
    <ul>
      <li v-for="n in printWeekends">
        Nome do feriado: {{ n.name }}
        Data: {{ n.nationalDate }}
      </li>
    </ul>

  </div>
    <div class="footer">
      <p class="header">
        Valor copiado para área de transferência: {{ copiado }}
      </p>
      <span class="material-symbols-outlined"> file_copy </span>
    </div>
    <h4>De onde vem nossas métricas?</h4>
    <p class="text">
      Sabemos que 48 mil caracteres são normalmente o que um narrador lê por
      hora. Usamos 800 para calcular um minuto de áudio. Sabemos que o GOT teve
      uma média arrendonda para baixo de 76 mil caracteres de pré por dia, ou
      seja => caracteres totais / 76.0000.
    </p>
  </div>
</template>

<style scoped>
.card {
  width: 500px;
}

button {
  height: 40px;
}
.header {
  margin: 20px;
  font-weight: 500;
}
textarea {
  resize: none;
  width: 370px;
  height: 200px;
  border-radius: 5px;
  padding: 5px;
}

textarea:focus {
  outline: none;
}

button:focus {
  outline: none;
}
.subCard {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin: 5px;
}

.footer {
  display: flex;
  align-items: center;
}

.text {
  text-align: justify;
}

li{
  list-style: none;
}

.date{
  margin: 10px 0px;
  text-align: start;
}
</style>

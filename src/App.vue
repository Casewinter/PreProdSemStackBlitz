<script setup>
import { ref } from 'vue';

const inputText = ref('');

const numeroDeDiasDePre = ref('0');
const numeroDeCaracteres = ref('0');
const numeroDeHoras = ref('00h00');

//experimental
const dataDeEntregaCopiado = ref('content_copy');
const dataDeEntrega = ref('');

const diasCopiado = ref('content_copy');
const caracteresCopiado = ref('content_copy');
const horasCopiado = ref('content_copy');

const copiado = ref('');

let numeroDeCaracteresCalculado = '';

const calcCaracter = () => {
  numeroDeCaracteresCalculado = inputText.value.length;
  numeroDeCaracteres.value = numeroDeCaracteresCalculado.toLocaleString();
  inputText.value = '';
};

const calcHoras = () => {
  // ja que 800 carcteres por minuto
  // equivale a 48000 por hora

  const totalMinutes = Math.round(numeroDeCaracteresCalculado / 800);
  const minutes = totalMinutes % 60;
  const hours = Math.floor(totalMinutes / 60);

  numeroDeHoras.value = `${String(hours).padStart(2, '0')}h${String(
    minutes
  ).padStart(2, '0')}`;
};

const calcPre = () => {
  numeroDeDiasDePre.value = Math.ceil(numeroDeCaracteresCalculado / 76000);
};

//experimental

const calcHour = () => {
  const baseDaCronometria = numeroDeDiasDePre.value;
  const baseDeCalculoParaFeriados = Math.floor(baseDaCronometria / 5);
  const feriadosCompensados =
    baseDeCalculoParaFeriados * 2 + baseDaCronometria - 1;

  const hoje = new Date();
  let base = new Date();

  const futuro = base.setDate(hoje.getDate() + feriadosCompensados);
  const formatado = new Intl.DateTimeFormat('pt-BR', {
    day: 'numeric',
    weekday: 'long',
    month: 'short',
    year: 'numeric',
  }).format(futuro);
  dataDeEntrega.value = formatado;
};

//experimental

const imprimirCalc = () => {
  if (inputText.value == '') return;
  calcCaracter();
  calcHoras();
  calcPre();
  calcHour();
  diasCopiado.value = 'content_copy';
  caracteresCopiado.value = 'content_copy';
  horasCopiado.value = 'content_copy';
  dataDeEntregaCopiado.value = 'content_copy';
};

const copiar = (n, m) => {
  navigator.clipboard
    .writeText(n)
    .then(() => console.log('Texto copiado com sucesso!'))
    .catch((err) => console.error('Falha ao copiar o texto:', err));

  const funcoesDeSimbolos = [
    function () {
      diasCopiado.value = 'file_copy';
    },
    function () {
      caracteresCopiado.value = 'file_copy';
    },
    function () {
      horasCopiado.value = 'file_copy';
    },
    function () {
      dataDeEntregaCopiado.value = 'file_copy';
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
</style>

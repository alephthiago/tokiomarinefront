<template>
  <div class="main-form">
    <div class="header-gradient">
  <img
    src="https://www.tokiomarine.com.br/wp-content/themes/tokiomarine/assets/icons/common/logo-tokiomarine.svg"
    alt="Logo Tokio Marine"
    class="logo-image"
  />
</div>
    <p v-if="errorMessage" class="error-message">{{ errorMessage }}</p>
    <h2>Transferência Bancária</h2>
    <form @submit.prevent="submitTransfer">
      <!-- Input para a conta de origem -->
      <div>
        <label for="originAccount">Conta de Origem:</label>
        <input
          type="text"
          id="originAccount"
          v-model="originAccount"
          placeholder="Digite a conta de origem"
          required
          maxlength="10"
          pattern="[A-Za-z0-9]+"
        />
      </div>

      <!-- Input para a conta de destino -->
      <div>
        <label for="destinationAccount">Conta de Destino:</label>
        <input
          type="text"
          id="destinationAccount"
          v-model="destinationAccount"
          placeholder="Digite a conta de destino"
          required
          maxlength="10"
          pattern="[A-Za-z0-9]+"
        />
      </div>

      <!-- Input para o valor -->
      <div>
        <label for="amount">Valor:</label>
        <input
          type="number"
          id="amount"
          v-model="amount"
          placeholder="Digite o valor"
          required
          min="0.01"
          step="0.01"
        />
      </div>

      <div>
        <label for="scheduleDate">Data da transferencia:</label>
        <input
          type="text"
          id="scheduleDate"
          v-model="scheduleDate"
          placeholder="DD/MM/AAAA"
          required
          pattern="\d{2}/\d{2}/\d{4}"
        />
      </div>

      <!-- Botão Enviar -->
      <button
        type="button"
        :class="{ error: errorMessage }"
        @click="checkAccountAndProceed"
      >
        Enviar
      </button>

      <!-- Botão Mostrar Agendamentos -->
      <button type="button" @click="fetchScheduledTransfers">Mostrar Agendamentos</button>
    </form>
    <!-- Componente do pop-up -->
    <Agendamentos
      :isOpen="isPopupOpen"
      :schedules="scheduledTransfers"
      @close="isPopupOpen = false"
    />
    <!-- Componente do pop-up de confirmação -->
    <ConfirmationPopup
      :isOpen="isConfirmationPopupOpen"
      :message="confirmationMessage"
      :canProceed="canProceed"
      @close="closeConfirmationPopup"
      @proceed="checkTaxAndProceed"
    />
  </div>
</template>

<script>
import axios from "axios";
import Agendamentos from "./Agendamentos.vue"; // Importe o componente do pop-up
import ConfirmationPopup from "./ConfirmationPopup.vue";
export default {
  components: {
    Agendamentos,
    ConfirmationPopup,
  },
  data() {
    return {
      originAccount: "",
      destinationAccount: "",
      amount: null,
      scheduleDate: "",
      tax: null, // Armazena a taxa retornada pela API
      errorMessage: "", // Armazena a mensagem de erro
      isPopupOpen: false, // Controla a exibição do pop-up
      scheduledTransfers: [], // Lista de agendamentos
      isConfirmationPopupOpen: false, // Controla o pop-up de confirmação
      confirmationMessage: "", // Mensagem do pop-up de confirmação
      canProceed: false, // Controla a exibição do botão "Prosseguir"
    };
  },
  methods: {
    formatDateToISO(date) {
      const [day, month, year] = date.split("/");
      const formattedDate = `${year}-${month}-${day}T00:00:00`;
      return formattedDate;
    },

    // Verifica a conta antes de prosseguir com a verificação da taxa
    async checkAccountAndProceed() {
      if (this.originAccount && this.destinationAccount && this.amount && this.scheduleDate) {
      try {
        // Faz a requisição POST para checar a conta
        const response = await axios.post("http://localhost:8080/api/conta/checar", {          
            contaOrigem: this.originAccount,
            contaDestino: this.destinationAccount,
            deduzir: this.amount,          
        });

        if (response.status === 200) {
          // Caso a resposta seja 200 OK, mostra a mensagem e habilita o botão "Prosseguir"
          this.confirmationMessage = response.data;
          this.canProceed = true;
          this.isConfirmationPopupOpen = true;
        }
      } catch (error) {
        if (error.response && error.response.status === 404) {
          // Caso a resposta seja 404, mostra a mensagem de erro e desabilita o botão "Prosseguir"
          this.confirmationMessage = error.response.data;
          this.canProceed = false;
          this.isConfirmationPopupOpen = true;
        } else {
          console.error("Erro ao checar a conta:", error);
          this.showError("Erro ao checar a conta. Tente novamente.");
        }
      }
      } else {
        alert("Por favor, preencha todos os campos.");
      }
    },

    // Fecha o pop-up de confirmação
    closeConfirmationPopup() {
      this.isConfirmationPopupOpen = false;
    },

    async checkTaxAndProceed() {
      this.closeConfirmationPopup(); 
      try {
        const formattedScheduleDate = this.formatDateToISO(this.scheduleDate);
          // Obter a data atual no formato AAAA-MM-DDTHH:MM:SS
        const currentDate = new Date().toISOString().slice(0, 19); // Formata para o padrão desejado
        // Faz a requisição POST para checar a taxa
        const response = await axios.post("http://localhost:8080/api/faixa/checar", {
          contaOrigem: this.originAccount,
          contaDestino: this.destinationAccount,
          valor: this.amount,
          dataAgendamento: this.formatDateToISO(this.scheduleDate),
          dataTransferencia: formattedScheduleDate,
        });

        // Verifica se há um retorno válido
        if (response.data && response.data.semEquadramento !== true) {
          this.tax = response.data.taxa; // Armazena a taxa
          this.errorMessage = ""; // Limpa a mensagem de erro
          this.submitTransfer(); // Prossegue para enviar a transferência
        } else {
          this.showError("Não existe taxa aplicável");
        }
      } catch (error) {
        console.error("Erro ao checar a taxa:", error);
        this.showError("Erro ao checar a taxa. Tente novamente.");
      }
    },

    showError(message) {
      this.errorMessage = message;
    },

    async submitTransfer() {
      // Validação básica dos campos de entrada
      if (this.originAccount && this.destinationAccount && this.amount) {
        try {
          const formattedScheduleDate = this.formatDateToISO(this.scheduleDate);
          // Obter a data atual no formato AAAA-MM-DDTHH:MM:SS
          const currentDate = new Date().toISOString().slice(0, 19); // Formata para o padrão desejado

          // Dados que serão enviados para o backend Spring Boot
          const transferData = {
            contaOrigem: this.originAccount,
            contaDestino: this.destinationAccount,
            valor: this.amount,
            dataAgendamento: currentDate,
            dataTransferencia: formattedScheduleDate,
            taxa: this.tax,
          };

          // Fazer o POST request para o endpoint do backend
          const response = await axios.post(
            "http://localhost:8080/api/transferencia/agendar", 
            transferData
          );

          // Mostrar uma mensagem de sucesso
          alert("Agendamento realizado com sucesso");
        } catch (error) {
          // Tratamento de erro
          console.error("Erro ao realizar a transferência:", error);
          alert("Erro ao realizar a transferência. Verifique os dados e tente novamente.");
        }
      } else {
        alert("Por favor, preencha todos os campos.");
      }
    },
    // Função para mostrar agendamentos
    async fetchScheduledTransfers() {
      try {
        const response = await axios.get("http://localhost:8080/api/transferencia/transferencias"); 
        this.scheduledTransfers = response.data;
        this.isPopupOpen = true; // Abre o pop-up após receber os dados
      } catch (error) {
        console.error("Erro ao buscar agendamentos:", error);
        alert("Erro ao buscar agendamentos.");
      }
    },
  },
};
</script>

<style scoped>
.main-form {
  width: 800px;
  margin: 0 auto;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

h2 {
  text-align: center;
}

.error-message {
  color: red;
  text-align: center;
  font-weight: bold;
}

form {
  display: flex;
  flex-direction: column;
}

div {
  margin-bottom: 10px;
}

label {
  margin-bottom: 5px;
  font-weight: bold;
}

input {
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

button {
  padding: 10px;
  margin-top: 10px;
  cursor: pointer;
  border: none;
  border-radius: 4px;
  background-color: #007bff;
  color: white;
  font-size: 14px;
  transition: background-color 0.3s;
}

button:hover {
  background-color: #0056b3;
}

button.error {
  background-color: red;
}

button[type="button"] {
  background-color: #28a745;
}

button[type="button"]:hover {
  background-color: #218838;
}

.header-gradient {
  background: transparent linear-gradient(89deg, #dfc762, #148163);
  height: 47px;
  display: flex;
  align-items: center;
  padding: 0 10px;
}

.logo-image {
  height: 47px;
}
</style>

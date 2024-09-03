<template>
    <div class="popup-overlay" v-if="isOpen">
      <div class="popup-content">
        <h3>Agendamentos</h3>
        <table>
          <thead>
            <tr>
              <th>Conta Origem</th>
              <th>Conta Destino</th>
              <th>Data Transferência</th>
              <th>Data Agendamento</th>
              <th>Valor</th>
              <th>Taxa</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(item, index) in schedules" :key="index">
              <td>{{ item.contaOrigem }}</td>
              <td>{{ item.contaDestino }}</td>
              <td>{{ formatDate(item.dataTransferencia) }}</td>
              <td>{{ formatDate(item.dataAgendamento) }}</td>
              <td>{{ formatCurrency(item.valor) }}</td>
              <td>{{ item.taxa }}</td>
            </tr>
          </tbody>
        </table>
        <button @click="closePopup">Fechar</button>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    props: {
      isOpen: {
        type: Boolean,
        required: true,
      },
      schedules: {
        type: Array,
        default: () => [],
      },
    },
    methods: {
      // Fecha o pop-up
      closePopup() {
        this.$emit("close");
      },
      // Converte data de AAAA-MM-DDTHH:MM:SS para DD/MM/AAAA
      formatDate(date) {
        if (!date) return "";
        const [year, month, day] = date.split("T")[0].split("-");
        return `${day}/${month}/${year}`;
      },
      // Formata o valor como moeda
      formatCurrency(value) {
        return new Intl.NumberFormat("pt-BR", {
          style: "currency",
          currency: "BRL",
        }).format(value);
      },
    },
  };
  </script>
  
  <style scoped>
  .popup-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.5);
    display: flex;
    justify-content: center;
    align-items: center;
  }
  
  .popup-content {
    background: white;
    padding: 20px;
    border-radius: 8px;
    width: 80%;
    max-width: 800px;
  }
  
  table {
    width: 100%;
    border-collapse: collapse;
    margin-bottom: 20px;
  }
  
  th,
  td {
    border: 1px solid #ddd;
    padding: 8px;
    text-align: center;
  }
  
  th {
    background-color: #f2f2f2;
  }
  
  button {
    padding: 10px 20px;
    cursor: pointer;
    border: none;
    border-radius: 4px;
    background-color: #007bff;
    color: white;
  }
  
  button:hover {
    background-color: #0056b3;
  }
  </style>
  
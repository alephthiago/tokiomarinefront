<template>
    <div v-if="isOpen" class="popup-overlay">
      <div class="popup-content">
        <p>{{ message }}</p>
        <div class="popup-buttons">
          <!-- Botão Fechar para todas as situações -->
          <button @click="closePopup">Fechar</button>
          <!-- Botão Prosseguir, exibido somente se o status for 200 -->
          <button v-if="canProceed" @click="proceed">Prosseguir</button>
        </div>
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
      message: {
        type: String,
        required: true,
      },
      canProceed: {
        type: Boolean,
        default: false,
      },
    },
    methods: {
      closePopup() {
        this.$emit("close");
      },
      proceed() {
        this.$emit("proceed");
      },
    },
  };
  </script>
  
  <style scoped>
  .popup-overlay {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: rgba(0, 0, 0, 0.7);
    display: flex;
    justify-content: center;
    align-items: center;
  }
  
  .popup-content {
    background: white;
    padding: 20px;
    border-radius: 8px;
    text-align: center;
  }
  
  .popup-buttons {
    margin-top: 20px;
  }
  
  button {
    margin: 5px;
    padding: 10px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
  }
  
  button:first-of-type {
    background-color: #f44336;
    color: white;
  }
  
  button:last-of-type {
    background-color: #4caf50;
    color: white;
  }
  </style>
  
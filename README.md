<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>💈 Controle de Cortes</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 20px;
      background: #f5f5f5;
    }
    h1 {
      text-align: center;
      color: #333;
    }
    .precos, .formulario {
      background: white;
      padding: 15px;
      border-radius: 10px;
      box-shadow: 0 0 5px rgba(0,0,0,0.2);
      margin-bottom: 20px;
    }
    table {
      width: 100%;
      border-collapse: collapse;
      margin-bottom: 20px;
      background: white;
    }
    th, td {
      padding: 10px;
      border: 1px solid #ccc;
      text-align: center;
    }
    th {
      background: #eee;
    }
    input, select, button {
      padding: 8px;
      margin: 5px 0;
      border-radius: 5px;
      border: 1px solid #ccc;
    }
    button {
      background: #333;
      color: white;
      cursor: pointer;
    }
    button:hover {
      background: #555;
    }
    .total {
      font-weight: bold;
      margin-top: 10px;
      font-size: 18px;
    }
  </style>
</head>
<body>
  <h1>💈 Controle de Cortes</h1>

  <!-- Tabela de preços -->
  <div class="precos">
    <h2>Tabela de Preços</h2>
    <table>
      <tr><th>Serviço</th><th>Preço (R$)</th></tr>
      <tr><td>Corte</td><td>25</td></tr>
      <tr><td>Barba</td><td>15</td></tr>
      <tr><td>Sobrancelha</td><td>10</td></tr>
      <tr><td>Corte + Barba</td><td>40</td></tr>
      <tr><td>Corte + Barba + Sobrancelha</td><td>50</td></tr>
    </table>
  </div>

  <!-- Registrar corte avulso -->
  <div class="formulario">
    <h2>Registrar Corte</h2>
    <label for="tipoCorte">Tipo de serviço:</label>
    <select id="tipoCorte">
      <option value="25">Corte - R$25</option>
      <option value="15">Barba - R$15</option>
      <option value="10">Sobrancelha - R$10</option>
      <option value="40">Corte + Barba - R$40</option>
      <option value="50">Corte + Barba + Sobrancelha - R$50</option>
    </select>
    <button onclick="registrarCorte()">Adicionar Corte</button>

    <h3>Histórico de cortes</h3>
    <table id="tabelaCortes">
      <tr>
        <th>Serviço</th>
        <th>Valor (R$)</th>
        <th>Data</th>
      </tr>
    </table>
    <p class="total">Total em cortes: R$ <span id="totalCortes">0</span></p>
  </div>

  <!-- Clientes com plano -->
  <div class="formulario">
    <h2>Clientes com Plano Mensal</h2>
    <label for="nomeCliente">Nome:</label>
    <input type="text" id="nomeCliente" placeholder="Nome do cliente">
    <label for="planoCliente">Plano:</label>
    <input type="text" id="planoCliente" placeholder="Ex: Corte semanal">
    <label for="valorPlanoCliente">Valor do plano (R$):</label>
    <input type="number" id="valorPlanoCliente" placeholder="Ex: 100">
    <button onclick="adicionarCliente()">Adicionar Cliente</button>

    <h3>Lista de Clientes</h3>
    <table id="tabelaClientes">
      <tr>
        <th>Nome</th>
        <th>Plano</th>
        <th>Valor (R$)</th>
        <th>Data de Entrada</th>
      </tr>
    </table>
    <p class="total">Total de planos cadastrados: R$ <span id="totalPlanos">0</span></p>
  </div>

  <script>
    let totalCortes = 0;
    let totalPlanos = 0;
    let clientes = [];

    function registrarCorte() {
      const select = document.getElementById('tipoCorte');
      const valor = parseInt(select.value);
      const texto = select.options[select.selectedIndex].text;
      const data = new Date().toLocaleString();

      const tabela = document.getElementById('tabelaCortes');
      const novaLinha = tabela.insertRow();
      novaLinha.insertCell(0).innerText = texto;
      novaLinha.insertCell(1).innerText = valor;
      novaLinha.insertCell(2).innerText = data;

      totalCortes += valor;
      atualizarResumo();
    }

    function adicionarCliente() {
      const nome = document.getElementById('nomeCliente').value;
      const plano = document.getElementById('planoCliente').value;
      const valor = parseFloat(document.getElementById('valorPlanoCliente').value);
      const data = new Date().toLocaleDateString();

      if (nome && plano && valor > 0) {
        clientes.push({nome, plano, valor, data});
        const tabela = document.getElementById('tabelaClientes');
        const novaLinha = tabela.insertRow();
        novaLinha.insertCell(0).innerText = nome;
        novaLinha.insertCell(1).innerText = plano;
        novaLinha.insertCell(2).innerText = valor.toFixed(2).replace(".", ",");
        novaLinha.insertCell(3).innerText = data;

        document.getElementById('nomeCliente').value = "";
        document.getElementById('planoCliente').value = "";
        document.getElementById('valorPlanoCliente').value = "";

        atualizarResumo();
      } else {
        alert("Preencha todos os campos corretamente.");
      }
    }

    function atualizarResumo() {
      const totalPlanosCalc = clientes.reduce((soma, c) => soma + c.valor, 0);
      totalPlanos = totalPlanosCalc;
      document.getElementById('totalCortes').innerText = totalCortes;
      document.getElementById('totalPlanos').innerText = totalPlanos.toFixed(2).replace(".", ",");
    }
  </script>
</body>
</html>



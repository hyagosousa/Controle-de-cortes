<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>Controle de Cortes - Barbearia</title>
  <style>
    body { font-family: Arial, sans-serif; margin: 20px; }
    h2 { margin-top: 20px; }
    input, button, select { margin: 5px 0; padding: 5px; }
    table { border-collapse: collapse; width: 100%; margin-top: 10px; }
    th, td { border: 1px solid #ccc; padding: 8px; text-align: center; }
    .box { border: 1px solid #000; padding: 15px; margin-bottom: 20px; border-radius: 8px; }
  </style>
</head>
<body>

  <h1>Controle de Cortes ✂️</h1>

  <!-- Cortes avulsos -->
  <div class="box">
    <h2>Cortes Avulsos</h2>
    <button onclick="registrarCorteAvulso()">Registrar Corte Avulso</button>
    <p>Total de cortes avulsos: <span id="totalAvulso">0</span></p>
  </div>

  <!-- Clientes de plano -->
  <div class="box">
    <h2>Clientes com Plano Mensal</h2>
    <input type="text" id="nomeCliente" placeholder="Nome do cliente">
    <input type="number" id="qtdPlano" placeholder="Qtd cortes no plano">
    <button onclick="adicionarCliente()">Adicionar Cliente</button>

    <h3>Lista de Clientes</h3>
    <table>
      <thead>
        <tr>
          <th>Nome</th>
          <th>Plano (cortes)</th>
          <th>Usados</th>
          <th>Restantes</th>
          <th>Ação</th>
        </tr>
      </thead>
      <tbody id="tabelaClientes"></tbody>
    </table>
  </div>

  <!-- Histórico -->
  <div class="box">
    <h2>Histórico de Cortes</h2>
    <table>
      <thead>
        <tr>
          <th>Data/Hora</th>
          <th>Cliente</th>
          <th>Tipo</th>
        </tr>
      </thead>
      <tbody id="tabelaHistorico"></tbody>
    </table>
  </div>

  <!-- Resumo -->
  <div class="box">
    <h2>Resumo</h2>
    <p>Total cortes avulsos: <span id="resumoAvulso">0</span></p>
    <p>Total cortes de planos: <span id="resumoPlano">0</span></p>
    <p><b>Total geral: <span id="resumoGeral">0</span></b></p>
  </div>

  <script>
    let totalAvulso = 0;
    let totalPlano = 0;
    let clientes = [];
    let historico = [];

    function atualizarResumo() {
      document.getElementById("resumoAvulso").innerText = totalAvulso;
      document.getElementById("resumoPlano").innerText = totalPlano;
      document.getElementById("resumoGeral").innerText = totalAvulso + totalPlano;
    }

    function registrarCorteAvulso() {
      totalAvulso++;
      let data = new Date().toLocaleString();
      historico.push({data, cliente: "Avulso", tipo: "Avulso"});
      atualizarHistorico();
      document.getElementById("totalAvulso").innerText = totalAvulso;
      atualizarResumo();
    }

    function adicionarCliente() {
      let nome = document.getElementById("nomeCliente").value;
      let qtdPlano = parseInt(document.getElementById("qtdPlano").value);

      if (nome && qtdPlano > 0) {
        clientes.push({nome, plano: qtdPlano, usados: 0});
        atualizarTabelaClientes();
        document.getElementById("nomeCliente").value = "";
        document.getElementById("qtdPlano").value = "";
      } else {
        alert("Preencha corretamente os dados do cliente.");
      }
    }

    function registrarCortePlano(index) {
      let cliente = clientes[index];
      if (cliente.usados < cliente.plano) {
        cliente.usados++;
        totalPlano++;
        let data = new Date().toLocaleString();
        historico.push({data, cliente: cliente.nome, tipo: "Plano"});
        atualizarTabelaClientes();
        atualizarHistorico();
        atualizarResumo();
      } else {
        alert("Este cliente já usou todos os cortes do plano.");
      }
    }

    function atualizarTabelaClientes() {
      let tabela = document.getElementById("tabelaClientes");
      tabela.innerHTML = "";
      clientes.forEach((c, i) => {
        let row = `<tr>
          <td>${c.nome}</td>
          <td>${c.plano}</td>
          <td>${c.usados}</td>
          <td>${c.plano - c.usados}</td>
          <td><button onclick="registrarCortePlano(${i})">Registrar Corte</button></td>
        </tr>`;
        tabela.innerHTML += row;
      });
    }

    function atualizarHistorico() {
      let tabela = document.getElementById("tabelaHistorico");
      tabela.innerHTML = "";
      historico.slice().reverse().forEach(h => {
        let row = `<tr>
          <td>${h.data}</td>
          <td>${h.cliente}</td>
          <td>${h.tipo}</td>
        </tr>`;
        tabela.innerHTML += row;
      });
    }
  </script>
</body>
</html>

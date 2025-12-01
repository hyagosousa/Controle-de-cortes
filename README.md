<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Controle de Cortes - Barbearia (Com Caixa)</title>

<style>
  :root{
    --bg:#0f0f10;
    --card:#141416;
    --muted:#8a8a8a;
    --blue:#2d7dff;
    --green:#34c759;
    --red:#ff3b30;
  }
  body{
    margin:0;
    background:var(--bg);
    color:white;
    font-family:Arial, sans-serif;
  }
  h2{
    text-align:center;
    margin:15px 0;
    font-weight:600;
  }
  .container{
    max-width:900px;
    margin:0 auto;
    padding:15px;
  }
  .card{
    background:var(--card);
    padding:15px;
    border-radius:10px;
    margin-bottom:20px;
  }
  label{font-size:15px;}
  input, select{
    width:100%;
    padding:10px;
    margin:6px 0 15px 0;
    border-radius:6px;
    border:none;
    outline:none;
    background:#1c1c1e;
    color:white;
  }
  button{
    width:100%;
    padding:12px;
    border:none;
    border-radius:8px;
    cursor:pointer;
    margin-top:8px;
    font-size:16px;
  }
  .btn-add{background:var(--blue);}
  .btn-pago{background:var(--green);}
  .btn-remove{background:var(--red);}
  .btn-limpar{background:#444;}
  table{
    width:100%;
    border-collapse:collapse;
    margin-top:10px;
  }
  table td, table th{
    border-bottom:1px solid #333;
    padding:8px;
    text-align:left;
  }
  .totais{
    font-size:17px;
    margin-top:10px;
    line-height:28px;
  }
</style>
</head>

<body>
<div class="container">

  <h2>Controle de Cortes - Barbearia</h2>

  <div class="card">
    <label>Cliente:</label>
    <input id="cliente" type="text" placeholder="Nome do cliente">

    <label>Serviço:</label>
    <select id="servico">
      <option value="Corte">Corte - R$ 30</option>
      <option value="Barba">Barba - R$ 20</option>
      <option value="Sobrancelha">Sobrancelha - R$ 10</option>
    </select>

    <button class="btn-add" onclick="adicionarCorte()">Adicionar Corte</button>
  </div>

  <div class="card">
    <h3>Lista de Cortes</h3>
    <table id="tabelaCortes">
      <thead>
        <tr><th>Cliente</th><th>Serviço</th><th>Valor</th><th>Ações</th></tr>
      </thead>
      <tbody></tbody>
    </table>
  </div>

  <div class="card">
    <h3>Controle de Caixa</h3>

    <div class="totais">
      Entradas totais: R$ <span id="entradas">0.00</span> <br>
      Saídas totais: R$ <span id="saidas">0.00</span> <br>
      Saldo atual: R$ <span id="saldo">0.00</span>
    </div>

    <button class="btn-limpar" onclick="limparCaixa()">LIMPAR CAIXA (ZERAR TUDO)</button>

    <h3>Histórico</h3>
    <table id="tabelaHistorico">
      <thead>
        <tr><th>Tipo</th><th>Valor</th><th>Data/Hora</th></tr>
      </thead>
      <tbody></tbody>
    </table>
  </div>

</div>

<script>
let entradas = 0;
let saidas = 0;

// ------------------- ADICIONAR CORTE -----------------------
function adicionarCorte(){
  const cliente = document.getElementById("cliente").value.trim();
  const servico = document.getElementById("servico").value;

  if(cliente === ""){
    alert("Digite o nome do cliente.");
    return;
  }

  let valor = 0;
  if(servico === "Corte") valor = 30;
  if(servico === "Barba") valor = 20;
  if(servico === "Sobrancelha") valor = 10;

  const tabela = document.querySelector("#tabelaCortes tbody");
  const row = document.createElement("tr");

  row.innerHTML = `
    <td>${cliente}</td>
    <td>${servico}</td>
    <td>R$ ${valor.toFixed(2)}</td>
    <td>
      <button class="btn-pago" onclick="marcarPago(${valor}, this)">Pago</button>
    </td>
  `;

  tabela.appendChild(row);
  document.getElementById("cliente").value = "";
}

// ------------------- MARCAR COMO PAGO -----------------------
function marcarPago(valor, btn){
  entradas += valor;
  atualizarTotais();

  registrarHistorico("Entrada", valor);

  btn.parentElement.parentElement.remove();
}

// ------------------- ATUALIZAR TOTAIS -----------------------
function atualizarTotais(){
  document.getElementById("entradas").innerText = entradas.toFixed(2);
  document.getElementById("saidas").innerText = saidas.toFixed(2);
  document.getElementById("saldo").innerText = (entradas - saidas).toFixed(2);
}

// ------------------- REGISTRAR HISTÓRICO --------------------
function registrarHistorico(tipo, valor){
  const tabela = document.querySelector("#tabelaHistorico tbody");
  const row = document.createElement("tr");
  row.innerHTML = `
    <td>${tipo}</td>
    <td>R$ ${valor.toFixed(2)}</td>
    <td>${new Date().toLocaleString()}</td>
  `;
  tabela.appendChild(row);
}

// ------------------- LIMPAR CAIXA (ZERAR TUDO) -------------------
function limparCaixa(){
  if(!confirm("Tem certeza que deseja zerar TODO o caixa?")) return;

  entradas = 0;
  saidas = 0;

  atualizarTotais();

  document.querySelector("#tabelaHistorico tbody").innerHTML = "";
}
</script>

</body>
</html>




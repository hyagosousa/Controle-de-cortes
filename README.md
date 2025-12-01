<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Controle de Cortes + Caixa</title>

<style>
    body{
        margin:0;
        background:#050505;
        font-family:Arial, sans-serif;
        color:white;
    }
    h2{
        margin:0 0 10px;
    }
    .container{
        width:95%;
        max-width:900px;
        margin:20px auto;
    }
    .card{
        background:#0f1220;
        padding:20px;
        border-radius:12px;
        margin-bottom:20px;
        box-shadow:0 0 10px #00000070;
    }
    input, select{
        width:100%;
        padding:10px;
        margin:5px 0 15px;
        border:none;
        border-radius:6px;
    }
    button{
        width:100%;
        padding:12px;
        border:none;
        background:#007bff;
        color:white;
        font-size:16px;
        border-radius:6px;
        margin-top:10px;
        cursor:pointer;
    }
    button:hover{ background:#005fcc; }
    table{
        width:100%;
        border-collapse:collapse;
        margin-top:10px;
    }
    th, td{
        padding:10px;
        border-bottom:1px solid #333;
        text-align:left;
    }
    .btn-small{
        padding:8px;
        font-size:14px;
        background:#28a745;
    }
    .btn-danger{
        background:#d9534f;
    }
</style>
</head>

<body>

<div class="container">

    <!-- DASHBOARD -->
    <div class="card">
        <h2>Dashboard</h2>
        <p>Total de Clientes: <b id="dashClientes">0</b></p>
        <p>Total de Cortes Realizados: <b id="dashCortes">0</b></p>
        <p>Total no Caixa: <b id="dashCaixa">R$ 0,00</b></p>
    </div>

    <!-- CADASTRO CLIENTE -->
    <div class="card">
        <h2>Cadastrar Cliente</h2>
        <input id="nomeCliente" placeholder="Nome do Cliente">
        <select id="planoCliente">
            <option value="2">Plano Mensal - 2 cortes</option>
        </select>
        <button onclick="cadastrarCliente()">Cadastrar</button>
    </div>

    <!-- LISTA CLIENTES -->
    <div class="card">
        <h2>Clientes</h2>
        <table id="tabelaClientes"></table>
    </div>

    <!-- CAIXA -->
    <div class="card">
        <h2>Caixa</h2>
        <p>Total no Caixa: <b id="totalCaixa">R$ 0,00</b></p>

        <h3>Histórico</h3>
        <table id="tabelaCaixa"></table>

        <button class="btn-danger" onclick="limparCaixa()">Limpar Caixa</button>
    </div>

</div>

<script>
let clientes = JSON.parse(localStorage.getItem("clientes")) || [];
let caixa = JSON.parse(localStorage.getItem("caixa")) || [];

function salvar(){
    localStorage.setItem("clientes", JSON.stringify(clientes));
    localStorage.setItem("caixa", JSON.stringify(caixa));
    atualizar();
}

function cadastrarCliente(){
    let nome = document.getElementById("nomeCliente").value.trim();
    if(nome === "") return alert("Digite um nome.");

    clientes.push({
        nome,
        cortesFeitos: 0,
        cortesPlano: 2
    });

    salvar();
    document.getElementById("nomeCliente").value = "";
}

function registrarCorte(index){
    let c = clientes[index];

    if(c.cortesFeitos >= c.cortesPlano){
        alert("Plano finalizado para este mês.");
        return;
    }

    c.cortesFeitos++;

    let restante = c.cortesPlano - c.cortesFeitos;

    alert(`Corte registrado! Ainda faltam ${restante} para completar o plano.`);

    salvar();
}

function adicionarAoCaixa(valor, cliente){
    caixa.push({
        valor,
        cliente,
        data: new Date().toLocaleString()
    });

    salvar();
}

function limparCaixa(){
    if(!confirm("Deseja realmente limpar TODO o caixa?")) return;

    caixa = [];
    salvar();
}

function atualizar(){
    // tabela clientes
    let html = `
        <tr>
            <th>Cliente</th>
            <th>Cortes</th>
            <th>Ações</th>
        </tr>
    `;

    clientes.forEach((c, i)=>{
        html += `
            <tr>
                <td>${c.nome}</td>
                <td>${c.cortesFeitos}/${c.cortesPlano}</td>
                <td>
                    <button class="btn-small" onclick="registrarCorte(${i})">Registrar Corte</button>
                    <button class="btn-small" onclick="adicionarAoCaixa(30, '${c.nome}')">Adicionar ao Caixa</button>
                </td>
            </tr>
        `;
    });

    document.getElementById("tabelaClientes").innerHTML = html;

    // caixa
    let htmlCaixa = `
        <tr>
            <th>Cliente</th>
            <th>Valor</th>
            <th>Data</th>
        </tr>
    `;

    let total = 0;

    caixa.forEach(e=>{
        total += e.valor;
        htmlCaixa += `
            <tr>
                <td>${e.cliente}</td>
                <td>R$ ${e.valor.toFixed(2)}</td>
                <td>${e.data}</td>
            </tr>
        `;
    });

    document.getElementById("tabelaCaixa").innerHTML = htmlCaixa;
    document.getElementById("totalCaixa").innerText = "R$ " + total.toFixed(2);

    // dashboard
    document.getElementById("dashClientes").innerText = clientes.length;
    document.getElementById("dashCortes").innerText = clientes.reduce((a,b)=>a+b.cortesFeitos,0);
    document.getElementById("dashCaixa").innerText = "R$ " + total.toFixed(2);
}

atualizar();
</script>

</body>
</html>





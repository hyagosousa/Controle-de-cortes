<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Controle de Cortes - Barbearia</title>

<style>
    body {
        font-family: Arial, sans-serif;
        background: #111;
        color: #fff;
        margin: 0;
        padding: 20px;
    }
    h2 { text-align: center; }
    .card {
        background: #1b1b1b;
        padding: 20px;
        margin: 15px 0;
        border-radius: 10px;
        box-shadow: 0 0 8px #000;
    }
    input, select, button {
        width: 100%;
        padding: 12px;
        margin: 8px 0;
        border-radius: 8px;
        border: none;
    }
    button {
        background: #0d6efd;
        color: #fff;
        font-weight: bold;
        cursor: pointer;
    }
    button:hover { background: #0b5ed7; }

    .whatsapp {
        background: #25d366 !important;
        margin-top: 5px;
    }

    table {
        width: 100%;
        border-collapse: collapse;
        margin-top: 15px;
    }
    th, td {
        border: 1px solid #333;
        padding: 10px;
        text-align: center;
    }
</style>
</head>
<body>

<h2>Controle de Cortes - Barbearia</h2>

<div class="card">
    <h3>Registrar Cliente</h3>
    <input id="nome" placeholder="Nome do cliente">
    <input id="telefone" placeholder="Telefone (somente números)">
    <input id="plano" type="number" placeholder="Quantidade de cortes no plano">
    <button onclick="addCliente()">Adicionar Cliente</button>
</div>

<div class="card">
    <h3>Registrar Corte</h3>
    <select id="clienteSelect"></select>
    <button onclick="registrarCorte()">Registrar Corte Feito</button>
</div>

<div class="card">
    <h3>Clientes e Progresso do Plano</h3>
    <table>
        <thead>
            <tr>
                <th>Cliente</th>
                <th>Plano</th>
                <th>Feitos</th>
                <th>Faltam</th>
                <th>WhatsApp</th>
            </tr>
        </thead>
        <tbody id="listaClientes"></tbody>
    </table>
</div>

<script>
    let clientes = JSON.parse(localStorage.getItem("clientesBarbearia")) || [];

    function salvar() {
        localStorage.setItem("clientesBarbearia", JSON.stringify(clientes));
    }

    function atualizarLista() {
        let lista = document.getElementById("listaClientes");
        let select = document.getElementById("clienteSelect");

        lista.innerHTML = "";
        select.innerHTML = "";

        clientes.forEach((c, index) => {
            let faltam = c.plano - c.cortes;

            lista.innerHTML += `
                <tr>
                    <td>${c.nome}</td>
                    <td>${c.plano}</td>
                    <td>${c.cortes}</td>
                    <td>${faltam}</td>
                    <td>
                        <button class="whatsapp" onclick="enviarWhatsApp(${index})">Enviar</button>
                    </td>
                </tr>
            `;

            select.innerHTML += `<option value="${index}">${c.nome}</option>`;
        });
    }

    function addCliente() {
        let nome = document.getElementById("nome").value;
        let telefone = document.getElementById("telefone").value;
        let plano = Number(document.getElementById("plano").value);

        if (!nome || plano <= 0) {
            alert("Preencha nome e plano corretamente.");
            return;
        }

        clientes.push({
            nome,
            telefone,
            plano,
            cortes: 0
        });

        salvar();
        atualizarLista();

        document.getElementById("nome").value = "";
        document.getElementById("telefone").value = "";
        document.getElementById("plano").value = "";
    }

    function registrarCorte() {
        let index = document.getElementById("clienteSelect").value;
        clientes[index].cortes++;

        salvar();
        atualizarLista();
    }

    function enviarWhatsApp(i) {
        let c = clientes[i];
        if (!c.telefone) {
            alert("Este cliente não tem telefone cadastrado.");
            return;
        }

        let faltam = c.plano - c.cortes;

        let msg = `Olá ${c.nome}! 👋\n\n` +
                  `Seu controle de cortes está assim:\n` +
                  `• Plano total: ${c.plano} cortes\n` +
                  `• Cortes realizados: ${c.cortes}\n` +
                  `• Faltam: ${faltam}\n\n` +
                  `Obrigado por escolher nossa barbearia! ✂🔥`;

        let url = `https://wa.me/55${c.telefone}?text=${encodeURIComponent(msg)}`;

        window.open(url, "_blank");
    }

    atualizarLista();
</script>

</body>
</html>




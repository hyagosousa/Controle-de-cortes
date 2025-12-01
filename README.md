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
    --muted:#cfcfcf;
    --accent:#0d6efd;
    --whats:#25d366;
    --danger:#d63333;
    --purple:#8b5cf6;
  }
  body{
    background:var(--bg);
    color:#fff;
    font-family:Inter, Arial, sans-serif;
    margin:0;
    padding:18px;
  }
  h1{ text-align:center; margin:8px 0 18px; font-size:20px; }
  .container{ max-width:1100px; margin:0 auto; }
  .grid{ display:grid; gap:12px; grid-template-columns: 1fr 1fr; }
  .card{ background:var(--card); padding:14px; border-radius:10px; box-shadow:0 6px 18px rgba(0,0,0,0.6); }
  label{ display:block; margin:6px 0 4px; color:var(--muted); font-size:13px; }
  input, select, button{ width:100%; padding:10px 12px; border-radius:8px; border:1px solid rgba(255,255,255,0.04); background:transparent; color:#fff; box-sizing:border-box; }
  .row{ display:flex; gap:8px; }
  .row > *{ flex:1; }
  button.primary{ background:var(--accent); border:none; color:#fff; font-weight:600; cursor:pointer; }
  button.ghost{ background:transparent; border:1px solid rgba(255,255,255,0.06); color:var(--muted); cursor:pointer; }
  button.whats{ background:var(--whats); color:#012; font-weight:700; }
  button.danger{ background:var(--danger); color:#fff; }
  button.caixa{ background:var(--purple); color:#fff; font-weight:600; }
  table{ width:100%; border-collapse:collapse; margin-top:12px; font-size:14px; border-radius:6px; overflow:hidden; }
  th, td{ padding:10px; border-bottom:1px solid rgba(255,255,255,0.04); text-align:left; background:transparent; }
  th { color:var(--muted); font-size:13px; background: rgba(255,255,255,0.02); }
  .small{ font-size:12px; color:var(--muted); }
  .actions button{ margin-right:6px; }
  #historyCard{ margin-top:12px; }
  .empty{ text-align:center; color:var(--muted); padding:20px 0; }
  .dashboard { display:grid; grid-template-columns: repeat(4, 1fr); gap:12px; margin-bottom:14px; }
  .dashCard { background:linear-gradient(180deg, rgba(255,255,255,0.02), transparent); padding:14px; border-radius:10px; text-align:center; }
  .dashNum { font-size:22px; color:var(--accent); margin-top:6px; }
  @media(max-width:980px){ .dashboard{ grid-template-columns: repeat(2,1fr);} .grid{ grid-template-columns: 1fr; } }
</style>
</head>
<body>
  <div class="container">
    <h1>Controle de Cortes - Barbearia</h1>

    <!-- Dashboard -->
    <div class="dashboard">
      <div class="dashCard card">
        <div class="small">Clientes</div>
        <div id="dashClientes" class="dashNum">0</div>
      </div>
      <div class="dashCard card">
        <div class="small">Cortes Feitos</div>
        <div id="dashCortes" class="dashNum">0</div>
      </div>
      <div class="dashCard card">
        <div class="small">Cortes Restantes</div>
        <div id="dashRestantes" class="dashNum">0</div>
      </div>
      <div class="dashCard card">
        <div class="small">Saldo Caixa (R$)</div>
        <div id="dashSaldo" class="dashNum">0.00</div>
      </div>
    </div>

    <div class="grid">
      <div class="card">
        <h3>Novo Cliente</h3>
        <label for="nome">Nome</label>
        <input id="nome" placeholder="Nome do cliente">
        <label for="telefone">Telefone (somente números, ex: 11999998888)</label>
        <input id="telefone" placeholder="Telefone (opcional)">
        <label for="plano">Quantidade de cortes do plano</label>
        <input id="plano" type="number" min="1" placeholder="Ex: 2">
        <div style="margin-top:8px" class="row">
          <button class="primary" id="btnAdd">Adicionar</button>
          <button class="ghost" id="btnClear">Limpar</button>
        </div>
      </div>

      <div class="card">
        <h3>Ações Rápidas</h3>
        <label for="clienteSelect">Selecionar cliente</label>
        <select id="clienteSelect"></select>

        <div style="margin-top:8px" class="row">
          <button class="primary" id="btnRegistrar">Registrar Corte</button>
          <button class="whats" id="btnWhats">Enviar WhatsApp</button>
        </div>

        <div style="margin-top:8px" class="row">
          <button class="ghost" id="btnRenovar">Renovar Plano</button>
          <button class="ghost" id="btnHistorico">Mostrar Histórico</button>
        </div>

        <p class="small" style="margin-top:10px">Observação: o histórico guarda data e hora de cada corte. Ao registrar um corte, o valor padrão será adicionado ao caixa automaticamente.</p>
      </div>
    </div>

    <div class="card" style="margin-top:12px">
      <h3>Clientes e Progresso</h3>
      <div id="tabelaWrapper"></div>
    </div>

    <div id="historyCard"></div>

    <!-- Caixa -->
    <div class="card" style="margin-top:12px">
      <h3>Controle de Caixa</h3>
      <div style="display:grid; grid-template-columns: 1fr 120px; gap:8px;">
        <input id="descCaixa" placeholder="Descrição (ex: Compra lâmina)">
        <input id="valorCaixa" type="number" placeholder="Valor (R$)">
      </div>
      <div class="row" style="margin-top:8px;">
        <button class="primary" onclick="entradaCaixa()">Registrar Entrada</button>
        <button class="danger" onclick="saidaCaixa()">Registrar Saída</button>
        <button class="ghost" onclick="exportCaixaCSV()">Exportar CSV</button>
      </div>

      <div style="margin-top:12px;">
        <div class="small">Totais</div>
        <div style="display:flex; gap:12px; margin-top:8px;">
          <div style="flex:1; background:rgba(255,255,255,0.02); padding:10px; border-radius:8px;">
            <div class="small">Entradas (R$)</div>
            <div id="totalEntradas" style="font-weight:700; margin-top:6px;">0.00</div>
          </div>
          <div style="flex:1; background:rgba(255,255,255,0.02); padding:10px; border-radius:8px;">
            <div class="small">Saídas (R$)</div>
            <div id="totalSaidas" style="font-weight:700; margin-top:6px;">0.00</div>
          </div>
          <div style="flex:1; background:rgba(255,255,255,0.02); padding:10px; border-radius:8px;">
            <div class="small">Saldo (R$)</div>
            <div id="saldoCaixa" style="font-weight:700; margin-top:6px;">0.00</div>
          </div>
        </div>
      </div>

      <h4 style="margin-top:14px">Histórico do Caixa</h4>
      <table id="tabelaCaixa">
        <thead>
          <tr><th>Tipo</th><th>Descrição</th><th>Valor</th><th>Data</th><th>Ações</th></tr>
        </thead>
        <tbody></tbody>
      </table>
    </div>

  </div>

<script>
/* ================== CONFIG ================== */
// ajuste aqui o valor padrão cobrado por corte (ao registrar corte entra automaticamente no caixa)
const VALOR_PADRAO_CORTE = 20.00;

/* ================== Storage Keys ================== */
const STORAGE_KEY = "barbearia_clientes_v2";
const STORAGE_CAIXA = "barbearia_caixa_v1";

/* ================== Dados ================== */
let clientes = JSON.parse(localStorage.getItem(STORAGE_KEY)) || [];
let caixa = JSON.parse(localStorage.getItem(STORAGE_CAIXA)) || [];

/* ================== Utilitários ================== */
function salvarClientes(){ localStorage.setItem(STORAGE_KEY, JSON.stringify(clientes)); atualizarUI(); atualizarDashboard(); }
function salvarCaixa(){ localStorage.setItem(STORAGE_CAIXA, JSON.stringify(caixa)); atualizarCaixa(); atualizarDashboard(); }
function nowStr(){ return new Date().toLocaleString(); }
function apenasDigitos(s){ return (s||"").toString().replace(/\D+/g,''); }
function formatMoney(v){ return Number(v).toFixed(2); }

/* ================== DOM Elements ================== */
const nomeI = document.getElementById("nome");
const telI  = document.getElementById("telefone");
const planoI= document.getElementById("plano");
const btnAdd= document.getElementById("btnAdd");
const btnClear = document.getElementById("btnClear");
const clienteSelect = document.getElementById("clienteSelect");
const btnRegistrar = document.getElementById("btnRegistrar");
const btnWhats = document.getElementById("btnWhats");
const btnRenovar = document.getElementById("btnRenovar");
const btnHistorico = document.getElementById("btnHistorico");
const tabelaWrapper = document.getElementById("tabelaWrapper");
const historyCard = document.getElementById("historyCard");
const descCaixaI = document.getElementById("descCaixa");
const valorCaixaI = document.getElementById("valorCaixa");

/* ================== Inicialização ================== */
atualizarUI();
atualizarCaixa();
atualizarDashboard();

/* ================== Helpers ================== */
function criarId(){ return Date.now().toString(36) + Math.random().toString(36).slice(2,7); }
function encontrarClientePorId(id){ return clientes.find(c => c.id === id); }
function indexClientePorId(id){ return clientes.findIndex(c => c.id === id); }

/* ================== Clientes: adicionar/limpar ================== */
function adicionarCliente(){
  const nome = (nomeI.value || "").trim();
  const telefoneRaw = telI.value || "";
  const telefone = apenasDigitos(telefoneRaw);
  const plano = Number(planoI.value);

  if(!nome){ alert("Preencha o nome."); return; }
  if(!plano || plano <= 0){ alert("Informe um plano válido (nº de cortes)."); return; }

  const novo = {
    id: criarId(),
    nome,
    telefone,
    plano,
    cortes: 0,
    historico: []
  };

  clientes.push(novo);
  salvarClientes();
  limparForm();
  alert("Cliente adicionado com sucesso.");
}

function limparForm(){
  nomeI.value=""; telI.value=""; planoI.value="";
}

/* ================== Atualizar UI (select + tabela) ================== */
function atualizarUI(){
  // Select
  clienteSelect.innerHTML = "";
  if(clientes.length === 0){
    clienteSelect.innerHTML = '<option value="">-- nenhum cliente cadastrado --</option>';
  } else {
    clienteSelect.innerHTML = '<option value="">-- selecione um cliente --</option>' +
      clientes.map(c => `<option value="${c.id}">${c.nome} — Plano ${c.plano} (${c.cortes} feitos)</option>`).join("");
  }

  // Tabela
  if(clientes.length === 0){
    tabelaWrapper.innerHTML = `<div class="empty">Nenhum cliente cadastrado. Adicione um acima.</div>`;
  } else {
    tabelaWrapper.innerHTML = `
      <table>
        <thead>
          <tr><th>Cliente</th><th>Plano</th><th>Feitos</th><th>Faltam</th><th class="small">Ações</th></tr>
        </thead>
        <tbody>
          ${clientes.map((c, idx)=>{
            const faltam = Math.max(0, c.plano - c.cortes);
            return `<tr data-id="${c.id}">
              <td><strong>${c.nome}</strong><div class="small">${c.telefone ? formatPhone(c.telefone) : 'Tel não cadastrado'}</div></td>
              <td>${c.plano}</td>
              <td>${c.cortes}</td>
              <td>${faltam}</td>
              <td class="actions">
                <button class="whats" onclick="enviarWhatsApp('${c.id}')">WhatsApp</button>
                <button class="ghost" onclick="mostrarHistorico('${c.id}')">Histórico</button>
                <button class="ghost" onclick="renovarPlanoPrompt('${c.id}')">Renovar</button>
                <button class="ghost" onclick="editarClientePrompt('${c.id}')">Editar</button>
                <button class="caixa" onclick="adicionarAoCaixaPrompt('${c.id}')">Adicionar ao Caixa</button>
                <button class="danger" onclick="excluirCliente('${c.id}')">Excluir</button>
              </td>
            </tr>`;
          }).join("")}
        </tbody>
      </table>
    `;
  }

  // habilitar/desabilitar botões
  const hasClients = clientes.length > 0;
  btnRegistrar.disabled = !hasClients;
  btnWhats.disabled = !hasClients;
  btnRenovar.disabled = !hasClients;
  btnHistorico.disabled = !hasClients;
}

/* ================== Registrar Corte (com entrada automática no caixa) ================== */
function registrarCorteSelecionado(){
  const id = clienteSelect.value;
  if(!id){ alert("Selecione um cliente primeiro."); return; }
  registrarCorte(id);
}
function registrarCorte(id){
  const c = encontrarClientePorId(id);
  if(!c) { alert("Cliente não encontrado."); return; }

  if(c.cortes >= c.plano){
    const ok = confirm(`${c.nome} já atingiu o plano (${c.cortes}/${c.plano}). Deseja registrar um corte EXTRA?`);
    if(!ok) return;
    // registra como extra (ainda assim soma cortes)
  }

  const antes = c.cortes;
  c.cortes = c.cortes + 1;
  c.historico.push({ tipo:'corte', data: nowStr(), info:`corte registrado (${antes} → ${c.cortes})` });

  // adicionar entrada automática no caixa
  const descricao = `Corte - ${c.nome}`;
  const valor = Number(VALOR_PADRAO_CORTE) || 0;
  caixa.push({ id: criarId(), tipo:'entrada', descricao, valor, data: nowStr(), origemClienteId: c.id, origemClienteNome: c.nome });
  salvarCaixa();

  salvarClientes();
  atualizarUI();
  mostrarHistorico(id);

  // mensagens de aviso de acordo com quantidade (específico ao seu pedido de 2 cortes mensais)
  if(c.plano === 2){
    if(c.cortes === 1){
      alert(`1º corte finalizado para ${c.nome}. Ainda falta 1 corte do plano.`);
    } else if(c.cortes === 2){
      alert(`Plano completo! Os 2 cortes do mês foram usados por ${c.nome}.`);
    }
  } else {
    // comportamento generalizado
    if(c.cortes < c.plano) {
      alert(`Corte registrado. Faltam ${Math.max(0, c.plano - c.cortes)} cortes.`);
    } else if(c.cortes === c.plano) {
      alert(`Corte registrado. Plano completo.`);
    } else {
      alert(`Corte registrado (extra).`);
    }
  }
}

/* ================== WhatsApp ================== */
function enviarWhatsApp(id){
  const c = encontrarClientePorId(id);
  if(!c) { alert("Cliente não encontrado."); return; }
  if(!c.telefone){
    const add = confirm("Cliente não tem telefone cadastrado. Deseja adicionar agora?");
    if(add){
      const t = prompt("Informe o telefone (somente números):", "");
      if(t){ c.telefone = apenasDigitos(t); salvarClientes(); }
      else { return; }
    } else return;
  }
  const faltam = Math.max(0, c.plano - c.cortes);
  const msg = `Olá ${c.nome}!%0A%0A` +
              `Seu controle de cortes está assim:%0A` +
              `• Plano total: ${c.plano} cortes%0A` +
              `• Cortes realizados: ${c.cortes}%0A` +
              `• Faltam: ${faltam}%0A%0A` +
              `Obrigado por escolher nossa barbearia! ✂️`;
  const telefone = c.telefone.replace(/^\+/, '').replace(/\s+/g,'');
  const url = `https://wa.me/55${telefone}?text=${msg}`;
  window.open(url, "_blank");
}

/* ================== Histórico (cliente) ================== */
function mostrarHistorico(id){
  const c = encontrarClientePorId(id);
  if(!c){ alert("Cliente não encontrado."); return; }
  clienteSelect.value = id;
  let html = `<div class="card"><h3>Histórico — ${c.nome}</h3>`;
  html += `<p class="small">Plano: ${c.plano} | Cortes: ${c.cortes} | Telefone: ${c.telefone ? formatPhone(c.telefone) : '—'}</p>`;
  if(!c.historico || c.historico.length === 0){
    html += `<div class="empty">Nenhuma ação registrada ainda.</div>`;
  } else {
    html += `<table><thead><tr><th>Data / Hora</th><th>Tipo</th><th>Info</th></tr></thead><tbody>`;
    const hist = [...c.historico].reverse();
    hist.forEach(h=>{
      html += `<tr><td class="small">${h.data}</td><td>${h.tipo}</td><td class="small">${h.info || ''}</td></tr>`;
    });
    html += `</tbody></table>`;
  }
  html += `<div style="margin-top:8px" class="row"><button class="primary" onclick="baixarHistorico('${c.id}')">Baixar CSV</button><button class="ghost" onclick="fecharHistorico()">Fechar</button></div>`;
  html += `</div>`;
  historyCard.innerHTML = html;
}
function fecharHistorico(){ historyCard.innerHTML = ""; }

/* ================== Renovar plano (adicionar cortes) ================== */
function renovarPlanoPrompt(id){
  if(!id){
    const sel = clienteSelect.value;
    if(!sel){ alert("Selecione um cliente ou clique Renovar na linha do cliente."); return; }
    id = sel;
  }
  const c = encontrarClientePorId(id);
  if(!c) return;
  const q = prompt(`Adicionar quantos cortes ao plano de ${c.nome}?`, "2");
  const n = Number(q);
  if(!n || n <= 0){ alert("Valor inválido."); return; }
  c.plano = c.plano + n;
  c.historico.push({ tipo:'renovacao', data: nowStr(), info: `+${n} cortes (novo plano ${c.plano})` });
  salvarClientes();
  mostrarHistorico(id);
}

/* ================== Editar / Excluir ================== */
function editarClientePrompt(id){
  const c = encontrarClientePorId(id);
  if(!c) { alert("Cliente não encontrado."); return; }
  const novoNome = prompt("Editar nome:", c.nome);
  if(!novoNome) return;
  const novoTel = prompt("Editar telefone (somente números):", c.telefone || "");
  const novoPlanoStr = prompt("Editar plano (total de cortes):", String(c.plano));
  const novoPlano = Number(novoPlanoStr);
  if(!novoPlano || novoPlano <= 0){ alert("Plano inválido."); return; }
  c.nome = novoNome.trim();
  c.telefone = apenasDigitos(novoTel || "");
  const antesPlano = c.plano;
  c.plano = novoPlano;
  c.historico.push({ tipo:'edicao', data: nowStr(), info:`plano ${antesPlano} → ${c.plano}` });
  salvarClientes();
  mostrarHistorico(id);
}

function excluirCliente(id){
  if(!confirm("Confirma exclusão do cliente e todo histórico?")) return;
  // remover também possíveis lançamentos no caixa associados? NÃO removemos automaticamente para manter histórico financeiro,
  // mas marcamos: caso queira, podemos filtrar por origemClienteId na exportação.
  clientes = clientes.filter(c => c.id !== id);
  salvarClientes();
  fecharHistorico();
}

/* ================== Export histórico CSV (cliente) ================== */
function baixarHistorico(id){
  const c = encontrarClientePorId(id);
  if(!c) return;
  let csv = 'tipo,data,info\n';
  c.historico.forEach(h => {
    const line = `${h.tipo},"${h.data.replace(/"/g,'""')}","${(h.info||'').replace(/"/g,'""')}"\n`;
    csv += line;
  });
  const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  const nomeArquivo = `historico_${c.nome.replace(/\s+/g,'_')}.csv`;
  a.download = nomeArquivo;
  document.body.appendChild(a);
  a.click();
  a.remove();
  URL.revokeObjectURL(url);
}

/* ================== Caixa: registrar entrada/saída manual ================== */
function entradaCaixa(){
  const desc = (descCaixaI.value || "").trim();
  const valor = Number(valorCaixaI.value);
  if(!desc || !valor || valor <= 0){ alert("Preencha descrição e valor válidos."); return; }
  caixa.push({ id: criarId(), tipo:'entrada', descricao: desc, valor, data: nowStr() });
  descCaixaI.value = ""; valorCaixaI.value = "";
  salvarCaixa();
  alert("Entrada registrada no caixa.");
}
function saidaCaixa(){
  const desc = (descCaixaI.value || "").trim();
  const valor = Number(valorCaixaI.value);
  if(!desc || !valor || valor <= 0){ alert("Preencha descrição e valor válidos."); return; }
  caixa.push({ id: criarId(), tipo:'saida', descricao: desc, valor, data: nowStr() });
  descCaixaI.value = ""; valorCaixaI.value = "";
  salvarCaixa();
  alert("Saída registrada no caixa.");
}

/* ================== Adicionar ao Caixa por cliente (botão na linha) ================== */
function adicionarAoCaixaPrompt(clienteId){
  const c = encontrarClientePorId(clienteId);
  if(!c) return;
  const valorStr = prompt(`Valor a adicionar ao caixa para ${c.nome}? (R$)`, String(VALOR_PADRAO_CORTE));
  if(!valorStr) return;
  const valor = Number(valorStr.replace(',', '.'));
  if(!valor || valor <= 0){ alert("Valor inválido."); return; }
  const desc = prompt("Descrição (opcional):", `Pagamento - ${c.nome}`) || `Pagamento - ${c.nome}`;
  caixa.push({ id: criarId(), tipo:'entrada', descricao: desc, valor, data: nowStr(), origemClienteId: c.id, origemClienteNome: c.nome });
  salvarCaixa();
  alert("Valor adicionado ao caixa.");
}

/* ================== Caixa: atualizar lista e totais ================== */
function atualizarCaixa(){
  const tbody = document.querySelector("#tabelaCaixa tbody");
  tbody.innerHTML = "";
  let totalEntradas = 0, totalSaidas = 0;
  caixa.forEach(item=>{
    if(item.tipo === 'entrada') totalEntradas += Number(item.valor);
    if(item.tipo === 'saida') totalSaidas += Number(item.valor);
    tbody.innerHTML += `<tr>
      <td>${item.tipo}</td>
      <td class="small">${item.descricao}${item.origemClienteNome ? ' — ' + item.origemClienteNome : ''}</td>
      <td>R$ ${formatMoney(item.valor)}</td>
      <td class="small">${item.data}</td>
      <td><button class="ghost" onclick="removerLancamentoCaixa('${item.id}')">Remover</button></td>
    </tr>`;
  });
  document.getElementById("totalEntradas").innerText = formatMoney(totalEntradas);
  document.getElementById("totalSaidas").innerText = formatMoney(totalSaidas);
  document.getElementById("saldoCaixa").innerText = formatMoney(totalEntradas - totalSaidas);
  // dashboard saldo
  document.getElementById("dashSaldo").innerText = formatMoney(totalEntradas - totalSaidas);
}

/* ================== Caixa: remover lançamento ================== */
function removerLancamentoCaixa(id){
  if(!confirm("Remover lançamento do caixa?")) return;
  caixa = caixa.filter(i => i.id !== id);
  salvarCaixa();
}

/* ================== Export Caixa CSV ================== */
function exportCaixaCSV(){
  if(!caixa || caixa.length === 0){ alert("Nenhum lançamento no caixa para exportar."); return; }
  let csv = 'tipo,descricao,valor,data\n';
  caixa.forEach(i=>{
    csv += `${i.tipo},"${(i.descricao||'').replace(/"/g,'""')}",${Number(i.valor).toFixed(2)},"${i.data.replace(/"/g,'""')}"\n`;
  });
  const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = `caixa_${new Date().toISOString().slice(0,10)}.csv`;
  document.body.appendChild(a);
  a.click();
  a.remove();
  URL.revokeObjectURL(url);
}

/* ================== Dashboard ================== */
function atualizarDashboard(){
  const totalClientes = clientes.length;
  const totalCortes = clientes.reduce((t,c)=>t+c.cortes,0);
  const totalRestantes = clientes.reduce((t,c)=>t + Math.max(0, c.plano - c.cortes),0);
  const totalEntradas = caixa.filter(i=>i.tipo==='entrada').reduce((t,i)=>t+Number(i.valor),0);
  const totalSaidas = caixa.filter(i=>i.tipo==='saida').reduce((t,i)=>t+Number(i.valor),0);
  const saldo = totalEntradas - totalSaidas;

  document.getElementById("dashClientes").innerText = totalClientes;
  document.getElementById("dashCortes").innerText = totalCortes;
  document.getElementById("dashRestantes").innerText = totalRestantes;
  document.getElementById("dashSaldo").innerText = formatMoney(saldo);
}

/* ================== Helpers: formatPhone ================== */
function formatPhone(p){
  const s = apenasDigitos(p);
  if(s.length === 11) return `(${s.slice(0,2)}) ${s.slice(2,7)}-${s.slice(7)}`;
  if(s.length === 10) return `(${s.slice(0,2)}) ${s.slice(2,6)}-${s.slice(6)}`;
  return s;
}

/* ================== Eventos DOM ================== */
btnAdd.addEventListener('click', adicionarCliente);
btnClear.addEventListener('click', limparForm);
btnRegistrar.addEventListener('click', () => {
  const id = clienteSelect.value;
  if(!id){ alert("Selecione um cliente antes de registrar corte."); return; }
  registrarCorte(id);
});
btnWhats.addEventListener('click', () => {
  const id = clienteSelect.value;
  if(!id){ alert("Selecione um cliente para enviar WhatsApp."); return; }
  enviarWhatsApp(id);
});
btnRenovar.addEventListener('click', () => {
  const id = clienteSelect.value;
  if(!id){ alert("Selecione um cliente para renovar plano."); return; }
  renovarPlanoPrompt(id);
});
btnHistorico.addEventListener('click', () => {
  const id = clienteSelect.value;
  if(!id){ alert("Selecione um cliente para ver histórico."); return; }
  mostrarHistorico(id);
});

/* ================== Remover / Export / Expor funções globalmente (para botões inline) ================== */
window.enviarWhatsApp = enviarWhatsApp;
window.mostrarHistorico = mostrarHistorico;
window.renovarPlanoPrompt = renovarPlanoPrompt;
window.editarClientePrompt = editarClientePrompt;
window.excluirCliente = excluirCliente;
window.baixarHistorico = baixarHistorico;
window.formatPhone = formatPhone;
window.adicionarAoCaixaPrompt = adicionarAoCaixaPrompt;
window.removerLancamentoCaixa = removerLancamentoCaixa;
window.exportCaixaCSV = exportCaixaCSV;

/* ================== Inicial atualizar ================== */
atualizarUI();
atualizarCaixa();
atualizarDashboard();

</script>
</body>
</html>




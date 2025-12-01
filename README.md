<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Controle de Cortes - Barbearia</title>
<style>
  :root{
    --bg:#0f0f10;
    --card:#141416;
    --muted:#cfcfcf;
    --accent:#0d6efd;
    --whats:#25d366;
    --danger:#d63333;
  }
  body{
    background:var(--bg);
    color:#fff;
    font-family:Inter, Arial, sans-serif;
    margin:0;
    padding:18px;
  }
  h1{ text-align:center; margin:8px 0 18px; font-size:20px; }
  .container{ max-width:980px; margin:0 auto; }
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
  table{ width:100%; border-collapse:collapse; margin-top:12px; font-size:14px; }
  th, td{ padding:10px; border-bottom:1px solid rgba(255,255,255,0.04); text-align:left; }
  th { color:var(--muted); font-size:13px; }
  .small{ font-size:12px; color:var(--muted); }
  .actions button{ margin-right:6px; }
  #historyCard{ margin-top:12px; }
  .empty{ text-align:center; color:var(--muted); padding:20px 0; }
  @media(max-width:820px){ .grid{ grid-template-columns: 1fr; } }
</style>
</head>
<body>
  <div class="container">
    <h1>Controle de Cortes - Barbearia</h1>

    <div class="grid">
      <div class="card">
        <h3>Novo Cliente</h3>
        <label for="nome">Nome</label>
        <input id="nome" placeholder="Nome do cliente">
        <label for="telefone">Telefone (somente números, ex: 11999998888)</label>
        <input id="telefone" placeholder="Telefone (opcional)">
        <label for="plano">Quantidade de cortes do plano</label>
        <input id="plano" type="number" min="1" placeholder="Ex: 10">
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

        <p class="small" style="margin-top:10px">Observação: o histórico guarda data e hora de cada corte. Se o cliente já terminou o plano, você será perguntado se deseja registrar cortes extras.</p>
      </div>
    </div>

    <div class="card" style="margin-top:12px">
      <h3>Clientes e Progresso</h3>
      <div id="tabelaWrapper"></div>
    </div>

    <div id="historyCard"></div>

  </div>

<script>
/* ====== Dados e armazenamento ====== */
const STORAGE_KEY = "barbearia_clientes_v2";
let clientes = JSON.parse(localStorage.getItem(STORAGE_KEY)) || [];

/* ====== Utilitários ====== */
function salvar(){
  localStorage.setItem(STORAGE_KEY, JSON.stringify(clientes));
}
function nowStr(){ return new Date().toLocaleString(); }
function apenasDigitos(s){ return (s||"").toString().replace(/\D+/g,''); }

/* ====== DOM ====== */
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

/* ====== Inicialização ====== */
atualizarUI();

/* ====== Funções principais ====== */
function criarId(){ return Date.now().toString(36) + Math.random().toString(36).slice(2,7); }

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
    historico: [] // array de {tipo:'corte'|'renovacao', data, info}
  };

  clientes.push(novo);
  salvar();
  limparForm();
  atualizarUI();
  alert("Cliente adicionado com sucesso.");
}

function limparForm(){
  nomeI.value=""; telI.value=""; planoI.value="";
}

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
          ${clientes.map(c=>{
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

function encontrarClientePorId(id){
  return clientes.find(c => c.id === id);
}

/* ====== Ações: registrar corte ====== */
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
    // registra como extra
  }

  const antes = c.cortes;
  c.cortes = c.cortes + 1;
  c.historico.push({ tipo:'corte', data: nowStr(), info:`corte registrado (${antes} → ${c.cortes})` });
  salvar();
  atualizarUI();
  mostrarHistorico(id);
}

/* ====== WhatsApp ====== */
function enviarWhatsApp(id){
  const c = encontrarClientePorId(id);
  if(!c) { alert("Cliente não encontrado."); return; }
  if(!c.telefone){
    const add = confirm("Cliente não tem telefone cadastrado. Deseja adicionar agora?");
    if(add){
      const t = prompt("Informe o telefone (somente números):", "");
      if(t){ c.telefone = apenasDigitos(t); salvar(); atualizarUI(); }
      else { return; }
    } else return;
  }
  // montar mensagem
  const faltam = Math.max(0, c.plano - c.cortes);
  const msg = `Olá ${c.nome}!%0A%0A` +
              `Seu controle de cortes está assim:%0A` +
              `• Plano total: ${c.plano} cortes%0A` +
              `• Cortes realizados: ${c.cortes}%0A` +
              `• Faltam: ${faltam}%0A%0A` +
              `Obrigado por escolher nossa barbearia! ✂️`;
  // usa wa.me com DDI 55 (Brasil). Se o número já tiver DDI, isso pode causar duplicação, mas normalmente usuário usa local.
  const telefone = c.telefone.replace(/^\+/, '').replace(/\s+/g,'');
  const url = `https://wa.me/55${telefone}?text=${msg}`;
  window.open(url, "_blank");
}

/* ====== Histórico ====== */
function mostrarHistorico(id){
  const c = encontrarClientePorId(id);
  if(!c){ alert("Cliente não encontrado."); return; }
  clienteSelect.value = id;
  // monta histórico
  let html = `<div class="card"><h3>Histórico — ${c.nome}</h3>`;
  html += `<p class="small">Plano: ${c.plano} | Cortes: ${c.cortes} | Telefone: ${c.telefone ? formatPhone(c.telefone) : '—'}</p>`;
  if(!c.historico || c.historico.length === 0){
    html += `<div class="empty">Nenhuma ação registrada ainda.</div>`;
  } else {
    html += `<table><thead><tr><th>Data / Hora</th><th>Tipo</th><th>Info</th></tr></thead><tbody>`;
    // mostrar do mais recente ao antigo
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

/* ====== Renovar plano (adicionar cortes) ====== */
function renovarPlanoPrompt(id){
  if(!id){
    const sel = clienteSelect.value;
    if(!sel){ alert("Selecione um cliente ou clique Renovar na linha do cliente."); return; }
    id = sel;
  }
  const c = encontrarClientePorId(id);
  if(!c) return;
  const q = prompt(`Adicionar quantos cortes ao plano de ${c.nome}?`, "5");
  const n = Number(q);
  if(!n || n <= 0){ alert("Valor inválido."); return; }
  c.plano = c.plano + n;
  c.historico.push({ tipo:'renovacao', data: nowStr(), info: `+${n} cortes (novo plano ${c.plano})` });
  salvar();
  atualizarUI();
  mostrarHistorico(id);
}

/* ====== Editar / Excluir ====== */
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
  // Se reduzir o plano para menos que cortes, mantém cortes, mas histórico anota
  const antesPlano = c.plano;
  c.plano = novoPlano;
  c.historico.push({ tipo:'edicao', data: nowStr(), info:`plano ${antesPlano} → ${c.plano}` });
  salvar();
  atualizarUI();
  mostrarHistorico(id);
}

function excluirCliente(id){
  if(!confirm("Confirma exclusão do cliente e todo histórico?")) return;
  clientes = clientes.filter(c => c.id !== id);
  salvar();
  atualizarUI();
  fecharHistorico();
}

/* ====== Helpers ====== */
function formatPhone(p){
  // formata números brasileiros simples (11 dígitos) -> (11) 9xxxx-xxxx
  const s = apenasDigitos(p);
  if(s.length === 11) return `(${s.slice(0,2)}) ${s.slice(2,7)}-${s.slice(7)}`;
  if(s.length === 10) return `(${s.slice(0,2)}) ${s.slice(2,6)}-${s.slice(6)}`;
  return s;
}

/* ====== Export histórico CSV ====== */
function baixarHistorico(id){
  const c = encontrarClientePorId(id);
  if(!c) return;
  let csv = 'tipo,data,info\\n';
  c.historico.forEach(h => {
    const line = `${h.tipo},"${h.data.replace(/"/g,'""')}","${(h.info||'').replace(/"/g,'""')}"\\n`;
    csv += line;
  });
  const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  const nomeArquivo = `historico_${c.nome.replace(/\\s+/g,'_')}.csv`;
  a.download = nomeArquivo;
  document.body.appendChild(a);
  a.click();
  a.remove();
  URL.revokeObjectURL(url);
}

/* ====== Eventos DOM ====== */
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

/* ====== Expor algumas funções ao global para uso nos botões inline da tabela ====== */
window.enviarWhatsApp = enviarWhatsApp;
window.mostrarHistorico = mostrarHistorico;
window.renovarPlanoPrompt = renovarPlanoPrompt;
window.editarClientePrompt = editarClientePrompt;
window.excluirCliente = excluirCliente;
window.baixarHistorico = baixarHistorico;
window.formatPhone = formatPhone;

</script>
</body>
</html>




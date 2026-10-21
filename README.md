<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Cripto Privado — Ganhos e Perdas</title>
<script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.1/dist/chart.umd.min.js"></script>
<style>
:root{--bg:#0b1020;--card:#141a2e;--card2:#1b2340;--text:#e8ecf8;--muted:#9aa4c2;--green:#22c55e;--red:#ef4444;--blue:#3b82f6}
*{box-sizing:border-box;margin:0;padding:0}
body{font-family:Inter,system-ui,Arial,sans-serif;background:var(--bg);color:var(--text);min-height:100vh}
.hidden{display:none!important}
#login{min-height:100vh;display:flex;align-items:center;justify-content:center}
.login-box{background:var(--card);padding:32px;border-radius:16px;border:1px solid #26304f;width:320px;text-align:center}
.login-box h2{margin-bottom:18px}
input,select{width:100%;padding:12px;border-radius:8px;border:1px solid #2c3760;background:var(--card2);color:var(--text);margin-bottom:12px}
button{padding:12px 18px;border:none;border-radius:8px;background:var(--blue);color:#fff;font-weight:700;cursor:pointer;width:100%}
.erro{color:var(--red);font-size:13px;margin-top:10px}
header{display:flex;justify-content:space-between;align-items:center;padding:16px 24px;background:#111a33;border-bottom:1px solid #26304f}
header h1{font-size:18px}
header button{width:auto;background:#2c3760;padding:8px 14px}
.container{max-width:1150px;margin:auto;padding:22px}
.grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(200px,1fr));gap:14px;margin-bottom:20px}
.card{background:var(--card);border:1px solid #26304f;border-radius:14px;padding:16px}
.kpi h3{font-size:11px;color:var(--muted);text-transform:uppercase;margin-bottom:6px}
.kpi .value{font-size:24px;font-weight:800}
.pos{color:var(--green)}.neg{color:var(--red)}
.charts{display:grid;grid-template-columns:1fr 1fr;gap:14px;margin-bottom:20px}
@media(max-width:850px){.charts{grid-template-columns:1fr}}
canvas{max-height:280px}
form{display:grid;grid-template-columns:repeat(auto-fit,minmax(140px,1fr));gap:10px;align-items:end}
label{font-size:11px;color:var(--muted);display:block;margin-bottom:4px}
table{width:100%;border-collapse:collapse;margin-top:10px}
th,td{padding:9px;border-bottom:1px solid #26304f;font-size:13px;text-align:left}
th{color:var(--muted);font-size:11px;text-transform:uppercase}
.tag{padding:2px 8px;border-radius:999px;font-size:11px;font-weight:700}
.compra{background:rgba(34,197,94,.15);color:var(--green)}
.venda{background:rgba(239,68,68,.15);color:var(--red)}
.excluir{background:#2c3760;padding:5px 9px;font-size:11px;width:auto}
.filtros{display:flex;gap:8px;flex-wrap:wrap;margin-bottom:10px}
.filtros select{max-width:170px;margin:0}
</style>
</head>
<body>

<div id="login">
  <div class="login-box">
    <h2>🔒 Acesso privado</h2>
    <input type="password" id="pin" placeholder="Digite seu PIN" inputmode="numeric">
    <button onclick="entrar()">Entrar</button>
    <div class="erro" id="erro"></div>
  </div>
</div>

<div id="app" class="hidden">
  <header>
    <h1>💰 Cripto Privado</h1>
    <button onclick="sair()">Sair</button>
  </header>

  <div class="container">
    <div class="grid">
      <div class="card kpi"><h3>Ganho hoje</h3><div class="value pos" id="ganhoHoje">R$ 0,00</div></div>
      <div class="card kpi"><h3>Perda hoje</h3><div class="value neg" id="perdaHoje">R$ 0,00</div></div>
      <div class="card kpi"><h3>Ganho no mês</h3><div class="value pos" id="ganhoMes">R$ 0,00</div></div>
      <div class="card kpi"><h3>Perda no mês</h3><div class="value neg" id="perdaMes">R$ 0,00</div></div>
      <div class="card kpi"><h3>Resultado do ano</h3><div class="value" id="resultadoAno">R$ 0,00</div></div>
      <div class="card kpi"><h3>Patrimônio estimado</h3><div class="value" id="patrimonio">R$ 0,00</div></div>
    </div>

    <div class="card" style="margin-bottom:20px">
      <h3 style="margin-bottom:12px">Nova operação</h3>
      <form id="form">
        <div><label>Data</label><input type="date" id="data" required></div>
        <div><label>Tipo</label>
          <select id="tipo">
            <option value="compra">Compra</option>
            <option value="venda">Venda</option>
          </select>
        </div>
        <div><label>Criptomoeda</label><input id="moeda" placeholder="BTC, ETH, SOL..." required></div>
        <div><label>Quantidade</label><input type="number" step="0.00000001" id="qtd" required></div>
        <div><label>Preço unitário R$</label><input type="number" step="0.01" id="preco" required></div>
        <div><label>Taxas R$</label><input type="number" step="0.01" id="taxa" value="0"></div>
        <div><label>Preço atual R$</label><input type="number" step="0.01" id="atual" placeholder="opcional"></div>
        <button type="submit">Salvar</button>
      </form>
    </div>

    <div class="charts">
      <div class="card"><h3 style="margin-bottom:10px">Ganhos e perdas por dia</h3><canvas id="graficoDia"></canvas></div>
      <div class="card"><h3 style="margin-bottom:10px">Resultado mensal</h3><canvas id="graficoMes"></canvas></div>
    </div>

    <div class="card">
      <div style="display:flex;justify-content:space-between;flex-wrap:wrap;gap:10px">
        <h3>Histórico</h3>
        <div class="filtros">
          <select id="fMoeda"><option value="">Todas as moedas</option></select>
          <select id="fPeriodo">
            <option value="">Todo período</option>
            <option value="hoje">Hoje</option>
            <option value="mes">Este mês</option>
            <option value="ano">Este ano</option>
          </select>
        </div>
      </div>

      <div style="overflow-x:auto">
        <table>
          <thead>
            <tr>
              <th>Data</th><th>Tipo</th><th>Moeda</th><th>Qtd</th><th>Preço</th>
              <th>Total</th><th>Taxa</th><th>Lucro/Prejuízo</th><th></th>
            </tr>
          </thead>
          <tbody id="tbody"></tbody>
        </table>
      </div>
    </div>
  </div>
</div>

<script>
const SENHA = "1234";
let ops = JSON.parse(localStorage.getItem("cripto_privado_ops") || "[]");

const brl = v => (v < 0 ? "-" : "") + "R$ " +
  Math.abs(v).toLocaleString("pt-BR", {minimumFractionDigits:2, maximumFractionDigits:2});

const salvar = () => localStorage.setItem("cripto_privado_ops", JSON.stringify(ops));

function entrar(){
  if(document.getElementById("pin").value === SENHA){
    document.getElementById("login").classList.add("hidden");
    document.getElementById("app").classList.remove("hidden");
    document.getElementById("data").value = new Date().toISOString().slice(0,10);
    render();
  } else {
    document.getElementById("erro").textContent = "PIN incorreto.";
  }
}

function sair(){
  document.getElementById("app").classList.add("hidden");
  document.getElementById("login").classList.remove("hidden");
  document.getElementById("pin").value = "";
  document.getElementById("erro").textContent = "";
}

document.getElementById("pin").addEventListener("keydown", e=>{
  if(e.key === "Enter") entrar();
});

document.getElementById("form").addEventListener("submit", e=>{
  e.preventDefault();

  const o = {
    id: Date.now(),
    data: document.getElementById("data").value,
    tipo: document.getElementById("tipo").value,
    moeda: document.getElementById("moeda").value.toUpperCase(),
    qtd: +document.getElementById("qtd").value,
    preco: +document.getElementById("preco").value,
    taxa: +document.getElementById("taxa").value,
    atual: document.getElementById("atual").value ? +document.getElementById("atual").value : null
  };

  o.total = o.qtd * o.preco + (o.tipo === "compra" ? o.taxa : -o.taxa);

  if(o.tipo === "compra"){
    const anteriores = ops.filter(x => x.moeda === o.moeda);
    const qtdAnt = anteriores.reduce((s,x)=> s + (x.tipo === "compra" ? x.qtd : -x.qtd), 0);
    const custoAnt = anteriores.reduce((s,x)=> s + (x.tipo === "compra" ? x.qtd*x.preco + x.taxa : -(x.qtd*x.preco)), 0);

    o.custoMedio = qtdAnt + o.qtd > 0
      ? (custoAnt + o.qtd*o.preco + o.taxa) / (qtdAnt + o.qtd)
      : o.preco;

  } else {
    const compras = ops.filter(x => x.moeda === o.moeda && x.tipo === "compra");
    const cm = compras.length
      ? compras[compras.length-1].custoMedio || compras[compras.length-1].preco
      : o.preco;

    o.custoMedio = cm;
    o.resultado = (o.preco - cm) * o.qtd - o.taxa;
  }

  ops.push(o);
  salvar();
  render();

  e.target.reset();
  document.getElementById("data").value = new Date().toISOString().slice(0,10);
  document.getElementById("taxa").value = 0;
});

function filtrar(){
  const moeda = document.getElementById("fMoeda").value;
  const periodo = document.getElementById("fPeriodo").value;
  const hoje = new Date().toISOString().slice(0,10);
  const mes = hoje.slice(0,7);
  const ano = hoje.slice(0,4);

  return ops.filter(o=>{
    if(moeda && o.moeda !== moeda) return false;
    if(periodo === "hoje" && o.data !== hoje) return false;
    if(periodo === "mes" && !o.data.startsWith(mes)) return false;
    if(periodo === "ano" && !o.data.startsWith(ano)) return false;
    return true;
  });
}

function ganhosPerdas(inicio, fim){
  const vendas = ops.filter(o=>{
    if(o.tipo !== "venda") return false;
    if(inicio && o.data < inicio) return false;
    if(fim && o.data > fim) return false;
    return true;
  });

  let ganho = 0;
  let perda = 0;

  vendas.forEach(o=>{
    const r = o.resultado || 0;
    if(r >= 0) ganho += r;
    else perda += Math.abs(r);
  });

  return {ganho, perda};
}

function render(){
  const hoje = new Date().toISOString().slice(0,10);
  const mes = hoje.slice(0,7);
  const ano = hoje.slice(0,4);

  const h = ganhosPerdas(hoje, hoje);
  const m = ganhosPerdas(mes + "-01", mes + "-31");
  const a = ganhosPerdas(ano + "-01-01", ano + "-12-31");

  document.getElementById("ganhoHoje").textContent = brl(h.ganho);
  document.getElementById("perdaHoje").textContent = brl(h.perda);
  document.getElementById("ganhoMes").textContent = brl(m.ganho);
  document.getElementById("perdaMes").textContent = brl(m.perda);

  const resultadoAno = a.ganho - a.perda;
  const elAno = document.getElementById("resultadoAno");
  elAno.textContent = brl(resultadoAno);
  elAno.className = "value " + (resultadoAno >= 0 ? "pos" : "neg");

  let patrimonio = 0;
  const moedas = [...new Set(ops.map(o=>o.moeda))];

  moedas.forEach(moeda=>{
    const qtd = ops
      .filter(o=>o.moeda === moeda)
      .reduce((s,o)=> s + (o.tipo === "compra" ? o.qtd : -o.qtd), 0);

    if(qtd <= 0) return;

    const ultimo = [...ops.filter(o=>o.moeda === moeda)]
      .reverse()
      .find(o=>o.atual || o.tipo === "compra");

    const preco = ultimo?.atual || ultimo?.preco || 0;
    patrimonio += qtd * preco;
  });

  document.getElementById("patrimonio").textContent = brl(patrimonio);

  const sel = document.getElementById("fMoeda");
  const valorAtual = sel.value;
  sel.innerHTML = '<option value="">Todas as moedas</option>' +
    moedas.map(x=>`<option ${x===valorAtual?"selected":""}>${x}</option>`).join("");

  const lista = filtrar();
  document.getElementById("tbody").innerHTML = lista.map(o=>`
    <tr>
      <td>${o.data.split("-").reverse().join("/")}</td>
      <td><span class="tag ${o.tipo}">${o.tipo === "compra" ? "Compra" : "Venda"}</span></td>
      <td><b>${o.moeda}</b></td>
      <td>${o.qtd}</td>
      <td>${brl(o.preco)}</td>
      <td>${brl(o.total)}</td>
      <td>${brl(o.taxa)}</td>
      <td class="${(o.resultado || 0) >= 0 ? "pos" : "neg"}">
        ${o.tipo === "venda" ? brl(o.resultado || 0) : "—"}
      </td>
      <td><button class="excluir" onclick="excluir(${o.id})">Excluir</button></td>
    </tr>
  `).join("");

  criarGraficoDia();
  criarGraficoMes();
}

function criarGraficoDia(){
  const vendas = ops.filter(o=>o.tipo === "venda");
  const porDia = {};

  vendas.forEach(o=>{
    const r = o.resultado || 0;
    porDia[o.data] = (porDia[o.data] || 0) + r;
  });

  const dias = Object.keys(porDia).sort();

  new Chart(document.getElementById("graficoDia"), {
    type: "bar",
    data: {
      labels: dias.map(d=>d.split("-").reverse().join("/")),
      datasets: [{
        label: "Resultado diário",
        data: dias.map(d=>porDia[d]),
        backgroundColor: dias.map(d=>porDia[d] >= 0 ? "#22c55e" : "#ef4444")
      }]
    },
    options:{
      plugins:{legend:{labels:{color:"#9aa4c2"}}},
      scales:{
        x:{ticks:{color:"#9aa4c2"}},
        y:{ticks:{color:"#9aa4c2"}}
      }
    }
  });
}

function criarGraficoMes(){
  const vendas = ops.filter(o=>o.tipo === "venda");
  const porMes = {};

  vendas.forEach(o=>{
    const chave = o.data.slice(0,7);
    porMes[chave] = (porMes[chave] || 0) + (o.resultado || 0);
  });

  const meses = Object.keys(porMes).sort();

  new Chart(document.getElementById("graficoMes"), {
    type: "bar",
    data: {
      labels: meses.map(m=>{
        const [ano, mes] = m.split("-");
        return mes + "/" + ano;
      }),
      datasets: [{
        label: "Resultado mensal",
        data: meses.map(m=>porMes[m]),
        backgroundColor: meses.map(m=>porMes[m] >= 0 ? "#3b82f6" : "#ef4444")
      }]
    },
    options:{
      plugins:{legend:{labels:{color:"#9aa4c2"}}},
      scales:{
        x:{ticks:{color:"#9aa4c2"}},
        y:{ticks:{color:"#9aa4c2"}}
      }
    }
  });
}

function excluir(id){
  ops = ops.filter(o=>o.id !== id);
  salvar();
  render();
}

document.getElementById("fMoeda").addEventListener("change", render);
document.getElementById("fPeriodo").addEventListener("change", render);
</script>

</body>
</html>

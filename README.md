html.index
<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>CryptoLedger — Contabilidade de Criptomoedas</title>
<script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.1/dist/chart.umd.min.js"></script>
<style>
:root{--bg:#0b1020;--card:#141a2e;--card2:#1b2340;--text:#e8ecf8;--muted:#9aa4c2;--green:#22c55e;--red:#ef4444;--blue:#3b82f6;--yellow:#f59e0b}
*{box-sizing:border-box;margin:0;padding:0}
body{font-family:Inter,system-ui,Arial,sans-serif;background:var(--bg);color:var(--text);min-height:100vh}
header{display:flex;justify-content:space-between;align-items:center;padding:18px 28px;background:linear-gradient(90deg,#111a33,#0b1020);border-bottom:1px solid #26304f}
h1{font-size:20px}
h1 span{color:var(--blue)}
.container{max-width:1200px;margin:0 auto;padding:24px}
.grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(220px,1fr));gap:16px;margin-bottom:24px}
.card{background:var(--card);border:1px solid #26304f;border-radius:14px;padding:18px}
.kpi h3{font-size:12px;color:var(--muted);text-transform:uppercase;letter-spacing:.08em;margin-bottom:8px}
.kpi .value{font-size:26px;font-weight:700}
.pos{color:var(--green)}.neg{color:var(--red)}.neutral{color:var(--text)}
.charts{display:grid;grid-template-columns:1fr 1fr;gap:16px;margin-bottom:24px}
@media(max-width:900px){.charts{grid-template-columns:1fr}}
canvas{max-height:300px}
form{display:grid;grid-template-columns:repeat(auto-fit,minmax(150px,1fr));gap:12px;align-items:end}
label{font-size:12px;color:var(--muted);display:block;margin-bottom:5px}
input,select{width:100%;padding:10px;border-radius:8px;border:1px solid #2c3760;background:var(--card2);color:var(--text)}
button{padding:10px 16px;border:none;border-radius:8px;background:var(--blue);color:#fff;font-weight:600;cursor:pointer}
button.secondary{background:#2c3760}
table{width:100%;border-collapse:collapse;margin-top:10px}
th,td{padding:10px;text-align:left;border-bottom:1px solid #26304f;font-size:14px}
th{color:var(--muted);font-size:12px;text-transform:uppercase}
.tag{padding:3px 8px;border-radius:999px;font-size:12px;font-weight:600}
.tag.buy{background:rgba(34,197,94,.15);color:var(--green)}
.tag.sell{background:rgba(239,68,68,.15);color:var(--red)}
.actions button{padding:5px 10px;font-size:12px;background:#2c3760}
.filters{display:flex;gap:10px;flex-wrap:wrap;margin-bottom:12px}
.filters select,.filters input{max-width:180px}
.empty{color:var(--muted);text-align:center;padding:24px}
</style>
</head>
<body>
<header>
  <h1>Crypto<span>Ledger</span> — Contabilidade de Cripto</h1>
  <button class="secondary" onclick="exportCSV()">Exportar CSV</button>
</header>

<div class="container">
  <div class="grid">
    <div class="card kpi"><h3>Patrimônio atual</h3><div class="value" id="kpiPatrimonio">R$ 0,00</div></div>
    <div class="card kpi"><h3>Ganho / Perda realizado</h3><div class="value" id="kpiRealizado">R$ 0,00</div></div>
    <div class="card kpi"><h3>Resultado hoje</h3><div class="value" id="kpiHoje">R$ 0,00</div></div>
    <div class="card kpi"><h3>Resultado no mês</h3><div class="value" id="kpiMes">R$ 0,00</div></div>
    <div class="card kpi"><h3>Resultado no ano</h3><div class="value" id="kpiAno">R$ 0,00</div></div>
  </div>

  <div class="card" style="margin-bottom:24px">
    <h3 style="margin-bottom:14px">Registrar operação</h3>
    <form id="form">
      <div><label>Data</label><input type="date" id="data" required></div>
      <div><label>Tipo</label><select id="tipo"><option value="compra">Compra</option><option value="venda">Venda</option></select></div>
      <div><label>Criptomoeda</label><input id="moeda" placeholder="BTC, ETH, SOL..." required></div>
      <div><label>Quantidade</label><input type="number" step="0.00000001" id="qtd" required></div>
      <div><label>Preço unitário (R$)</label><input type="number" step="0.01" id="preco" required></div>
      <div><label>Taxas (R$)</label><input type="number" step="0.01" id="taxa" value="0"></div>
      <div><label>Preço atual (R$)</label><input type="number" step="0.01" id="atual" placeholder="opcional"></div>
      <button type="submit">Adicionar</button>
    </form>
  </div>

  <div class="charts">
    <div class="card"><h3 style="margin-bottom:12px">Evolução do resultado</h3><canvas id="chartLinha"></canvas></div>
    <div class="card"><h3 style="margin-bottom:12px">Distribuição da carteira</h3><canvas id="chartPizza"></canvas></div>
  </div>

  <div class="card">
    <div style="display:flex;justify-content:space-between;align-items:center;flex-wrap:wrap;gap:10px">
      <h3>Histórico de operações</h3>
      <div class="filters">
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
    <div class="empty" id="empty" style="display:none">Nenhuma operação registrada ainda.</div>
  </div>
</div>

<script>
let ops = JSON.parse(localStorage.getItem('crypto_ops') || '[]');

const brl = v => (v < 0 ? '-' : '') + 'R$ ' + Math.abs(v).toLocaleString('pt-BR', {
  minimumFractionDigits: 2,
  maximumFractionDigits: 2
});

const save = () => localStorage.setItem('crypto_ops', JSON.stringify(ops));

document.getElementById('data').value = new Date().toISOString().slice(0, 10);

document.getElementById('form').addEventListener('submit', e => {
  e.preventDefault();

  const o = {
    id: Date.now(),
    data: document.getElementById('data').value,
    tipo: document.getElementById('tipo').value,
    moeda: document.getElementById('moeda').value.toUpperCase(),
    qtd: +document.getElementById('qtd').value,
    preco: +document.getElementById('preco').value,
    taxa: +document.getElementById('taxa').value,
    atual: document.getElementById('atual').value ? +document.getElementById('atual').value : null
  };

  o.total = o.qtd * o.preco + (o.tipo === 'compra' ? o.taxa : -o.taxa);

  if (o.tipo === 'compra') {
    const anteriores = ops.filter(x => x.moeda === o.moeda);
    const qtdAnt = anteriores.reduce((s, x) => s + (x.tipo === 'compra' ? x.qtd : -x.qtd), 0);
    const custoAnt = anteriores.reduce((s, x) => s + (x.tipo === 'compra' ? x.qtd * x.preco + x.taxa : -(x.qtd * x.preco)), 0);

    o.custoMedio = qtdAnt + o.qtd > 0
      ? (custoAnt + o.qtd * o.preco + o.taxa) / (qtdAnt + o.qtd)
      : o.preco;

  } else {
    const anteriores = ops.filter(x => x.moeda === o.moeda && x.tipo === 'compra');
    const custoMedio = anteriores.length
      ? anteriores[anteriores.length - 1].custoMedio || anteriores[anteriores.length - 1].preco
      : o.preco;

    o.custoMedio = custoMedio;
    o.resultado = (o.preco - custoMedio) * o.qtd - o.taxa;
  }

  ops.push(o);
  save();
  render();

  e.target.reset();
  document.getElementById('data').value = new Date().toISOString().slice(0, 10);
  document.getElementById('taxa').value = 0;
});

function filtrar() {
  const m = document.getElementById('fMoeda').value;
  const p = document.getElementById('fPeriodo').value;
  const hoje = new Date().toISOString().slice(0, 10);
  const mes = hoje.slice(0, 7);
  const ano = hoje.slice(0, 4);

  return ops.filter(o => {
    if (m && o.moeda !== m) return false;
    if (p === 'hoje' && o.data !== hoje) return false;
    if (p === 'mes' && !o.data.startsWith(mes)) return false;
    if (p === 'ano' && !o.data.startsWith(ano)) return false;
    return true;
  });
}

function resultadoPeriodo(inicio) {
  return ops
    .filter(o => o.tipo === 'venda' && o.data >= inicio)
    .reduce((s, o) => s + (o.resultado || 0), 0);
}

function render() {
  const fs = filtrar();
  const tbody = document.getElementById('tbody');

  tbody.innerHTML = fs.map(o => `
    <tr>
      <td>${o.data.split('-').reverse().join('/')}</td>
      <td><span class="tag ${o.tipo}">${o.tipo === 'compra' ? 'Compra' : 'Venda'}</span></td>
      <td><b>${o.moeda}</b></td>
      <td>${o.qtd}</td>
      <td>${brl(o.preco)}</td>
      <td>${brl(o.total)}</td>
      <td>${brl(o.taxa)}</td>
      <td class="${(o.resultado || 0) >= 0 ? 'pos' : 'neg'}">
        ${o.tipo === 'venda' ? brl(o.resultado || 0) : '—'}
      </td>
      <td><button onclick="del(${o.id})">Excluir</button></td>
    </tr>
  `).join('');

  document.getElementById('empty').style.display = fs.length ? 'none' : 'block';

  const compras = ops.filter(o => o.tipo === 'compra');
  const vendas = ops.filter(o => o.tipo === 'venda');

  const investido = compras.reduce((s, o) => s + o.qtd * o.preco + o.taxa, 0);
  const vendido = vendas.reduce((s, o) => s + o.qtd * o.preco - o.taxa, 0);
  const realizado = vendas.reduce((s, o) => s + (o.resultado || 0), 0);

  let patrimonio = 0;
  const moedas = [...new Set(ops.map(o => o.moeda))];

  moedas.forEach(m => {
    const qtd = ops
      .filter(o => o.moeda === m)
      .reduce((s, o) => s + (o.tipo === 'compra' ? o.qtd : -o.qtd), 0);

    if (qtd <= 0) return;

    const ult = [...ops.filter(o => o.moeda === m)]
      .reverse()
      .find(o => o.atual || o.tipo === 'compra');

    const preco = ult?.atual || ult?.preco || 0;
    patrimonio += qtd * preco;
  });

  document.getElementById('kpiPatrimonio').textContent = brl(patrimonio);

  const kr = document.getElementById('kpiRealizado');
  kr.textContent = brl(realizado);
  kr.className = 'value ' + (realizado >= 0 ? 'pos' : 'neg');

  const hoje = new Date().toISOString().slice(0, 10);
  const mes = hoje.slice(0, 7);
  const ano = hoje.slice(0, 4);

  const rh = resultadoPeriodo(hoje);
  const rm = resultadoPeriodo(mes + '-01');
  const ra = resultadoPeriodo(ano + '-01-01');

  const set = (id, v) => {
    const el = document.getElementById(id);
    el.textContent = brl(v);
    el.className = 'value ' + (v >= 0 ? 'pos' : 'neg');
  };

  set('kpiHoje', rh);
  set('kpiMes', rm);
  set('kpiAno', ra);

  const sel = document.getElementById('fMoeda');
  const atual = sel.value;

  sel.innerHTML = '<option value="">Todas as moedas</option>' +
    moedas.map(m => `<option ${m === atual ? 'selected' : ''}>${m}</option>`).join('');

  const porData = {};
  vendas.forEach(o => {
    porData[o.data] = (porData[o.data] || 0) + (o.resultado || 0);
  });

  const datas = Object.keys(porData).sort();
  let acc = 0;

  const serie = datas.map(d => {
    acc += porData[d];
    return +acc.toFixed(2);
  });

  new Chart(document.getElementById('chartLinha'), {
    type: 'line',
    data: {
      labels: datas.map(d => d.split('-').reverse().join('/')),
      datasets: [{
        label: 'Resultado acumulado (R$)',
        data: serie,
        borderColor: '#3b82f6',
        backgroundColor: 'rgba(59,130,246,.15)',
        fill: true,
        tension: .3
      }]
    },
    options: {
      plugins: { legend: { labels: { color: '#9aa4c2' } } },
      scales: {
        x: { ticks: { color: '#9aa4c2' } },
        y: { ticks: { color: '#9aa4c2' } }
      }
    }
  });

  const dist = moedas.map(m => {
    const qtd = ops
      .filter(o => o.moeda === m)
      .reduce((s, o) => s + (o.tipo === 'compra' ? o.qtd : -o.qtd), 0);

    const ult = [...ops.filter(o => o.moeda === m)]
      .reverse()
      .find(o => o.atual || o.tipo === 'compra');

    return {
      m,
      v: Math.max(0, qtd * (ult?.atual || ult?.preco || 0))
    };
  }).filter(x => x.v > 0);

  new Chart(document.getElementById('chartPizza'), {
    type: 'doughnut',
    data: {
      labels: dist.map(x => x.m),
      datasets: [{
        data: dist.map(x => x.v),
        backgroundColor: ['#3b82f6', '#22c55e', '#f59e0b', '#ef4444', '#a855f7', '#06b6d4', '#f472b6']
      }]
    },
    options: {
      plugins: { legend: { labels: { color: '#9aa4c2' } } }
    }
  });
}

function del(id) {
  ops = ops.filter(o => o.id !== id);
  save();
  render();
}

function exportCSV() {
  const rows = [
    ['Data', 'Tipo', 'Moeda', 'Quantidade', 'Preco', 'Total', 'Taxa', 'Resultado']
  ];

  filtrar().forEach(o => {
    rows.push([
      o.data,
      o.tipo,
      o.moeda,
      o.qtd,
      o.preco,
      o.total.toFixed(2),
      o.taxa,
      (o.resultado || 0).toFixed(2)
    ]);
  });

  const csv = rows.map(r => r.join(';')).join('
');
  const a = document.createElement('a');

  a.href = URL.createObjectURL(new Blob(['﻿' + csv], { type: 'text/csv' }));
  a.download = 'cryptoledger.csv';
  a.click();
}

render();
</script>
</body>
</html>
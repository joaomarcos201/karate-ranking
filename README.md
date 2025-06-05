<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Ranking de Karatê</title>
  <style>
    body { font-family: Arial, sans-serif; padding: 20px; background: #f9f9f9; }
    h1, h2, h3 { color: #222; }
    table { width: 100%; border-collapse: collapse; margin-bottom: 30px; }
    th, td { border: 1px solid #ccc; padding: 8px; text-align: left; }
    th { background: #eee; }
    input, select { padding: 5px; margin: 5px; }
    button { padding: 5px 10px; margin: 5px 5px 15px 0; cursor: pointer; }
    .progress-container { margin: 20px 0; }
    .progress-bar { height: 20px; background: #4caf50; text-align: center; color: white; }
    .list-box { background: white; border: 1px solid #ccc; padding: 10px; margin: 5px 0; max-height: 200px; overflow-y: auto; }
    .list-box h4 { margin: 5px 0; }
  </style>
</head>
<body>
  <h1>Ranking de Karatê</h1>

  <h2>Adicionar Resultado</h2>
  <form id="resultForm">
    <input type="text" id="etapa" placeholder="Nome da Etapa" required />
    <select id="categoria" required><option value="">Selecione a Categoria</option></select>
    <input type="text" id="atleta" placeholder="Nome do Atleta" required />
    <input type="text" id="associacao" placeholder="Associação" required />
    <select id="colocacao" required>
      <option value="">Selecione a colocação</option>
      <option value="ouro">🥇 Ouro</option>
      <option value="prata">🥈 Prata</option>
      <option value="bronze">🥉 Bronze</option>
    </select>
    <button type="submit">Adicionar</button>
  </form>

  <div class="progress-container">
    <h2>Progresso das Categorias Executadas</h2>
    <div style="border: 1px solid #ccc; width: 100%;">
      <div id="progressBar" class="progress-bar" style="width: 0%;">0%</div>
    </div>
    <p id="progressText"></p>
    <div class="list-box">
      <h4>✅ Concluídas:</h4>
      <ul id="listaConcluidas"></ul>
      <h4>⏳ Faltando:</h4>
      <ul id="listaFaltando"></ul>
    </div>
  </div>

  <h2>Filtrar por Categoria</h2>
  <select id="filtroCategoria"><option value="">Todas as Categorias</option></select>

  <h2>Bônus / Penalidades por Associação</h2>
  <form id="bonusForm">
    <select id="bonusAssociacao" required><option value="">Selecione Associação</option></select>
    <input type="number" id="bonusValor" placeholder="Valor (+ ou -)" required />
    <button type="submit">Aplicar Bônus/Penalidade</button>
  </form>

  <button onclick="exportarExcel()">📤 Exportar para Excel</button>
  <button onclick="salvarDados()">💾 Salvar manualmente</button>
  <button onclick="limparDados()">🗑 Limpar dados</button>

  <h2>Ranking por Categoria e Atleta</h2>
  <div id="rankingCategorias"></div>

  <h2>Ranking Geral por Associação</h2>
  <table id="rankingAssociacoes">
    <thead>
      <tr>
        <th>Associação</th>
        <th>Pontos</th>
        <th>🥇</th>
        <th>🥈</th>
        <th>🥉</th>
        <th>Bônus/Penalidades</th>
      </tr>
    </thead>
    <tbody></tbody>
  </table>
<script src="https://cdn.sheetjs.com/xlsx-latest/package/dist/xlsx.full.min.js"></script>
<script>
  const categoriasOficiais = [
    "1 - Mirim A - KATA", "2 - Mirim B - KATA", "3 - Mirim C 1 - KATA", "4 - Mirim C 2 - KATA", "6 - Mirim C 3 - KATA",
    "7 - Infantil A - KATA", "8 - Infantil B - KATA", "9 - Infanto-Juvenil A - KATA", "10 - Infanto-Juvenil B - KATA",
    "11 - Juvenil A - KATA", "12 - Juvenil B - KATA", "13 - Adulto A - KATA", "14 - Adulto B - KATA", "15 - Adulto C - KATA",
    "16 - Master A 1 - KATA", "17 - Master A 2 - KATA", "18 - Master B 1 - KATA", "19 - Master B 2 - KATA", "20 - Master C - KATA",
    "21 - Mirim A - KATA", "22 - Mirim B - KATA", "23 - Mirim C 1 - KATA", "24 - Mirim C 2 - KATA", "25 - Mirim C 3 - KATA",
    "26 - Infantil A - KATA", "27 - Infantil B - KATA", "28 - Infanto-Juvenil A - KATA", "29 - Infanto-Juvenil B - KATA",
    "30 - Infanto-Juvenil C - KATA", "31 - Juvenil A - KATA", "32 - Juvenil B - KATA", "33 - Juvenil C - KATA",
    "34 - Adulto A - KATA", "35 - Adulto B - KATA", "36 - Adulto C - KATA", "37 - Master A 1 - KATA", "38 - Master A 2 - KATA",
    "39 - Master B 1 - KATA", "40 - Master B 2 - KATA", "41 - Master C - KATA",
    "42 - Mirim A - KUMITE", "43 - Mirim B - KUMITE", "44 - Mirim C 1 - KUMITE", "45 - Mirim C 2 - KUMITE",
    "46 - Mirim C 3 - KUMITE", "47 - Mirim C 4 - KUMITE", "48 - Infantil A - KUMITE", "49 - Infantil B - KUMITE",
    "50 - Infanto-Juvenil A 1 - KUMITE", "51 - Infanto-Juvenil A 2 - KUMITE", "52 - Infanto-Juvenil B - KUMITE",
    "53 - Juvenil A - KUMITE", "54 - Juvenil B 1 - KUMITE", "55 - Juvenil B 2 - KUMITE", "56 - Adulto A - KUMITE",
    "57 - Adulto B - KUMITE", "58 - Adulto C 1 - KUMITE", "59 - Adulto C 2 - KUMITE", "60 - Master A 1 - KUMITE",
    "61 - Master A 2 - KUMITE", "62 - Master B 1 - KUMITE", "63 - Master B 2 - KUMITE", "64 - Master B 3 - KUMITE",
    "65 - Mirim A - KUMITE", "66 - Mirim B - KUMITE", "67 - Mirim C 1 - KUMITE", "68 - Mirim C 2 - KUMITE",
    "69 - Mirim C 3 - KUMITE", "70 - Mirim C 4 - KUMITE", "71 - Infantil A 1 - KUMITE", "72 - Infantil A 2 - KUMITE",
    "73 - Infantil B 1 - KUMITE", "74 - Infantil B 2 - KUMITE", "75 - Infanto-Juvenil A 1 - KUMITE",
    "76 - Infanto-Juvenil A 2 - KUMITE", "77 - Infanto-Juvenil B 1 - KUMITE", "78 - Infanto-Juvenil B 2 - KUMITE",
    "79 - Infanto-Juvenil C 1 - KUMITE", "80 - Infanto-Juvenil C 2 - KUMITE", "81 - Juvenil A 1 - KUMITE",
    "82 - Juvenil A 2 - KUMITE", "83 - Juvenil B 1 - KUMITE", "84 - Juvenil B 2 - KUMITE", "85 - Juvenil C 1 - KUMITE",
    "86 - Juvenil C 2 - KUMITE", "87 - Adulto A 1 - KUMITE", "88 - Adulto A 2 - KUMITE", "89 - Adulto B 1 - KUMITE",
    "90 - Adulto B 2 - KUMITE", "91 - Adulto C 1 - KUMITE", "92 - Adulto C 2 - KUMITE", "93 - Adulto C 3 - KUMITE",
    "94 - Master A 1 - KUMITE", "95 - Master A 2 - KUMITE", "96 - Master B 1 - KUMITE", "97 - Master B 2 - KUMITE",
    "98 - Master C 1 - KUMITE", "99 - Master C 2 - KUMITE", "100 - Master C 3 - KUMITE"
  ];

  const pontosPorColocacao = { ouro: 7, prata: 5, bronze: 3 };
  let resultados = [];
  let bonusPenalidades = {};
  function carregarDados() {
    const dados = localStorage.getItem('rankingKarateDados');
    if (dados) resultados = JSON.parse(dados);
    const bonus = localStorage.getItem('rankingKarateBonus');
    if (bonus) bonusPenalidades = JSON.parse(bonus);
    preencherSelects();
    atualizarRankings();
    atualizarProgresso();
  }

  function salvarDados() {
    localStorage.setItem('rankingKarateDados', JSON.stringify(resultados));
    localStorage.setItem('rankingKarateBonus', JSON.stringify(bonusPenalidades));
    alert('Dados salvos!');
  }

  function limparDados() {
    if (confirm('Deseja realmente apagar todos os dados?')) {
      resultados = [];
      bonusPenalidades = {};
      localStorage.clear();
      atualizarRankings();
      atualizarProgresso();
      preencherSelects(); // limpa os selects também
    }
  }

  function adicionarResultado(etapa, categoria, atleta, associacao, colocacao) {
    if (!pontosPorColocacao[colocacao]) return alert("Colocação inválida.");
    const existe = resultados.some(r => r.categoria === categoria && r.colocacao === colocacao);
    if (existe) return alert(`Já existe ${colocacao} para ${categoria}. Use editar.`);

    resultados.push({ etapa, categoria, atleta, associacao, colocacao, pontos: pontosPorColocacao[colocacao] });
    salvarDados();
    atualizarRankings();
    atualizarProgresso();
    preencherSelects(); // ← CORREÇÃO aplicada aqui!
  }

  function atualizarProgresso() {
    const feitas = new Set(resultados.map(r => r.categoria));
    const total = categoriasOficiais.length;
    const percentual = total ? Math.round((feitas.size / total) * 100) : 0;
    document.getElementById('progressBar').style.width = percentual + '%';
    document.getElementById('progressBar').textContent = percentual + '%';
    document.getElementById('progressText').textContent = `${feitas.size} de ${total} categorias concluídas.`;

    const concluida = document.getElementById('listaConcluidas');
    const faltando = document.getElementById('listaFaltando');
    concluida.innerHTML = '';
    faltando.innerHTML = '';
    categoriasOficiais.forEach(cat => {
      const li = `<li>${cat}</li>`;
      (feitas.has(cat) ? concluida : faltando).innerHTML += li;
    });
  }

  function atualizarRankings() {
    const filtro = document.getElementById('filtroCategoria').value;
    const catMap = {};
    resultados.forEach(r => {
      if (!catMap[r.categoria]) catMap[r.categoria] = [];
      catMap[r.categoria].push(r);
    });

    const div = document.getElementById('rankingCategorias');
    div.innerHTML = '';
    Object.keys(catMap).sort().forEach(categoria => {
      if (filtro && filtro !== categoria) return;
      let html = `<h3>${categoria}</h3><table><tr><th>Etapa</th><th>Atleta</th><th>Colocação</th><th>Pontos</th><th>Ações</th></tr>`;
      catMap[categoria].forEach(r => {
        html += `<tr>
          <td>${r.etapa}</td>
          <td>${r.atleta}</td>
          <td>${r.colocacao}</td>
          <td>${r.pontos}</td>
          <td><button onclick="editarResultado('${r.categoria}','${r.colocacao}')">Editar</button></td>
        </tr>`;
      });
      html += '</table>';
      div.innerHTML += html;
    });

    const assoc = {};
    resultados.forEach(({ associacao, pontos, colocacao }) => {
      if (!assoc[associacao]) assoc[associacao] = { pontos: 0, ouro: 0, prata: 0, bronze: 0 };
      assoc[associacao].pontos += pontos;
      assoc[associacao][colocacao]++;
    });

    Object.entries(bonusPenalidades).forEach(([assocName, bonus]) => {
      if (!assoc[assocName]) assoc[assocName] = { pontos: 0, ouro: 0, prata: 0, bronze: 0 };
      assoc[assocName].pontos += bonus;
    });

    const tabela = document.querySelector('#rankingAssociacoes tbody');
    tabela.innerHTML = '';
    Object.entries(assoc).sort((a, b) => b[1].pontos - a[1].pontos).forEach(([nome, d]) => {
      tabela.innerHTML += `<tr><td>${nome}</td><td>${d.pontos}</td><td>${d.ouro}</td><td>${d.prata}</td><td>${d.bronze}</td><td>${bonusPenalidades[nome] || 0}</td></tr>`;
    });
  }

  function preencherSelects() {
    const catSelect = document.getElementById('categoria');
    const filtroSelect = document.getElementById('filtroCategoria');
    catSelect.innerHTML = '<option value="">Selecione a Categoria</option>';
    filtroSelect.innerHTML = '<option value="">Todas as Categorias</option>';
    categoriasOficiais.forEach(cat => {
      catSelect.innerHTML += `<option value="${cat}">${cat}</option>`;
      filtroSelect.innerHTML += `<option value="${cat}">${cat}</option>`;
    });

    const bonusSelect = document.getElementById('bonusAssociacao');
    bonusSelect.innerHTML = '<option value="">Selecione Associação</option>';
    const associacoes = [...new Set(resultados.map(r => r.associacao))];
    associacoes.forEach(a => {
      bonusSelect.innerHTML += `<option value="${a}">${a}</option>`;
    });
  }

  function editarResultado(categoria, colocacao) {
    const resultado = resultados.find(r => r.categoria === categoria && r.colocacao === colocacao);
    if (resultado) {
      const novoEtapa = prompt("Editar Etapa:", resultado.etapa);
      const novoAtleta = prompt("Editar Nome do Atleta:", resultado.atleta);
      const novaAssociacao = prompt("Editar Associação:", resultado.associacao);

      if (novoEtapa && novoAtleta && novaAssociacao) {
        resultado.etapa = novoEtapa.trim();
        resultado.atleta = novoAtleta.trim();
        resultado.associacao = novaAssociacao.trim();
        salvarDados();
        atualizarRankings();
        atualizarProgresso();
        preencherSelects();
      }
    }
  }

  function exportarExcel() {
    const wb = XLSX.utils.book_new();
    const ws = XLSX.utils.json_to_sheet(resultados);
    XLSX.utils.book_append_sheet(wb, ws, "Ranking");
    XLSX.writeFile(wb, "ranking-karate.xlsx");
  }

  document.getElementById('resultForm').addEventListener('submit', e => {
    e.preventDefault();
    const etapa = document.getElementById('etapa').value.trim();
    const categoria = document.getElementById('categoria').value;
    const atleta = document.getElementById('atleta').value.trim();
    const associacao = document.getElementById('associacao').value.trim();
    const colocacao = document.getElementById('colocacao').value;
    adicionarResultado(etapa, categoria, atleta, associacao, colocacao);
    e.target.reset();
  });

  document.getElementById('bonusForm').addEventListener('submit', e => {
    e.preventDefault();
    const assoc = document.getElementById('bonusAssociacao').value;
    const val = parseInt(document.getElementById('bonusValor').value);
    if (assoc && !isNaN(val)) {
      bonusPenalidades[assoc] = (bonusPenalidades[assoc] || 0) + val;
      salvarDados();
      atualizarRankings();
    }
  });

  document.getElementById('filtroCategoria').addEventListener('change', atualizarRankings);
  window.onload = carregarDados;
</script>
</body>
</html>

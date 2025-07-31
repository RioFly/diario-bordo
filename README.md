<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Diário de Bordo - RioFly Aviation</title>
  <link href="https://fonts.googleapis.com/css2?family=Outfit:wght@400;600&display=swap" rel="stylesheet" />
  <style>
    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
      font-family: 'Outfit', sans-serif;
    }
    body {
      background: #eaf0f5;
      color: #333;
    }
    header {
      background-color: #003366;
      color: white;
      padding: 2rem;
      text-align: center;
    }
    main {
      max-width: 800px;
      margin: 2rem auto;
      background: white;
      padding: 2rem;
      border-radius: 16px;
      box-shadow: 0 6px 18px rgba(0,0,0,0.1);
    }
    h2 {
      margin-bottom: 1rem;
      color: #003366;
      text-align: center;
    }
    form {
      display: flex;
      flex-direction: column;
      gap: 1rem;
    }
    label {
      font-weight: 600;
    }
    input, select, textarea, button {
      padding: 0.75rem;
      border: 1px solid #ccc;
      border-radius: 8px;
      font-size: 1rem;
    }
    input::placeholder, textarea::placeholder {
      color: #999;
    }
    button {
      background-color: #0066cc;
      color: white;
      font-weight: 600;
      cursor: pointer;
      transition: background 0.3s;
    }
    button:hover {
      background-color: #004c99;
    }
    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 2rem;
    }
    th, td {
      border: 1px solid #ccc;
      padding: 0.75rem;
      text-align: left;
    }
    th {
      background-color: #f4f4f4;
    }
    .btn-apagar {
      background-color: #cc3300;
      border: none;
      padding: 0.5rem 1rem;
      color: white;
      border-radius: 6px;
      cursor: pointer;
      transition: background 0.3s;
    }
    .btn-apagar:hover {
      background-color: #991f00;
    }
    @media (max-width: 600px) {
      main {
        margin: 1rem;
        padding: 1rem;
      }
      table {
        font-size: 0.85rem;
      }
      .btn-apagar {
        padding: 0.3rem 0.6rem;
        font-size: 0.85rem;
      }
    }
    #statusManutencao {
      margin-top: 1rem;
      font-weight: 600;
      color: #cc3300;
      text-align: center;
    }
  </style>
</head>
<body>
  <header>
    <h1>Diário de Bordo</h1>
    <p>RioFly Aviation</p>
  </header>

  <main>
    <h2>Registrar Voo</h2>
    <form id="diarioBordoForm">
      <label for="comandante">Nome:</label>
      <input type="text" id="comandante" required placeholder="Nome completo" />

      <label for="vid">VID:</label>
      <input type="text" id="vid" required placeholder="Ex: 01234" />

      <label for="data">Data:</label>
      <input type="date" id="data" required />

      <label for="aeronave">Aeronave:</label>
      <select id="aeronave" required>
        <option value="">-- Selecione uma aeronave --</option>
        <option value="R66 PP-JMB">R66 PP-JMB</option>
        <option value="H130 PP-RIO">H130 PP-RIO</option>
        <option value="H130 PP-ADR">H130 PP-ADR</option>
        <option value="Phenom 100 PP-EMB">Phenom 100 PP-EMB</option>
        <option value="Phenom 300 PR-FLA">Phenom 300 PR-FLA</option>
        <option value="F90 King Air PP-JCA">F90 King Air PP-JCA</option>
        <option value="Pilatus PC-12 PS-STQ">Pilatus PC-12 PS-STQ</option>
        <option value="Citation CJ3 PS-SCC">Citation CJ3 PS-SCC</option>
        <option value="B99 Airliner PT-FSC">B99 Airliner PT-FSC</option>
      </select>

      <div id="statusManutencao"></div>

      <label for="rota">Rota (Origem → Destino):</label>
      <input type="text" id="rota" placeholder="Ex: SBGL → SBSP" required />

      <label for="milhas">Distância (Milhas Náuticas):</label>
      <input type="number" id="milhas" required placeholder="Ex: 210" />

      <label for="duracao">Duração (ex: 1h 30min, 45min, 2h):</label>
      <input type="text" id="duracao" required placeholder="Ex: 1h 30min" />

      <label for="custos">Taxas/Custos (R$):</label>
      <input type="number" id="custos" required placeholder="Ex: 3200" />

      <label for="observacoes">Observações:</label>
      <textarea id="observacoes" rows="4" placeholder="Ex: Voo realizado em condições visuais, sem intercorrências."></textarea>

      <button type="submit">Enviar Voo</button>
    </form>

    <h2>Histórico de Voos</h2>
    <table>
      <thead>
        <tr>
          <th>Data</th>
          <th>Nome (VID)</th>
          <th>Aeronave</th>
          <th>Rota</th>
          <th>Milhas (NM)</th>
          <th>Receita (R$)</th>
          <th>Custos (R$)</th>
          <th>Lucro (R$)</th>
          <th>Duração</th>
          <th>Observações</th>
          <th>Ações</th>
        </tr>
      </thead>
      <tbody id="historicoTabela">
        <!-- Voos carregados aqui -->
      </tbody>
    </table>
  </main>

  <script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/9.22.0/firebase-app.js";
    import { getDatabase, ref, push, onValue, remove, runTransaction, get, update } from "https://www.gstatic.com/firebasejs/9.22.0/firebase-database.js";

    const firebaseConfig = {
      apiKey: "AIzaSyD7SE9XR48nqXKS_vvmk6c4cJ9ITJAumko",
      authDomain: "riofly-aviation.firebaseapp.com",
      projectId: "riofly-aviation",
      storageBucket: "riofly-aviation.firebasestorage.app",
      messagingSenderId: "162992793097",
      appId: "1:162992793097:web:d4ee41e33dd948d0a0bad8",
      databaseURL: "https://riofly-aviation-default-rtdb.firebaseio.com"
    };

    const app = initializeApp(firebaseConfig);
    const db = getDatabase(app);
    const diarioRef = ref(db, "diario_bordo");
    const saldoRef = ref(db, "banco/saldo");
    const manutencaoRef = ref(db, "manutencao_status");

    const form = document.getElementById("diarioBordoForm");
    const historicoTabela = document.getElementById("historicoTabela");
    const statusManutencaoDiv = document.getElementById("statusManutencao");

    let voosAtuais = {};
    let manutencaoStatus = {}; // objeto para status manutenção por aeronave

    // Limite para manutenção: 75h em minutos
    const LIMITE_MANUTENCAO_MINUTOS = 75 * 60;
    const VALOR_MANUTENCAO = 20000; // R$ 20.000,00 desconto ao entrar em manutenção

    // Função para formatar minutos em "Xh Ymin"
    function formatarDuracao(minutos) {
      const h = Math.floor(minutos / 60);
      const m = minutos % 60;
      let result = "";
      if (h > 0) result += `${h}h`;
      if (m > 0) result += (h > 0 ? " " : "") + `${m}min`;
      if (result === "") result = "0min";
      return result;
    }

    // Função para converter string "1h 30min" em minutos
    function parseDuracao(texto) {
      texto = texto.toLowerCase().trim();
      let horas = 0;
      let minutos = 0;

      // Regex para encontrar partes de horas e minutos
      const regexHora = /(\d+)\s*h/;
      const regexMinuto = /(\d+)\s*min/;

      const matchHora = texto.match(regexHora);
      const matchMinuto = texto.match(regexMinuto);

      if (matchHora) horas = parseInt(matchHora[1]);
      if (matchMinuto) minutos = parseInt(matchMinuto[1]);

      return horas * 60 + minutos;
    }

    function atualizarStatusManutencaoVisual(aeronave) {
      if (!aeronave) {
        statusManutencaoDiv.textContent = "";
        return;
      }
      const status = manutencaoStatus[aeronave]?.status || "operacional";
      if (status === "manutencao") {
        statusManutencaoDiv.textContent = `⚠️ ${aeronave} está EM MANUTENÇÃO - não é possível registrar voos até liberação.`;
        statusManutencaoDiv.style.color = "#cc3300";
      } else {
        statusManutencaoDiv.textContent = "";
      }
    }

    async function atualizarStatusManutencaoEFinanceiro(voos) {
      const duracaoPorAeronave = {};
      if (!voos) return;

      Object.values(voos).forEach(voo => {
        if (!voo.aeronave) return;
        duracaoPorAeronave[voo.aeronave] = (duracaoPorAeronave[voo.aeronave] || 0) + (voo.duracaoMinutos || 0);
      });

      for (const aeronave in duracaoPorAeronave) {
        const duracaoTotal = duracaoPorAeronave[aeronave];
        const statusAtual = manutencaoStatus[aeronave]?.status || "operacional";

        if (duracaoTotal >= LIMITE_MANUTENCAO_MINUTOS && statusAtual === "operacional") {
          manutencaoStatus[aeronave] = { status: "manutencao" };
          await update(ref(db, `manutencao_status/${aeronave}`), { status: "manutencao" });

          await runTransaction(saldoRef, currentSaldo => {
            if (currentSaldo === null) return 0 - VALOR_MANUTENCAO;
            return currentSaldo - VALOR_MANUTENCAO;
          });

          alert(`Aeronave ${aeronave} entrou em MANUTENÇÃO automática. R$ ${VALOR_MANUTENCAO.toLocaleString('pt-BR')} foram descontados do saldo.`);
        } else if (duracaoTotal < LIMITE_MANUTENCAO_MINUTOS) {
          if (statusAtual === "manutencao") {
            manutencaoStatus[aeronave] = { status: "operacional" };
            await update(ref(db, `manutencao_status/${aeronave}`), { status: "operacional" });
          }
        }
      }
    }

    function renderizarHistorico(voos, filtroAeronave = "") {
      historicoTabela.innerHTML = "";
      if (!voos) {
        historicoTabela.innerHTML = `<tr><td colspan="11" style="text-align:center; padding:1rem;">Nenhum voo registrado.</td></tr>`;
        return;
      }

      let duracaoTotalMinutos = 0;
      let voosExibidos = 0;

      Object.entries(voos).forEach(([id, voo]) => {
        if (filtroAeronave && voo.aeronave !== filtroAeronave) return;

        voosExibidos++;

        duracaoTotalMinutos += voo.duracaoMinutos || 0;

        const tr = document.createElement("tr");
        tr.innerHTML = `
          <td>${voo.data}</td>
          <td>${voo.comandante} (${voo.vid})</td>
          <td>${voo.aeronave}</td>
          <td>${voo.rota}</td>
          <td>${voo.milhas}</td>
          <td>R$ ${voo.receita.toLocaleString("pt-BR")}</td>
          <td>R$ ${voo.custos.toLocaleString("pt-BR")}</td>
          <td>R$ ${voo.lucro.toLocaleString("pt-BR")}</td>
          <td>${voo.duracaoStr || ''}</td>
          <td>${voo.observacoes}</td>
          <td><button class="btn-apagar" data-id="${id}">Apagar</button></td>
        `;
        historicoTabela.appendChild(tr);
      });

      if (voosExibidos === 0) {
        historicoTabela.innerHTML = `<tr><td colspan="11" style="text-align:center; padding:1rem;">Nenhum voo registrado.</td></tr>`;
      } else {
        const trResumo = document.createElement("tr");
        trResumo.style.fontWeight = "600";
        trResumo.innerHTML = `
          <td colspan="8" style="text-align:right;">Duração Total Exibida:</td>
          <td colspan="3">${formatarDuracao(duracaoTotalMinutos)}</td>
        `;
        historicoTabela.appendChild(trResumo);
      }

      document.querySelectorAll(".btn-apagar").forEach(btn => {
        btn.addEventListener("click", (e) => {
          const id = e.target.getAttribute("data-id");
          if (confirm("Tem certeza que deseja apagar este voo?")) {
            remove(ref(db, `diario_bordo/${id}`))
              .then(() => {
                alert("Voo apagado com sucesso!");
                if (voosAtuais && voosAtuais[id]) {
                  const lucroApagado = voosAtuais[id].lucro || 0;
                  runTransaction(saldoRef, (currentSaldo) => {
                    return (currentSaldo || 0) - lucroApagado;
                  });
                }
              })
              .catch(err => alert("Erro ao apagar voo: " + err.message));
          }
        });
      });
    }

    onValue(diarioRef, async (snapshot) => {
      voosAtuais = snapshot.val() || {};

      const aeronaveSelecionada = form.aeronave.value;

      await atualizarStatusManutencaoEFinanceiro(voosAtuais);

      const manutencaoSnap = await get(manutencaoRef);
      manutencaoStatus = manutencaoSnap.val() || {};

      atualizarStatusManutencaoVisual(aeronaveSelecionada);

      renderizarHistorico(voosAtuais, aeronaveSelecionada);
    });

    form.aeronave.addEventListener("change", () => {
      atualizarStatusManutencaoVisual(form.aeronave.value);
      renderizarHistorico(voosAtuais, form.aeronave.value);
    });

    form.addEventListener("submit", async (e) => {
      e.preventDefault();

      const comandante = form.comandante.value.trim();
      const vid = form.vid.value.trim();
      const data = form.data.value;
      const aeronave = form.aeronave.value;
      const rota = form.rota.value.trim();
      const milhas = parseFloat(form.milhas.value);
      const custos = parseFloat(form.custos.value);
      const observacoes = form.observacoes.value.trim();
      const duracaoTexto = form.duracao.value.trim();

      // Converter duração texto para minutos
      const duracaoMinutos = parseDuracao(duracaoTexto);
      if (duracaoMinutos <= 0) {
        alert("Duração inválida. Use formato como '1h 30min', '45min' ou '2h'.");
        return;
      }

      if (!comandante || !vid || !data || !aeronave || !rota || isNaN(milhas) || isNaN(custos)) {
        alert("Preencha todos os campos corretamente.");
        return;
      }

      if (manutencaoStatus[aeronave]?.status === "manutencao") {
        alert(`A aeronave ${aeronave} está em MANUTENÇÃO e não pode registrar novos voos.`);
        return;
      }

      const receita = milhas * 100;
      const lucro = receita - custos;

      const novoVoo = {
        comandante,
        vid,
        data,
        aeronave,
        rota,
        milhas,
        custos,
        receita,
        lucro,
        duracaoMinutos,
        duracaoStr: duracaoTexto,
        observacoes,
        timestamp: Date.now()
      };

      try {
        await push(diarioRef, novoVoo);
        alert("Voo registrado com sucesso!");
        await runTransaction(saldoRef, (currentSaldo) => {
          return (currentSaldo || 0) + lucro;
        });
        form.reset();
        atualizarStatusManutencaoVisual(""); // limpa status
      } catch (err) {
        alert("Erro ao registrar voo: " + err.message);
      }
    });
  </script>
</body>
</html>

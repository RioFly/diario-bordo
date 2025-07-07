<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Diário de Bordo - RioFly Aviation</title>
  <link href="https://fonts.googleapis.com/css2?family=Outfit:wght@400;600&display=swap" rel="stylesheet" />
  <style>
    /* ... seu CSS permanece igual ... */
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
    /* Estilo para tabela de histórico */
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
    @media (max-width: 600px) {
      main {
        margin: 1rem;
        padding: 1rem;
      }
      table {
        font-size: 0.85rem;
      }
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
        <option value="Phenom 300">Phenom 300</option>
        <option value="Phenom 100">Phenom 100</option>
        <option value="Citation CJ3">Citation CJ3</option>
        <option value="King Air B200">King Air B200</option>
        <option value="Bell 407">Bell 407</option>
        <option value="Agusta AW109SP">Agusta AW109SP</option>
        <option value="Robinson R66">Robinson R66</option>
        <option value="H125">H125</option>
      </select>

      <label for="rota">Rota (Origem → Destino):</label>
      <input type="text" id="rota" placeholder="Ex: SBGL → SBSP" required />

      <label for="milhas">Distância (Milhas Náuticas):</label>
      <input type="number" id="milhas" required placeholder="Ex: 210" />

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
          <th>Observações</th>
        </tr>
      </thead>
      <tbody id="historicoTabela">
        <!-- Voos carregados aqui -->
      </tbody>
    </table>
  </main>

  <script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/9.22.0/firebase-app.js";
    import { getDatabase, ref, push, onValue } from "https://www.gstatic.com/firebasejs/9.22.0/firebase-database.js";

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

    const form = document.getElementById("diarioBordoForm");
    const historicoTabela = document.getElementById("historicoTabela");

    let voosAtuais = {};

    function renderizarHistorico(voos, filtroAeronave = "") {
      historicoTabela.innerHTML = "";
      if (!voos) return;

      // Só mostra se filtro preenchido (aeronave selecionada)
      if (!filtroAeronave) return;

      Object.entries(voos).forEach(([id, voo]) => {
        if (voo.aeronave !== filtroAeronave) return;

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
          <td>${voo.observacoes}</td>
        `;
        historicoTabela.appendChild(tr);
      });
    }

    onValue(diarioRef, (snapshot) => {
      voosAtuais = snapshot.val();
      renderizarHistorico(voosAtuais, form.aeronave.value);
    });

    form.aeronave.addEventListener("change", () => {
      renderizarHistorico(voosAtuais, form.aeronave.value);
    });

    form.addEventListener("submit", (e) => {
      e.preventDefault();

      const comandante = form.comandante.value.trim();
      const vid = form.vid.value.trim();
      const data = form.data.value;
      const aeronave = form.aeronave.value;
      const rota = form.rota.value.trim();
      const milhas = parseFloat(form.milhas.value);
      const custos = parseFloat(form.custos.value);
      const observacoes = form.observacoes.value.trim();

      if (!comandante || !vid || !data || !aeronave || !rota || isNaN(milhas) || isNaN(custos)) {
        alert("Preencha todos os campos corretamente.");
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
        observacoes,
        timestamp: Date.now()
      };

      push(diarioRef, novoVoo)
        .then(() => {
          alert("Voo registrado com sucesso!");
          form.reset();
        })
        .catch(err => alert("Erro ao registrar voo: " + err.message));
    });
  </script>
</body>
</html>

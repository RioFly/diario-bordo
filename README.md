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
    @media (max-width: 600px) {
      main {
        margin: 1rem;
        padding: 1rem;
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
      <label for="comandante">Comandante:</label>
      <input type="text" id="comandante" required placeholder="Nome completo" />

      <label for="vid">VID:</label>
      <input type="text" id="vid" required placeholder="Ex: 01234" />

      <label for="data">Data:</label>
      <input type="date" id="data" required />

      <label for="aeronave">Aeronave:</label>
      <select id="aeronave" required>
        <option value="">Selecione...</option>
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
  </main>

  <!-- Firebase SDKs -->
  <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-app.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-database.js"></script>

  <script>
    // Configuração do Firebase - substitua pelos seus dados
    const firebaseConfig = {
      apiKey: "SUA_API_KEY",
      authDomain: "SEU_PROJETO.firebaseapp.com",
      databaseURL: "https://SEU_PROJETO.firebaseio.com",
      projectId: "SEU_PROJETO",
      storageBucket: "SEU_PROJETO.appspot.com",
      messagingSenderId: "SENDER_ID",
      appId: "APP_ID"
    };

    // Inicializa o Firebase
    const app = firebase.initializeApp(firebaseConfig);
    const database = firebase.database();

    document.getElementById('diarioBordoForm').addEventListener('submit', function(e) {
      e.preventDefault();

      const comandante = document.getElementById('comandante').value.trim();
      const vid = document.getElementById('vid').value.trim();
      const data = document.getElementById('data').value;
      const aeronave = document.getElementById('aeronave').value;
      const rota = document.getElementById('rota').value.trim();
      const milhas = parseFloat(document.getElementById('milhas').value);
      const custos = parseFloat(document.getElementById('custos').value);
      const observacoes = document.getElementById('observacoes').value.trim();

      const receita = milhas * 100;
      const lucro = receita - custos;

      // Dados do voo para salvar
      const voo = {
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

      // Salvar no diário de bordo
      database.ref('diario_bordo').push(voo)
        .then(() => {
          // Atualizar saldo no banco
          return database.ref('banco/saldo').once('value');
        })
        .then(snapshot => {
          const saldoAtual = snapshot.val() || 5000000; // saldo inicial de 5 milhões se não tiver valor ainda
          const novoSaldo = saldoAtual + lucro;
          return database.ref('banco').update({ saldo: novoSaldo });
        })
        .then(() => {
          alert(
            `Voo registrado com sucesso!\n\n` +
            `Comandante: ${comandante} (VID ${vid})\n` +
            `Data: ${data}\n` +
            `Aeronave: ${aeronave}\n` +
            `Rota: ${rota}\n` +
            `Milhas: ${milhas} NM\n` +
            `Receita: R$ ${receita.toLocaleString('pt-BR')}\n` +
            `Custos: R$ ${custos.toLocaleString('pt-BR')}\n` +
            `Lucro: R$ ${lucro.toLocaleString('pt-BR')}\n\n` +
            `Saldo bancário atualizado!`
          );
          // Limpa formulário
          document.getElementById('diarioBordoForm').reset();
        })
        .catch(error => {
          alert('Erro ao salvar os dados: ' + error.message);
        });
    });
  </script>
</body>
</html>

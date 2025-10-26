<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Ponto Eletrônico</title>
  <script type="module">
    // ======= CONFIG FIREBASE =======
    import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.0/firebase-app.js";
    import {
      getFirestore, collection, getDocs, addDoc, updateDoc,
      deleteDoc, doc, query, orderBy
    } from "https://www.gstatic.com/firebasejs/10.12.0/firebase-firestore.js";
    const firebaseConfig = {
      apiKey: "AIzaSyCpBiFzqOod4K32cWMr5hfx13fw6LGcPVY",
      authDomain: "ponto-eletronico-f35f9.firebaseapp.com",
      projectId: "ponto-eletronico-f35f9",
      storageBucket: "ponto-eletronico-f35f9.firebasestorage.app",
      messagingSenderId: "208638350255",
      appId: "1:208638350255:web:63d016867a67575b5e155a"
    };
    const app = initializeApp(firebaseConfig);
    const db = getFirestore(app);
    // ======= LOGIN POR SENHA =======
    const senha = prompt("Digite a senha de acesso:");
    if (senha !== "02072007") {
      alert("Senha incorreta!");
      document.body.innerHTML = "<h2>Acesso negado!</h2>";
      throw new Error("Acesso negado");
    }
    // ======= ELEMENTOS =======
    const tabela = document.getElementById("tabela-colaboradores");
    const entradaTabela = document.getElementById("tabela-entradas");
    const saidaTabela = document.getElementById("tabela-saidas");
    const colaboradoresRef = collection(db, "colaboradores");
    const registrosRef = collection(db, "registros");
    // ======= FUNÇÕES =======
    async function carregarColaboradores() {
      const q = query(colaboradoresRef, orderBy("nome"));
      const snapshot = await getDocs(q);
      const lista = [];
      snapshot.forEach(docSnap => {
        lista.push({ id: docSnap.id, ...docSnap.data() });
      });
      // Reordena IDs numéricos
      lista.sort((a, b) => a.nome.localeCompare(b.nome));
      for (let i = 0; i < lista.length; i++) {
        lista[i].id_num = i + 1;
        await updateDoc(doc(db, "colaboradores", lista[i].id), { id_num: i + 1 });
      }
      tabela.innerHTML = "";
      lista.forEach(colab => {
        const tr = document.createElement("tr");
        tr.innerHTML = `
          <td>${colab.id_num}</td>
          <td>${colab.nome}</td>
          <td>${colab.email || ""}</td>
          <td>
            <button onclick="registrarEntrada('${colab.id}', '${colab.nome}', '${colab.email || ""}')">Entrada</button>
            <button onclick="registrarSaida('${colab.id}', '${colab.nome}', '${colab.email || ""}')">Saída</button>
            <button onclick="editarColaborador('${colab.id}', '${colab.nome}', '${colab.email || ""}')">Editar</button>
            <button onclick="excluirColaborador('${colab.id}')">Excluir</button>
          </td>
        `;
        tabela.appendChild(tr);
      });
    }
    // Registrar entrada
    window.registrarEntrada = async (id, nome, email) => {
      await addDoc(registrosRef, {
        colaboradorId: id,
        nome,
        email,
        tipo: "Entrada",
        data: new Date().toLocaleString()
      });
      carregarRegistros();
    };
    // Registrar saída
    window.registrarSaida = async (id, nome, email) => {
      await addDoc(registrosRef, {
        colaboradorId: id,
        nome,
        email,
        tipo: "Saída",
        data: new Date().toLocaleString()
      });
      carregarRegistros();
    };
    // Editar colaborador
    window.editarColaborador = async (id, nome, email) => {
      const novoNome = prompt("Novo nome:", nome);
      const novoEmail = prompt("Novo e-mail:", email);
      if (novoNome && novoEmail) {
        await updateDoc(doc(db, "colaboradores", id), {
          nome: novoNome,
          email: novoEmail
        });
        carregarColaboradores();
      }
    };
    // Excluir colaborador
    window.excluirColaborador = async (id) => {
      if (confirm("Deseja excluir este colaborador?")) {
        await deleteDoc(doc(db, "colaboradores", id));
        carregarColaboradores();
      }
    };
    // Carregar registros
    async function carregarRegistros() {
      const snapshot = await getDocs(registrosRef);
      const entradas = [];
      const saidas = [];
      snapshot.forEach(d => {
        const data = d.data();
        if (data.tipo === "Entrada") entradas.push(data);
        else if (data.tipo === "Saída") saidas.push(data);
      });
      entradaTabela.innerHTML = "";
      saidaTabela.innerHTML = "";
      entradas.sort((a, b) => a.nome.localeCompare(b.nome));
      saidas.sort((a, b) => a.nome.localeCompare(b.nome));
      entradas.forEach(r => {
        const tr = document.createElement("tr");
        tr.innerHTML = `<td>${r.nome}</td><td>${r.email}</td><td>${r.data}</td>`;
        entradaTabela.appendChild(tr);
      });
      saidas.forEach(r => {
        const tr = document.createElement("tr");
        tr.innerHTML = `<td>${r.nome}</td><td>${r.email}</td><td>${r.data}</td>`;
        saidaTabela.appendChild(tr);
      });
    }
    // Exportar Excel
    window.exportarExcel = async () => {
      const XLSX = await import("https://cdn.sheetjs.com/xlsx-0.20.0/package/xlsx.mjs");
      const wb = XLSX.utils.book_new();
      // Entradas
      const entradasData = [["Nome", "E-mail", "Data/Hora"]];
      entradaTabela.querySelectorAll("tr").forEach(tr => {
        const tds = tr.querySelectorAll("td");
        if (tds.length) entradasData.push([tds[0].innerText, tds[1].innerText, tds[2].innerText]);
      });
      const ws1 = XLSX.utils.aoa_to_sheet(entradasData);
      XLSX.utils.book_append_sheet(wb, ws1, "Entradas");
      // Saídas
      const saidasData = [["Nome", "E-mail", "Data/Hora"]];
      saidaTabela.querySelectorAll("tr").forEach(tr => {
        const tds = tr.querySelectorAll("td");
        if (tds.length) saidasData.push([tds[0].innerText, tds[1].innerText, tds[2].innerText]);
      });
      const ws2 = XLSX.utils.aoa_to_sheet(saidasData);
      XLSX.utils.book_append_sheet(wb, ws2, "Saídas");
      XLSX.writeFile(wb, "Ponto_Eletronico.xlsx");
    };
    // Inicialização
    carregarColaboradores();
    carregarRegistros();
  </script>
  <style>
    body { font-family: Arial; margin: 30px; }
    table { width: 100%; border-collapse: collapse; margin-top: 15px; }
    th, td { border: 1px solid #ccc; padding: 6px; text-align: center; }
    th { background: #4CAF50; color: white; }
    button { margin: 3px; padding: 5px 8px; cursor: pointer; border-radius: 5px; }
    h1 { color: #333; }
    .botoes { margin-bottom: 10px; }
  </style>
</head>

<body>
  <h1>Ponto Eletrônico</h1>
  <div class="botoes">
    <button onclick="exportarExcel()">Exportar Excel</button>
  </div>

  <h2>Colaboradores</h2>
  <table>
    <thead>
      <tr><th>ID</th><th>Nome</th><th>E-mail</th><th>Ações</th></tr>
    </thead>
    <tbody id="tabela-colaboradores"></tbody>
  </table>

  <h2>Entradas</h2>
  <table>
    <thead><tr><th>Nome</th><th>E-mail</th><th>Data/Hora</th></tr></thead>
    <tbody id="tabela-entradas"></tbody>
  </table>

  <h2>Saídas</h2>
  <table>
    <thead><tr><th>Nome</th><th>E-mail</th><th>Data/Hora</th></tr></thead>
    <tbody id="tabela-saidas"></tbody>
  </table>
</body>
</html>

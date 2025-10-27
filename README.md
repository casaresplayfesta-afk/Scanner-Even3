<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Ponto Eletrônico CLX</title>
  <script type="module">
    // ---------------- IMPORTS FIREBASE ----------------
    import { initializeApp } from "https://www.gstatic.com/firebasejs/10.14.0/firebase-app.js";
    import { 
      getFirestore, collection, addDoc, getDocs, deleteDoc, doc, query, where 
    } from "https://www.gstatic.com/firebasejs/10.14.0/firebase-firestore.js";
    // ---------------- CONFIG FIREBASE ----------------
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
    // ---------------- LOGIN ----------------
    const loginDiv = document.createElement('div');
    loginDiv.innerHTML = `
      <div style="display:flex;justify-content:center;align-items:center;height:100vh;background:#f3f4f6;">
        <div style="background:white;padding:30px;border-radius:10px;box-shadow:0 2px 10px rgba(0,0,0,0.1);width:300px;text-align:center">
          <h2>Login</h2>
          <input id="user" placeholder="Usuário" style="width:100%;margin:5px 0;padding:8px">
          <input id="pass" type="password" placeholder="Senha" style="width:100%;margin:5px 0;padding:8px">
          <button id="entrarBtn" style="width:100%;margin-top:10px;padding:8px;background:#2563eb;color:white;border:none;border-radius:5px;cursor:pointer">Entrar</button>
          <p style="font-size:12px;color:#666;margin-top:6px">
            Usuário: <b>CLX</b> / Senha: <b>02072007</b>
          </p>
        </div>
      </div>`;
    document.body.appendChild(loginDiv);
    document.getElementById('entrarBtn').addEventListener('click', ()=>{
      const u = document.getElementById('user').value.trim();
      const p = document.getElementById('pass').value.trim();
      if(u==="CLX" && p==="02072007"){
        loginDiv.remove();
        iniciarSistema();
      } else alert("Usuário ou senha incorretos!");
    });
    // ---------------- SISTEMA PRINCIPAL ----------------
    async function iniciarSistema(){
      document.body.innerHTML = `
      <div style="padding:20px;">
        <h2>Ponto Eletrônico</h2>
        <button id="addColab">Adicionar Colaborador</button>
        <button id="baixarExcel">Baixar Planilha</button>
        <button id="limparTodos">Limpar Todos os Pontos</button>
        <h3 style="margin-top:20px;">Colaboradores</h3>
        <table border="1" width="100%" style="border-collapse:collapse">
          <thead>
            <tr>
              <th>Nome</th><th>Matrícula</th><th>Email</th><th>Ações</th>
            </tr>
          </thead>
          <tbody id="colabTable"></tbody>
        </table>
        <h3 style="margin-top:20px;">Registros de Ponto</h3>
        <table border="1" width="100%" style="border-collapse:collapse">
          <thead>
            <tr><th>Nome</th><th>Tipo</th><th>Data</th><th>Hora</th></tr>
          </thead>
          <tbody id="pontoTable"></tbody>
        </table>
      </div>`;
      const colabTable = document.getElementById('colabTable');
      const pontoTable = document.getElementById('pontoTable');
      const addBtn = document.getElementById('addColab');
      const limparTodosBtn = document.getElementById('limparTodos');
      let colaboradores = [];
      let pontos = [];
      // ----- CARREGAR DADOS DO FIREBASE -----
      const colabSnap = await getDocs(collection(db, "colaboradores"));
      colabSnap.forEach(d => colaboradores.push({id: d.id, ...d.data()}));
      const pontoSnap = await getDocs(collection(db, "pontos"));
      pontoSnap.forEach(d => pontos.push({id: d.id, ...d.data()}));
      renderAll();
      // ----- ADICIONAR COLAB -----
      addBtn.addEventListener('click', async ()=>{
        const nome = prompt("Nome do colaborador:");
        const matricula = prompt("Matrícula:");
        const email = prompt("Email:");
        if(nome && matricula && email){
          const docRef = await addDoc(collection(db, "colaboradores"), {nome, matricula, email});
          colaboradores.push({id: docRef.id, nome, matricula, email});
          renderColab();
        }
      });
      // ----- RENDERIZAÇÕES -----
      function renderAll(){ renderColab(); renderPontos(); }
      function renderColab(){
        colabTable.innerHTML = colaboradores.map(c=>`
          <tr>
            <td>${c.nome}</td>
            <td>${c.matricula}</td>
            <td>${c.email}</td>
            <td>
              <button onclick="registrarPonto('${c.id}','Entrada')">Entrada</button>
              <button onclick="registrarPonto('${c.id}','Saída')">Saída</button>
              <button onclick="removerColab('${c.id}')">Excluir</button>
            </td>
          </tr>
        `).join('');
      }
      function renderPontos(){
        pontoTable.innerHTML = pontos.map(p=>`
          <tr>
            <td>${p.nome}</td>
            <td>${p.tipo}</td>
            <td>${p.data}</td>
            <td>${p.hora}</td>
          </tr>
        `).join('');
      }
      // ----- REGISTRAR PONTO -----
      window.registrarPonto = async (idColab, tipo)=>{
        const c = colaboradores.find(x=>x.id===idColab);
        if(!c) return;
        const now = new Date();
        const registro = {
          nome:c.nome, tipo,
          data: now.toLocaleDateString('pt-BR'),
          hora: now.toLocaleTimeString('pt-BR',{hour12:false}),
          idColab: idColab
        };
        const docRef = await addDoc(collection(db, "pontos"), registro);
        pontos.push({id: docRef.id, ...registro});
        renderPontos();
      };
      // ----- EXCLUIR COLAB -----
      window.removerColab = async (id)=>{
        if(confirm('Deseja realmente excluir este colaborador e seus pontos?')){
          await deleteDoc(doc(db, "colaboradores", id));
          const q = query(collection(db, "pontos"), where("idColab", "==", id));
          const snap = await getDocs(q);
          snap.forEach(async (docSnap)=> await deleteDoc(doc(db, "pontos", docSnap.id)));
          colaboradores = colaboradores.filter(x=>x.id!==id);
          pontos = pontos.filter(p=>p.idColab!==id);
          renderAll();
          alert('Excluído com sucesso!');
        }
      };
      // ----- LIMPAR TODOS -----
      limparTodosBtn.addEventListener('click', async ()=>{
        if(confirm('Deseja realmente apagar todos os pontos?')){
          const snap = await getDocs(collection(db, "pontos"));
          snap.forEach(async (d)=> await deleteDoc(doc(db,"pontos",d.id)));
          pontos = [];
          renderPontos();
          alert('Todos os pontos foram apagados!');
        }
      });
    }
  </script>
</head>
<body>
</body>
</html>

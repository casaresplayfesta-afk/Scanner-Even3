<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
    <title>Sistema de Ponto Eletrônico</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 20px;
            background-color: #f5f5f5;
        }
        .container {
            max-width: 1200px;
            margin: 0 auto;
            background-color: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 0 10px rgba(0,0,0,0.1);
        }
        h1 {
            color: #333;
            text-align: center;
            margin-bottom: 30px;
        }
        .button-group {
            display: flex;
            flex-wrap: wrap;
            gap: 10px;
            margin-bottom: 20px;
        }
        button {
            padding: 10px 15px;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 16px;
            transition: background-color 0.3s;
        }
        button:hover { background-color: #45a049; }
        .tech-button { background-color: #2196F3; }
        .tech-button:hover { background-color: #0b7dda; }
        .facial-button { background-color: #ff9800; }
        .facial-button:hover { background-color: #e68a00; }

        .modal {
            display: none;
            position: fixed;
            z-index: 1;
            left: 0; top: 0;
            width: 100%; height: 100%;
            overflow: auto;
            background-color: rgba(0,0,0,0.4);
        }
        .modal-content {
            background-color: #fefefe;
            margin: 8% auto;
            padding: 20px;
            border: 1px solid #888;
            width: 90%;
            max-width: 700px;
            border-radius: 8px;
        }
        .close {
            color: #aaa;
            float: right;
            font-size: 28px;
            font-weight: bold;
            cursor: pointer;
        }
        .close:hover { color: black; }
        form { display: flex; flex-direction: column; gap: 12px; }
        input, select { padding: 10px; border: 1px solid #ddd; border-radius: 4px; font-size: 16px; }
        table { width: 100%; border-collapse: collapse; margin-top: 20px; }
        th, td { border: 1px solid #ddd; padding: 12px; text-align: left; }
        th { background-color: #f2f2f2; }
        tr:nth-child(even) { background-color: #f9f9f9; }

        .camera-container { display:flex; flex-direction:column; align-items:center; gap:12px; }
        #video { background-color: #ddd; width:100%; max-width:500px; }
        #canvas { display:none; }

        .footer {
            text-align:center;
            margin-top:30px;
            padding-top:20px;
            border-top:1px solid #ddd;
            color:#666;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Sistema de Ponto Eletrônico</h1>

        <div class="button-group">
            <button id="addColabBtn">Adicionar Colaborador</button>
            <button id="editColabBtn">Editar Colaborador</button>
            <button id="deleteColabBtn">Excluir Colaborador</button>
            <button id="trocaTurnoBtn">Troca de Turnos</button>
            <button id="registrarPontoBtn">Registrar Ponto</button>
            <button id="reconhecimentoBtn" class="facial-button">Reconhecimento Facial</button>
            <button id="techBtn" class="tech-button">Profissional T.I</button>
        </div>

        <div id="pontosTable">
            <h2>Registros de Ponto</h2>
            <table id="registrosTable">
                <thead>
                    <tr>
                        <th>ID</th>
                        <th>Nome</th>
                        <th>Data/Hora</th>
                        <th>Tipo</th>
                        <th>Método</th>
                    </tr>
                </thead>
                <tbody id="registrosBody"></tbody>
            </table>
        </div>

        <div id="colaboradoresTable">
            <h2>Colaboradores</h2>
            <table id="colabTable">
                <thead>
                    <tr>
                        <th>ID</th>
                        <th>Nome</th>
                        <th>Matrícula</th>
                        <th>Cargo</th>
                        <th>Turno</th>
                    </tr>
                </thead>
                <tbody id="colabBody"></tbody>
            </table>
        </div>

        <div class="footer">
            <p>Sistema de Ponto Eletrônico v1.0</p>
        </div>
    </div>

    <!-- Modal Adicionar Colaborador -->
    <div id="addColabModal" class="modal">
        <div class="modal-content">
            <span class="close" data-target="addColabModal">&times;</span>
            <h2>Adicionar Colaborador</h2>
            <form id="addColabForm">
                <input type="text" id="colabNome" placeholder="Nome Completo" required>
                <input type="text" id="colabMatricula" placeholder="Matrícula (somente números)" required pattern="\d{4,11}">
                <input type="text" id="colabCargo" placeholder="Cargo" required>
                <select id="colabTurno" required>
                    <option value="">Selecione o Turno</option>
                    <option value="Manhã">Manhã (06:00 - 12:00)</option>
                    <option value="Tarde">Tarde (12:00 - 18:00)</option>
                    <option value="Noite">Noite (18:00 - 00:00)</option>
                    <option value="Madrugada">Madrugada (00:00 - 06:00)</option>
                </select>
                <button type="submit">Salvar</button>
            </form>
        </div>
    </div>

    <!-- Modal Editar Colaborador -->
    <div id="editColabModal" class="modal">
        <div class="modal-content">
            <span class="close" data-target="editColabModal">&times;</span>
            <h2>Editar Colaborador</h2>
            <form id="editColabForm">
                <select id="editColabId" required>
                    <option value="">Selecione o Colaborador</option>
                </select>
                <input type="text" id="editColabNome" placeholder="Nome Completo" required>
                <input type="text" id="editColabMatricula" placeholder="Matrícula (somente números)" required pattern="\d{4,11}">
                <input type="text" id="editColabCargo" placeholder="Cargo" required>
                <select id="editColabTurno" required>
                    <option value="">Selecione o Turno</option>
                    <option value="Manhã">Manhã (06:00 - 12:00)</option>
                    <option value="Tarde">Tarde (12:00 - 18:00)</option>
                    <option value="Noite">Noite (18:00 - 00:00)</option>
                    <option value="Madrugada">Madrugada (00:00 - 06:00)</option>
                </select>
                <button type="submit">Atualizar</button>
            </form>
        </div>
    </div>

    <!-- Modal Excluir Colaborador -->
    <div id="deleteColabModal" class="modal">
        <div class="modal-content">
            <span class="close" data-target="deleteColabModal">&times;</span>
            <h2>Excluir Colaborador</h2>
            <form id="deleteColabForm">
                <select id="deleteColabId" required>
                    <option value="">Selecione o Colaborador</option>
                </select>
                <p>Tem certeza que deseja excluir este colaborador?</p>
                <button type="submit">Confirmar Exclusão</button>
            </form>
        </div>
    </div>

    <!-- Modal Troca de Turnos -->
    <div id="trocaTurnoModal" class="modal">
        <div class="modal-content">
            <span class="close" data-target="trocaTurnoModal">&times;</span>
            <h2>Troca de Turnos</h2>
            <form id="trocaTurnoForm">
                <select id="substitutoId" required>
                    <option value="">Selecione o Colaborador Substituto</option>
                </select>
                <select id="substituidoId" required>
                    <option value="">Selecione o Colaborador Substituído</option>
                </select>
                <input type="datetime-local" id="dataTroca" required>
                <button type="submit">Registrar Troca</button>
            </form>
        </div>
    </div>

    <!-- Modal Registrar Ponto -->
    <div id="registrarPontoModal" class="modal">
        <div class="modal-content">
            <span class="close" data-target="registrarPontoModal">&times;</span>
            <h2>Registrar Ponto</h2>
            <form id="registrarPontoForm">
                <select id="pontoColabId" required>
                    <option value="">Selecione o Colaborador</option>
                </select>
                <select id="pontoTipo" required>
                    <option value="">Selecione o Tipo</option>
                    <option value="Entrada">Entrada</option>
                    <option value="Saída">Saída</option>
                    <option value="Intervalo">Intervalo</option>
                    <option value="Retorno">Retorno</option>
                </select>
                <button type="submit">Registrar</button>
            </form>
        </div>
    </div>

    <!-- Modal Profissional T.I -->
    <div id="techModal" class="modal">
        <div class="modal-content">
            <span class="close" data-target="techModal">&times;</span>
            <h2>Opções Técnicas</h2>
            <div class="button-group" style="flex-direction: column;">
                <button id="backupBtn">Backup</button>
                <button id="salvarBtn">Salvar</button>
                <button id="exportPdfBtn">Exportar PDF</button>
                <button id="imprimirBtn">Imprimir</button>
                <button id="regFacialBtn">Registro de Reconhecimento Facial</button>
            </div>
        </div>
    </div>

    <!-- Modal Reconhecimento Facial -->
    <div id="facialModal" class="modal">
        <div class="modal-content">
            <span class="close" data-target="facialModal">&times;</span>
            <h2>Reconhecimento Facial</h2>
            <div class="camera-container">
                <video id="video" width="640" height="480" autoplay></video>
                <canvas id="canvas" width="640" height="480"></canvas>
                <button id="captureBtn">Capturar e Registrar Ponto</button>
                <div id="facialResult"></div>
            </div>
        </div>
    </div>

    <!-- Modal Backup -->
    <div id="backupModal" class="modal">
        <div class="modal-content">
            <span class="close" data-target="backupModal">&times;</span>
            <h2>Backup</h2>
            <p>Backup realizado com sucesso em: <span id="backupPath"></span></p>
        </div>
    </div>

    <script>
        // --- Dados iniciais (localStorage) ---
        let colaboradores = JSON.parse(localStorage.getItem('colaboradores')) || [];
        let registrosPonto = JSON.parse(localStorage.getItem('registrosPonto')) || [];
        let ultimoIdColab = colaboradores.length > 0 ? Math.max(...colaboradores.map(c => c.id)) : 0;
        let ultimoIdRegistro = registrosPonto.length > 0 ? Math.max(...registrosPonto.map(r => r.id)) : 0;

        // --- Elementos DOM ---
        const colabBody = document.getElementById('colabBody');
        const registrosBody = document.getElementById('registrosBody');

        // Modais
        const addColabModal = document.getElementById('addColabModal');
        const editColabModal = document.getElementById('editColabModal');
        const deleteColabModal = document.getElementById('deleteColabModal');
        const trocaTurnoModal = document.getElementById('trocaTurnoModal');
        const registrarPontoModal = document.getElementById('registrarPontoModal');
        const techModal = document.getElementById('techModal');
        const facialModal = document.getElementById('facialModal');
        const backupModal = document.getElementById('backupModal');

        // --- Ações dos botões (abrir modais) ---
        document.getElementById('addColabBtn').addEventListener('click', () => addColabModal.style.display = 'block');
        document.getElementById('editColabBtn').addEventListener('click', () => {
            carregarSelectColaboradores('editColabId');
            editColabModal.style.display = 'block';
        });
        document.getElementById('deleteColabBtn').addEventListener('click', () => {
            carregarSelectColaboradores('deleteColabId');
            deleteColabModal.style.display = 'block';
        });
        document.getElementById('trocaTurnoBtn').addEventListener('click', () => {
            carregarSelectColaboradores('substitutoId');
            carregarSelectColaboradores('substituidoId');
            trocaTurnoModal.style.display = 'block';
        });
        document.getElementById('registrarPontoBtn').addEventListener('click', () => {
            carregarSelectColaboradoresAtivos('pontoColabId');
            registrarPontoModal.style.display = 'block';
        });
        document.getElementById('techBtn').addEventListener('click', () => techModal.style.display = 'block');
        document.getElementById('reconhecimentoBtn').addEventListener('click', () => {
            iniciarCamera();
            facialModal.style.display = 'block';
        });

        // Botões técnicos
        document.getElementById('backupBtn').addEventListener('click', fazerBackup);
        document.getElementById('salvarBtn').addEventListener('click', salvarDados);
        document.getElementById('exportPdfBtn').addEventListener('click', exportarPDF);
        document.getElementById('imprimirBtn').addEventListener('click', imprimirRegistros);
        document.getElementById('regFacialBtn').addEventListener('click', () => {
            techModal.style.display = 'none';
            iniciarCamera();
            facialModal.style.display = 'block';
        });

        // Fechar modais ao clicar no X (cada close tem data-target)
        const closeButtons = document.querySelectorAll('.close');
        closeButtons.forEach(btn => {
            btn.addEventListener('click', function() {
                const target = this.getAttribute('data-target');
                if (target) {
                    document.getElementById(target).style.display = 'none';
                } else {
                    // fallback: fechar o modal pai mais próximo
                    this.closest('.modal').style.display = 'none';
                }
                pararCamera();
            });
        });

        // Fechar modais ao clicar fora
        window.addEventListener('click', function(event) {
            if (event.target.classList.contains('modal')) {
                event.target.style.display = 'none';
                pararCamera();
            }
        });

        // Formulários
        document.getElementById('addColabForm').addEventListener('submit', adicionarColaborador);
        document.getElementById('editColabForm').addEventListener('submit', editarColaborador);
        document.getElementById('deleteColabForm').addEventListener('submit', excluirColaborador);
        document.getElementById('trocaTurnoForm').addEventListener('submit', registrarTrocaTurno);
        document.getElementById('registrarPontoForm').addEventListener('submit', registrarPonto);
        document.getElementById('captureBtn').addEventListener('click', capturarFoto);

        // Carregar dados iniciais na interface
        carregarColaboradores();
        carregarRegistrosPonto();

        // ---------------- Funções de renderização ----------------
        function carregarColaboradores() {
            colabBody.innerHTML = '';
            colaboradores.forEach(colab => {
                const row = document.createElement('tr');
                row.innerHTML = `
                    <td>${colab.id}</td>
                    <td>${colab.nome}</td>
                    <td>${formatarMatricula(colab.matricula)}</td>
                    <td>${colab.cargo}</td>
                    <td>${colab.turno}</td>
                `;
                colabBody.appendChild(row);
            });
        }

        function carregarRegistrosPonto() {
            registrosBody.innerHTML = '';
            registrosPonto.forEach(registro => {
                const colab = colaboradores.find(c => c.id === registro.colabId);
                const row = document.createElement('tr');
                row.innerHTML = `
                    <td>${registro.id}</td>
                    <td>${colab ? colab.nome : 'Colaborador não encontrado'}</td>
                    <td>${formatarDataHora(registro.dataHora)}</td>
                    <td>${registro.tipo}</td>
                    <td>${registro.metodo}</td>
                `;
                registrosBody.appendChild(row);
            });
        }

        function carregarSelectColaboradores(selectId, incluirInativos = false) {
            const select = document.getElementById(selectId);
            if (!select) return;
            select.innerHTML = '<option value="">Selecione o Colaborador</option>';
            colaboradores.forEach(colab => {
                if (incluirInativos || !colab.inativo) {
                    const option = document.createElement('option');
                    option.value = colab.id;
                    option.textContent = `${colab.nome} (${formatarMatricula(colab.matricula)})`;
                    select.appendChild(option);
                }
            });
        }

        function carregarSelectColaboradoresAtivos(selectId) {
            carregarSelectColaboradores(selectId, false);
        }

        // ---------------- Formulários: Adicionar / Editar / Excluir ----------------
        function adicionarColaborador(e) {
            e.preventDefault();
            const nome = document.getElementById('colabNome').value.trim();
            const matricula = document.getElementById('colabMatricula').value.trim();
            const cargo = document.getElementById('colabCargo').value.trim();
            const turno = document.getElementById('colabTurno').value;

            if (!validarMatricula(matricula)) {
                alert('Matrícula inválida! Use somente números (4 a 11 dígitos).');
                return;
            }
            if (colaboradores.some(c => c.matricula === matricula)) {
                alert('Já existe um colaborador com esta matrícula!');
                return;
            }

            ultimoIdColab++;
            const novoColab = {
                id: ultimoIdColab,
                nome,
                matricula,
                cargo,
                turno,
                inativo: false
            };
            colaboradores.push(novoColab);
            salvarDados();
            carregarColaboradores();
            addColabModal.style.display = 'none';
            document.getElementById('addColabForm').reset();
        }

        function editarColaborador(e) {
            e.preventDefault();
            const id = parseInt(document.getElementById('editColabId').value);
            const nome = document.getElementById('editColabNome').value.trim();
            const matricula = document.getElementById('editColabMatricula').value.trim();
            const cargo = document.getElementById('editColabCargo').value.trim();
            const turno = document.getElementById('editColabTurno').value;

            if (!validarMatricula(matricula)) {
                alert('Matrícula inválida! Use somente números (4 a 11 dígitos).');
                return;
            }

            // Verificar duplicidade de matrícula em outro colaborador
            const dup = colaboradores.find(c => c.matricula === matricula && c.id !== id);
            if (dup) {
                alert('Outra pessoa já usa essa matrícula!');
                return;
            }

            const index = colaboradores.findIndex(c => c.id === id);
            if (index !== -1) {
                colaboradores[index] = {
                    ...colaboradores[index],
                    nome,
                    matricula,
                    cargo,
                    turno
                };
                salvarDados();
                carregarColaboradores();
                editColabModal.style.display = 'none';
                document.getElementById('editColabForm').reset();
            }
        }

        function excluirColaborador(e) {
            e.preventDefault();
            const id = parseInt(document.getElementById('deleteColabId').value);
            const index = colaboradores.findIndex(c => c.id === id);
            if (index !== -1) {
                // marca como inativo em vez de excluir
                colaboradores[index].inativo = true;
                salvarDados();
                carregarColaboradores();
                deleteColabModal.style.display = 'none';
                document.getElementById('deleteColabForm').reset();
            }
        }

        function registrarTrocaTurno(e) {
            e.preventDefault();
            const substitutoId = parseInt(document.getElementById('substitutoId').value);
            const substituidoId = parseInt(document.getElementById('substituidoId').value);
            const dataTroca = document.getElementById('dataTroca').value;
            if (substitutoId === substituidoId) {
                alert('O colaborador substituto não pode ser o mesmo que o substituído!');
                return;
            }
            // Aqui poderia gravar uma estrutura de trocas se necessário
            alert(`Troca de turno registrada: Colaborador ${substitutoId} substitui ${substituidoId} em ${dataTroca}`);
            trocaTurnoModal.style.display = 'none';
            document.getElementById('trocaTurnoForm').reset();
        }

        function registrarPonto(e) {
            e.preventDefault();
            const colabId = parseInt(document.getElementById('pontoColabId').value);
            const tipo = document.getElementById('pontoTipo').value;
            const agora = new Date();

            // Evitar duplicados em 5 minutos
            const registroRepetido = registrosPonto.some(r =>
                r.colabId === colabId &&
                r.tipo === tipo &&
                (new Date(agora) - new Date(r.dataHora)) < 300000 // 5 minutos
            );
            if (registroRepetido) {
                alert('Este ponto já foi registrado recentemente!');
                return;
            }

            ultimoIdRegistro++;
            const novoRegistro = {
                id: ultimoIdRegistro,
                colabId,
                dataHora: agora.toISOString(),
                tipo,
                metodo: 'Manual'
            };
            registrosPonto.push(novoRegistro);
            salvarDados();
            carregarRegistrosPonto();
            registrarPontoModal.style.display = 'none';
            document.getElementById('registrarPontoForm').reset();
        }

        // ---------------- Reconhecimento facial (simulado) ----------------
        let stream = null;
        function iniciarCamera() {
            const video = document.getElementById('video');
            if (!navigator.mediaDevices || !navigator.mediaDevices.getUserMedia) {
                document.getElementById('facialResult').textContent = "Navegador não suporta getUserMedia.";
                return;
            }
            navigator.mediaDevices.getUserMedia({ video: true, audio: false })
                .then(function(s) {
                    stream = s;
                    video.srcObject = stream;
                })
                .catch(function(err) {
                    console.error("Erro ao acessar a câmera: ", err);
                    document.getElementById('facialResult').textContent = "Erro ao acessar a câmera: " + err.message;
                });
        }
        function pararCamera() {
            if (stream) {
                stream.getTracks().forEach(track => track.stop());
                stream = null;
            }
        }
        function capturarFoto() {
            const video = document.getElementById('video');
            const canvas = document.getElementById('canvas');
            const context = canvas.getContext('2d');
            context.drawImage(video, 0, 0, canvas.width, canvas.height);

            const agora = new Date();
            // Simulação: escolhe um colaborador ativo aleatório
            const colabsAtivos = colaboradores.filter(c => !c.inativo);
            if (colabsAtivos.length === 0) {
                alert('Nenhum colaborador ativo cadastrado!');
                return;
            }
            const colabAleatorio = colabsAtivos[Math.floor(Math.random() * colabsAtivos.length)];
            const tipo = Math.random() > 0.5 ? 'Entrada' : 'Saída';

            ultimoIdRegistro++;
            const novoRegistro = {
                id: ultimoIdRegistro,
                colabId: colabAleatorio.id,
                dataHora: agora.toISOString(),
                tipo,
                metodo: 'Reconhecimento Facial'
            };
            registrosPonto.push(novoRegistro);
            salvarDados();
            carregarRegistrosPonto();
            document.getElementById('facialResult').textContent = `Ponto registrado para ${colabAleatorio.nome} (${tipo})`;

            setTimeout(() => {
                facialModal.style.display = 'none';
                pararCamera();
                document.getElementById('facialResult').textContent = '';
            }, 2000);
        }

        // ---------------- Funções técnicas ----------------
        function fazerBackup() {
            const agora = new Date();
            const backupPath = `C:/backups/ponto_eletronico_${agora.getFullYear()}${(agora.getMonth()+1).toString().padStart(2,'0')}${agora.getDate().toString().padStart(2,'0')}_${agora.getHours().toString().padStart(2,'0')}${agora.getMinutes().toString().padStart(2,'0')}${agora.getSeconds().toString().padStart(2,'0')}.json`;
            document.getElementById('backupPath').textContent = backupPath;
            techModal.style.display = 'none';
            backupModal.style.display = 'block';
            setTimeout(() => backupModal.style.display = 'none', 3000);
        }

        function salvarDados() {
            localStorage.setItem('colaboradores', JSON.stringify(colaboradores));
            localStorage.setItem('registrosPonto', JSON.stringify(registrosPonto));
        }

        function exportarPDF() {
            alert('PDF exportado com sucesso!');
            techModal.style.display = 'none';
        }

        function imprimirRegistros() {
            window.print();
            techModal.style.display = 'none';
        }

        // ---------------- Utilitários ----------------
        function formatarMatricula(m) {
            if (!m) return '';
            // aqui deixamos a matrícula como está (pode formatar se tiver padrão)
            return m;
        }

        function formatarDataHora(dataHora) {
            const date = new Date(dataHora);
            return date.toLocaleString('pt-BR');
        }

        function validarMatricula(m) {
            // validação simples: somente dígitos, entre 4 e 11 caracteres
            return /^\d{4,11}$/.test(m);
        }

        // Salvar dados ao fechar a janela
        window.addEventListener('beforeunload', function() {
            salvarDados();
        });

        // Preencher dados ao selecionar colaborador para edição
        document.getElementById('editColabId').addEventListener('change', function() {
            const id = parseInt(this.value);
            const colab = colaboradores.find(c => c.id === id);
            if (colab) {
                document.getElementById('editColabNome').value = colab.nome;
                document.getElementById('editColabMatricula').value = colab.matricula;
                document.getElementById('editColabCargo').value = colab.cargo;
                document.getElementById('editColabTurno').value = colab.turno;
            }
        });

        // Caso queira popular selects sempre que abrir os modais, adicionamos listeners para carregá-los dinamicamente
        document.getElementById('editColabBtn').addEventListener('click', () => carregarSelectColaboradores('editColabId'));
        document.getElementById('deleteColabBtn').addEventListener('click', () => carregarSelectColaboradores('deleteColabId'));
        document.getElementById('trocaTurnoBtn').addEventListener('click', () => { carregarSelectColaboradores('substitutoId'); carregarSelectColaboradores('substituidoId'); });
        document.getElementById('registrarPontoBtn').addEventListener('click', () => carregarSelectColaboradoresAtivos('pontoColabId'));

    </script>
</body>
</html>

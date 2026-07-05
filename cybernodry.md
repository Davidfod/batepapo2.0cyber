<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cyber Chat & Call 3.0</title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-database.js"></script>

    <style>
        /* Cores e Configurações Gerais */
        :root {
            --bg-main: #0f1115;
            --bg-sidebar: #161920;
            --bg-card: #1f232b;
            --accent: #00ffcc;
            --accent-hover: #00ccaa;
            --text-main: #f5f6f8;
            --text-muted: #8a94a6;
            --border-color: rgba(255, 255, 255, 0.06);
        }

        * { 
            margin: 0; 
            padding: 0; 
            box-sizing: border-box; 
            font-family: 'Segoe UI', -apple-system, BlinkMacSystemFont, Roboto, sans-serif; 
        }

        body { 
            background: var(--bg-main); 
            color: var(--text-main); 
            display: flex; 
            height: 100vh; 
            overflow: hidden; 
        }

        /* Barra Lateral Esquerda (Canais) */
        .sidebar-left { 
            width: 260px; 
            background: var(--bg-sidebar); 
            border-right: 1px solid var(--border-color); 
            display: flex; 
            flex-direction: column; 
            padding: 24px 16px; 
            justify-content: space-between; 
        }

        .sidebar-left h2 { 
            font-size: 0.85rem; 
            margin-bottom: 16px; 
            color: var(--text-muted); 
            text-transform: uppercase; 
            letter-spacing: 1.5px; 
            padding-left: 8px;
        }

        .btn-call { 
            width: 100%; 
            display: flex; 
            align-items: center; 
            gap: 14px; 
            background: transparent; 
            border: 1px solid transparent; 
            color: var(--text-muted); 
            padding: 12px 16px; 
            border-radius: 12px; 
            cursor: pointer; 
            font-weight: 600; 
            transition: all 0.25s ease; 
        }

        .btn-call i { 
            background: #2b2f3a; 
            padding: 10px; 
            border-radius: 50%; 
            font-size: 0.95rem; 
            width: 36px; 
            height: 36px; 
            display: flex; 
            align-items: center; 
            justify-content: center; 
            transition: all 0.25s ease;
            color: var(--text-main);
        }

        .btn-call:hover {
            background: rgba(255, 255, 255, 0.03);
            color: var(--text-main);
        }

        .btn-call.active { 
            border-color: rgba(0, 255, 204, 0.2); 
            background: rgba(0, 255, 204, 0.06); 
            color: var(--accent);
        }

        .btn-call.active i { 
            background: var(--accent); 
            color: #000000; 
            box-shadow: 0 0 12px rgba(0, 255, 204, 0.4);
        }

        .version-text { 
            font-size: 0.7rem; 
            color: var(--text-muted); 
            letter-spacing: 2px; 
            font-weight: bold; 
            text-align: center;
            opacity: 0.5;
        }

        /* Área Central do Chat */
        .chat-area { 
            flex: 1; 
            display: flex; 
            flex-direction: column; 
            background: var(--bg-main); 
        }

        .chat-header { 
            padding: 20px 24px; 
            background: var(--bg-sidebar); 
            border-bottom: 1px solid var(--border-color); 
            font-weight: 700; 
            font-size: 1.1rem; 
            display: flex;
            align-items: center;
            gap: 10px;
        }

        .chat-header i {
            color: var(--text-muted);
            font-size: 1.2rem;
        }

        .chat-messages { 
            flex: 1; 
            padding: 24px; 
            overflow-y: auto; 
            display: flex; 
            flex-direction: column; 
            gap: 20px; 
        }

        /* Estilização das Mensagens */
        .message { 
            display: flex; 
            gap: 16px; 
            padding: 4px 8px;
            border-radius: 8px;
            transition: background 0.15s ease;
        }

        .message:hover {
            background: rgba(255, 255, 255, 0.01);
        }

        .message img { 
            width: 44px; 
            height: 44px; 
            border-radius: 50%; 
            object-fit: cover; 
            border: 2px solid rgba(255, 255, 255, 0.08); 
        }

        .message-content { 
            display: flex; 
            flex-direction: column; 
            gap: 4px;
        }

        .message .user-name { 
            font-size: 0.95rem; 
            color: var(--accent); 
            font-weight: 600; 
        }

        .message .user-text { 
            font-size: 0.98rem; 
            color: var(--text-main);
            white-space: pre-wrap; 
            word-break: break-word; 
            line-height: 1.4;
        }

        /* Input de Texto */
        .chat-input-container { 
            padding: 24px; 
            background: var(--bg-main);
        }

        .chat-input-form { 
            display: flex; 
            background: var(--bg-card);
            border-radius: 14px;
            padding: 6px 8px;
            border: 1px solid var(--border-color);
            transition: border-color 0.2s ease;
        }

        .chat-input-form:focus-within {
            border-color: rgba(0, 255, 204, 0.4);
        }

        .chat-input { 
            flex: 1; 
            background: transparent;
            border: none;
            padding: 14px 16px; 
            color: var(--text-main); 
            outline: none; 
            font-size: 1rem; 
        }

        .chat-input::placeholder {
            color: var(--text-muted);
            opacity: 0.7;
        }

        /* Barra Lateral Direita (Perfil) */
        .sidebar-right { 
            width: 300px; 
            background: var(--bg-sidebar); 
            border-left: 1px solid var(--border-color); 
            padding: 24px; 
            display: flex; 
            flex-direction: column; 
            gap: 28px; 
        }

        .sidebar-right h2 {
            font-size: 0.85rem;
            color: var(--text-muted);
            text-transform: uppercase;
            letter-spacing: 1.5px;
        }

        .profile-card { 
            background: var(--bg-card); 
            padding: 28px 20px; 
            border-radius: 20px; 
            text-align: center; 
            border: 1px solid var(--border-color); 
            box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
        }

        .profile-card img { 
            width: 90px; 
            height: 90px; 
            border-radius: 50%; 
            object-fit: cover; 
            margin-bottom: 16px; 
            border: 3px solid var(--accent); 
            box-shadow: 0 0 15px rgba(0, 255, 204, 0.2);
        }

        .profile-card h3 { 
            font-size: 1.2rem; 
            color: var(--text-main); 
            font-weight: 600;
        }

        /* Customização dos Inputs de Edição */
        .edit-profile-box { 
            display: flex; 
            flex-direction: column; 
            gap: 18px; 
        }

        .edit-profile-box label { 
            font-size: 0.75rem; 
            color: var(--text-muted); 
            text-transform: uppercase; 
            font-weight: 700; 
            letter-spacing: 1px;
            margin-bottom: 6px;
            display: block;
        }

        .edit-profile-box input { 
            width: 100%;
            background: var(--bg-card); 
            border: 1px solid var(--border-color); 
            padding: 14px; 
            border-radius: 10px; 
            color: var(--text-main); 
            outline: none; 
            font-size: 0.95rem; 
            transition: border-color 0.2s ease;
        }

        .edit-profile-box input:focus {
            border-color: var(--accent);
        }

        .btn-save { 
            background: var(--accent); 
            color: #0b0c10; 
            border: none; 
            padding: 14px; 
            border-radius: 10px; 
            font-weight: 700; 
            cursor: pointer; 
            transition: all 0.2s ease;
            font-size: 0.95rem;
            margin-top: 4px;
        }

        .btn-save:hover { 
            background: var(--accent-hover);
            transform: translateY(-1px);
            box-shadow: 0 4px 12px rgba(0, 255, 204, 0.2);
        }

        /* Barra de rolagem customizada e moderna */
        ::-webkit-scrollbar { width: 6px; }
        ::-webkit-scrollbar-track { background: transparent; }
        ::-webkit-scrollbar-thumb { background: #2b303b; border-radius: 10px; }
        ::-webkit-scrollbar-thumb:hover { background: #3a4150; }
    </style>
</head>
<body>

    <div class="sidebar-left">
        <div>
            <h2>Canais de Voz</h2>
            <button class="btn-call" id="callBtn" onclick="toggleCall()">
                <i class="fa-solid fa-phone"></i>
                <span>Call 1</span>
            </button>
        </div>
        <div class="version-text">SISTEMA PRIVADO 3.0</div>
    </div>

    <div class="chat-area">
        <div class="chat-header">
            <i class="fa-solid fa-hashtag"></i> <span>chat-geral</span>
        </div>
        <div class="chat-messages" id="chatMessages"></div>
        <div class="chat-input-container">
            <form class="chat-input-form" onsubmit="sendMessage(event)">
                <input type="text" id="messageInput" class="chat-input" placeholder="Conversar em #chat-geral" autocomplete="off" maxlength="500">
            </form>
        </div>
    </div>

    <div class="sidebar-right">
        <h2>Seu Perfil</h2>
        <div class="profile-card">
            <img id="currentAvatar" src="" alt="Sua Foto">
            <h3 id="currentName">Carregando...</h3>
        </div>
        <div class="edit-profile-box">
            <div>
                <label>Nome de Usuário</label>
                <input type="text" id="inputName" maxlength="25">
            </div>
            <div>
                <label>Link da Foto (URL)</label>
                <input type="text" id="inputAvatar" placeholder="https://...">
            </div>
            <button class="btn-save" onclick="updateProfile()">Salvar Alterações</button>
        </div>
    </div>

    <script>
        // Mantendo suas configurações do Firebase intactas e funcionando!
        const firebaseConfig = {
            apiKey: "AIzaSyDegW2sEjWvx3Ih7U6vs45Gp8PeXyfJ90w",
            authDomain: "cyber-a41d9.firebaseapp.com",
            projectId: "cyber-a41d9",
            storageBucket: "cyber-a41d9.appspot.com",
            messagingSenderId: "563443982918",
            appId: "1:563443982918:web:61cf4b88f1b4ddd89ee047",
            databaseURL: "https://cyber-a41d9-default-rtdb.firebaseio.com/"
        };
        
        // Inicializando o Banco de Dados
        firebase.initializeApp(firebaseConfig);
        const database = firebase.database();
        const messagesRef = database.ref('messages');

        const DEFAULT_AVATAR = 'https://picsum.photos/id/64/150/150';
        let localStream = null;
        let currentUser = { name: '', avatar: '' };

        function initUser() {
            const savedName = localStorage.getItem('cyber_name');
            const savedAvatar = localStorage.getItem('cyber_avatar');
            currentUser.name = savedName || "CyberUser_" + Math.floor(Math.random() * 900 + 100);
            currentUser.avatar = savedAvatar || `https://picsum.photos/id/${Math.floor(Math.random() * 50) + 10}/150/150`;
            
            document.getElementById('currentName').innerText = currentUser.name;
            document.getElementById('currentAvatar').src = currentUser.avatar;
            document.getElementById('inputName').value = currentUser.name;
            document.getElementById('inputAvatar').value = savedAvatar || '';
        }
        initUser();

        // Escuta novas mensagens em tempo real
        messagesRef.limitToLast(50).on('child_added', (snapshot) => {
            const data = snapshot.val();
            const chatMessages = document.getElementById('chatMessages');
            const messageDiv = document.createElement('div');
            messageDiv.classList.add('message');

            const img = document.createElement('img');
            img.src = data.avatar || DEFAULT_AVATAR;
            img.onerror = function() { this.src = DEFAULT_AVATAR; };

            const contentDiv = document.createElement('div');
            contentDiv.classList.add('message-content');

            const nameDiv = document.createElement('div');
            nameDiv.classList.add('user-name');
            nameDiv.textContent = data.name;

            const textDiv = document.createElement('div');
            textDiv.classList.add('user-text');
            textDiv.textContent = data.text;

            contentDiv.appendChild(nameDiv);
            contentDiv.appendChild(textDiv);
            messageDiv.appendChild(img);
            messageDiv.appendChild(contentDiv);
            chatMessages.appendChild(messageDiv);
            chatMessages.scrollTop = chatMessages.scrollHeight;
        });

        function updateProfile() {
            const nameInput = document.getElementById('inputName').value.trim();
            const avatarInput = document.getElementById('inputAvatar').value.trim();
            if (nameInput !== "") currentUser.name = nameInput;
            if (avatarInput !== "") currentUser.avatar = avatarInput;

            localStorage.setItem('cyber_name', currentUser.name);
            localStorage.setItem('cyber_avatar', avatarInput);
            document.getElementById('currentName').innerText = currentUser.name;
            document.getElementById('currentAvatar').src = currentUser.avatar;
        }

        function sendMessage(event) {
            event.preventDefault();
            const input = document.getElementById('messageInput');
            const text = input.value.trim();
            if (text !== '') {
                messagesRef.push({
                    name: currentUser.name,
                    avatar: currentUser.avatar,
                    text: text
                });
                input.value = '';
            }
        }

        async function toggleCall() {
            const btn = document.getElementById('callBtn');
            if (!btn.classList.contains('active')) {
                try {
                    localStream = await navigator.mediaDevices.getUserMedia({ audio: true });
                    btn.classList.add('active');
                    btn.querySelector('span').innerText = 'Em Call (Ativo)';
                } catch (err) {
                    alert('Permissão de áudio recusada.');
                }
            } else {
                if (localStream) {
                    localStream.getTracks().forEach(track => track.stop());
                    localStream = null;
                }
                btn.classList.remove('active');
                btn.querySelector('span').innerText = 'Call 1';
            }
        }
    </script>
</body>
</html>

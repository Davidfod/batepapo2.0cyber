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
        * { margin: 0; padding: 0; box-sizing: border-box; font-family: 'Segoe UI', sans-serif; }
        body { background: #0b0c10; color: #ffffff; display: flex; height: 100vh; overflow: hidden; }
        .sidebar-left { width: 260px; background: #141519; border-right: 1px solid rgba(0, 255, 204, 0.1); display: flex; flex-direction: column; padding: 20px; justify-content: space-between; }
        .sidebar-left h2 { font-size: 1rem; margin-bottom: 20px; color: #00ffcc; text-transform: uppercase; letter-spacing: 1.5px; }
        .btn-call { width: 100%; display: flex; align-items: center; gap: 12px; background: #1f2026; border: 1px solid rgba(255, 255, 255, 0.05); color: #ffffff; padding: 12px; border-radius: 50px; cursor: pointer; font-weight: 600; transition: all 0.3s ease; }
        .btn-call i { background: #5865F2; padding: 10px; border-radius: 50%; font-size: 1rem; width: 36px; height: 36px; display: flex; align-items: center; justify-content: center; }
        .btn-call.active { border-color: #00ffcc; background: #132220; box-shadow: 0 0 15px rgba(0, 255, 204, 0.2); }
        .btn-call.active i { background: #00ffcc; color: #0b0c10; }
        .version-text { font-size: 0.75rem; color: rgba(0, 255, 204, 0.3); letter-spacing: 1px; font-weight: bold; }
        .chat-area { flex: 1; display: flex; flex-direction: column; background: #1c1d24; }
        .chat-header { padding: 20px; background: #141519; border-bottom: 1px solid rgba(255, 255, 255, 0.05); font-weight: bold; font-size: 1.1rem; }
        .chat-messages { flex: 1; padding: 20px; overflow-y: auto; display: flex; flex-direction: column; gap: 16px; }
        .message { display: flex; gap: 14px; background: rgba(255, 255, 255, 0.02); padding: 14px; border-radius: 12px; width: fit-content; max-width: 75%; border: 1px solid rgba(255, 255, 255, 0.02); }
        .message img { width: 42px; height: 42px; border-radius: 50%; object-fit: cover; border: 1px solid rgba(0, 255, 204, 0.2); }
        .message-content { display: flex; flex-direction: column; }
        .message .user-name { font-size: 0.9rem; color: #00ffcc; font-weight: bold; margin-bottom: 4px; }
        .message .user-text { font-size: 0.95rem; white-space: pre-wrap; word-break: break-word; }
        .chat-input-container { padding: 20px; background: #141519; border-top: 1px solid rgba(255, 255, 255, 0.05); }
        .chat-input-form { display: flex; gap: 12px; }
        .chat-input { flex: 1; background: #1f2026; border: 1px solid rgba(255, 255, 255, 0.08); padding: 16px; border-radius: 10px; color: #ffffff; outline: none; font-size: 1rem; }
        .chat-input:focus { border-color: #00ffcc; }
        .sidebar-right { width: 280px; background: #141519; border-left: 1px solid rgba(0, 255, 204, 0.1); padding: 20px; display: flex; flex-direction: column; gap: 24px; }
        .profile-card { background: #1c1d24; padding: 24px 20px; border-radius: 16px; text-align: center; border: 1px solid rgba(255, 255, 255, 0.05); }
        .profile-card img { width: 84px; height: 84px; border-radius: 50%; object-fit: cover; margin-bottom: 12px; border: 2px solid #0077ff; }
        .profile-card h3 { font-size: 1.1rem; color: #ffffff; }
        .edit-profile-box { display: flex; flex-direction: column; gap: 16px; }
        .edit-profile-box label { font-size: 0.8rem; color: #888; text-transform: uppercase; font-weight: bold; }
        .edit-profile-box input { background: #1c1d24; border: 1px solid rgba(255, 255, 255, 0.08); padding: 12px; border-radius: 8px; color: #ffffff; outline: none; font-size: 0.95rem; }
        .btn-save { background: #00ffcc; color: #0b0c10; border: none; padding: 12px; border-radius: 8px; font-weight: bold; cursor: pointer; }
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
            <i class="fa-solid fa-hashtag"></i> chat-geral
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
        // Configurações do seu Firebase vinculadas ao Realtime Database
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

        // Escuta novas mensagens em tempo real para todos os conectados
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

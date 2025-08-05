# OpenXI
OpenXI
<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <title>XIAI — Математический Ассистент</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <script src="https://cdnjs.cloudflare.com/ajax/libs/mathjs/11.11.0/math.min.js"></script>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: linear-gradient(to bottom right, #dbe9f4, #f7f9fc);
      margin: 0;
      padding: 0;
    }

    .container {
      max-width: 600px;
      margin: auto;
      padding: 30px;
      background-color: white;
      box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
      border-radius: 12px;
      margin-top: 50px;
    }

    h2 {
      text-align: center;
      color: #333;
    }

    form {
      display: none;
      margin-top: 20px;
    }

    label {
      display: block;
      margin: 10px 0 5px;
    }

    input[type="text"], input[type="password"] {
      width: 100%;
      padding: 10px;
      margin-bottom: 15px;
      border-radius: 6px;
      border: 1px solid #ccc;
    }

    button {
      padding: 10px 20px;
      background-color: #0099cc;
      color: white;
      border: none;
      border-radius: 6px;
      cursor: pointer;
      font-size: 16px;
    }

    button:hover {
      background-color: #007799;
    }

    .links {
      text-align: center;
      margin-top: 20px;
    }

    .links a {
      margin: 0 10px;
      color: #0099cc;
      cursor: pointer;
      text-decoration: underline;
    }

    #chatBox {
      height: 300px;
      overflow-y: auto;
      border: 1px solid #ccc;
      padding: 10px;
      margin-bottom: 10px;
    }

    .user { color: #000099; margin-bottom: 10px; }
    .bot { color: #009900; margin-bottom: 10px; }

    #chat {
      display: none;
    }

    #history {
      display: none;
      margin-top: 20px;
      border-top: 1px solid #ccc;
      padding-top: 10px;
    }
  </style>
</head>
<body>
  <div class="container">
    <h2>Добро пожаловать в XIAI</h2>

    <div class="links">
      <a onclick="showForm('login')">Вход</a> |
      <a onclick="showForm('register')">Регистрация</a>
    </div>

    <!-- Вход -->
    <form id="loginForm">
      <label>Имя:</label>
      <input type="text" id="loginName" required>
      <label>Пароль:</label>
      <input type="password" id="loginPass" required>
      <button type="button" onclick="login()">Войти</button>
    </form>

    <!-- Регистрация -->
    <form id="registerForm">
      <label>Имя:</label>
      <input type="text" id="regName" required>
      <label>Пароль:</label>
      <input type="password" id="regPass" required>
      <label><input type="checkbox" id="agree"> Я согласен с условиями</label><br><br>
      <button type="button" onclick="register()">Зарегистрироваться</button>
    </form>

    <!-- Чат -->
    <div id="chat">
      <h3>Чат XIAI</h3>
      <div id="chatBox"></div>
      <input type="text" id="userInput" placeholder="Напиши математический вопрос...">
      <button onclick="sendMessage()">Отправить</button>
      <button onclick="showHistory()">Мои решения</button>
    </div>

    <!-- Архив -->
    <div id="history">
      <h4>Архив решений:</h4>
      <ul id="historyList"></ul>
    </div>
  </div>

  <script>
    function showForm(formId) {
      document.getElementById('loginForm').style.display = 'none';
      document.getElementById('registerForm').style.display = 'none';
      document.getElementById('chat').style.display = 'none';
      document.getElementById('history').style.display = 'none';
      document.getElementById(formId + 'Form').style.display = 'block';
    }

    let currentUser = null;

    function login() {
      const name = document.getElementById('loginName').value;
      const pass = document.getElementById('loginPass').value;
      if (name && pass) {
        currentUser = name;
        alert("Добро пожаловать, " + name + "!");
        document.getElementById('loginForm').style.display = 'none';
        document.getElementById('chat').style.display = 'block';
        loadHistory();
      } else {
        alert("Введите имя и пароль.");
      }
    }

    function register() {
      const name = document.getElementById('regName').value;
      const pass = document.getElementById('regPass').value;
      const agree = document.getElementById('agree').checked;
      if (name && pass && agree) {
        alert("Успешная регистрация! Теперь войдите.");
        showForm('login');
      } else {
        alert("Заполните все поля и согласитесь с условиями.");
      }
    }

    const chatBox = document.getElementById('chatBox');

    function sendMessage() {
      const input = document.getElementById('userInput');
      const userText = input.value.trim();
      if (userText === '') return;

      appendMessage('user', userText);
      input.value = '';

      setTimeout(() => {
        const botReply = getBotReply(userText);
        appendMessage('bot', botReply);
        saveToHistory(userText, botReply);
      }, 500);
    }

    function appendMessage(sender, text) {
      const msg = document.createElement('div');
      msg.className = sender;
      msg.textContent = sender === 'user' ? "Вы: " + text : "XIAI: " + text;
      chatBox.appendChild(msg);
      chatBox.scrollTop = chatBox.scrollHeight;
    }

    function getBotReply(text) {
      try {
        if (text.includes('=')) {
          const eq = math.parse(text);
          const simplified = math.simplify(eq);
          const solution = math.solve(text, 'x');
          return `Решение: x = ${solution}`;
        }

        const result = math.evaluate(text);
        return `Ответ: ${result}`;
      } catch (e) {
        return "Ошибка: Некорректный пример или уравнение.";
      }
    }

    function saveToHistory(question, answer) {
      if (!currentUser) return;
      const historyKey = `history_${currentUser}`;
      const existing = JSON.parse(localStorage.getItem(historyKey)) || [];
      existing.push({ question, answer, time: new Date().toLocaleString() });
      localStorage.setItem(historyKey, JSON.stringify(existing));
    }

    function loadHistory() {
      if (!currentUser) return;
      const historyKey = `history_${currentUser}`;
      const history = JSON.parse(localStorage.getItem(historyKey)) || [];
      const list = document.getElementById('historyList');
      list.innerHTML = '';
      history.forEach(item => {
        const li = document.createElement('li');
        li.textContent = `[${item.time}] ${item.question} => ${item.answer}`;
        list.appendChild(li);
      });
    }

    function showHistory() {
      loadHistory();
      document.getElementById('history').style.display = 'block';
    }
  </script>
</body>
</html>

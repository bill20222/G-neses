# CooberGeneses

<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Chatbot</title>

    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 20px;
            background-color: #f7f7f7;
        }

        .chatbox {
            border: 1px solid #ccc;
            max-width: 600px;
            margin: auto;
            background-color: #fff;
            border-radius: 10px;
            box-shadow: 0px 2px 5px rgba(0, 0, 0, 0.1);
            overflow: hidden;
        }

        .chat-header {
            background-color: #4CAF50;
            color: #fff;
            padding: 10px;
            font-weight: bold;
            text-align: center;
        }

        .chat-messages {
            max-height: 400px;
            overflow-y: scroll;
            padding: 10px;
        }

        .chat-message {
            margin: 10px 0;
        }

        .user-message {
            text-align: right;
            color: #4CAF50;
        }

        .bot-message {
            text-align: left;
            color: #333;
        }

        .chat-input {
            width: calc(100% - 20px);
            padding: 10px;
            border: none;
            border-top: 1px solid #ccc;
            font-size: 16px;
        }

        .chat-input:focus {
            outline: none;
        }

        .chat-send-button {
            width: 20px;
            height: 20px;
            border: none;
            background-color: #4CAF50;
            color: #fff;
            border-radius: 50%;
            cursor: pointer;
            font-size: 14px;
            margin-left: 10px;
        }

        .chat-send-button:focus {
            outline: none;
        }

        .chat-options {
            text-align: center;
            margin-top: 10px;
        }
    </style>
</head>
<body>
    <div class="chatbox">
        <div class="chat-header">
            Chatbot
        </div>
        <div class="chat-messages" id="chat"></div>
        <div class="chat-input-container">
            <input type="text" class="chat-input" id="userInput" placeholder="Digite sua mensagem...">
            <button class="chat-send-button" onclick="sendMessage()">➤</button>
        </div>
        <div class="chat-options" id="options">
            <button onclick="consultarRespostas()">Consultar Respostas</button>
            <div id="respostasSalvas" style="display:none;">
                <h2>Respostas Salvas:</h2>
                <ul id="respostasList"></ul>
            </div>
        </div>
    </div>

    <div id="questionPanel" style="display:none;">
        <h2>Pergunta:</h2>
        <textarea id="questionText" rows="4" cols="50"></textarea>
        <button onclick="submitQuestion()">Enviar Pergunta</button>
    </div>

    <div id="answerPanel" style="display:none;">
        <h2>Resposta:</h2>
        <textarea id="answerText" rows="4" cols="50"></textarea>
        <button onclick="submitAnswer()">Enviar Resposta</button>
    </div>

    <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
    <script src="https://www.gstatic.com/dialogflow-console/fast/messenger/bootstrap.js?v=1"></script>

    <script>
        var qaPairs = JSON.parse(localStorage.getItem('qaPairs')) || [];

        document.getElementById('userInput').addEventListener('keypress', function(event) {
            if (event.key === 'Enter') {
                sendMessage();
            }
        });

        function falarMensagemBot(mensagem) {
            var synth = window.speechSynthesis;
            var utterance = new SpeechSynthesisUtterance(mensagem);
            synth.speak(utterance);
        }

        function generateBotResponse(message) {
            var botMessage = '<div class="chat-message bot-message">' + message + '</div>';
            var chat = document.getElementById('chat');
            chat.innerHTML += botMessage;
            falarMensagemBot(message); // Fala a resposta
        }

        function sendMessage() {
            var userInput = document.getElementById('userInput');
            var message = userInput.value;
            userInput.value = '';

            var chat = document.getElementById('chat');
            var userMessage = '<div class="chat-message user-message">' + message + '</div>';
            chat.innerHTML += userMessage;

            if (qaPairs.length > 0) {
                var foundAnswer = false;
                for (var i = 0; i < qaPairs.length; i++) {
                    var qaPair = qaPairs[i];
                    if (qaPair.question.toLowerCase() === message.toLowerCase()) {
                        generateBotResponse(qaPair.answer);
                        foundAnswer = true;
                        break;
                    }
                }
                if (!foundAnswer) {
                    if (isMathExpression(message)) {
                        var result = calculateMathExpression(message);
                        generateBotResponse('O resultado é ' + result);
                    } else {
                        askHowToProceed(message);
                    }
                }
            } else {
                askHowToProceed(message);
            }
        }

        function isMathExpression(message) {
            var regex = /^[\d+\-*/\s()]+$/;
            return regex.test(message);
        }

        function calculateMathExpression(expression) {
            try {
                return eval(expression);
            } catch (error) {
                return 'Desculpe, não consigo calcular isso.';
            }
        }

        function askHowToProceed(question) {
            var chat = document.getElementById('chat');
            var botMessage = '<div class="chat-message bot-message">Como proceder com esta informação?</div>';
            chat.innerHTML += botMessage;

            var questionPanel = document.getElementById('questionPanel');
            questionPanel.style.display = 'block';
            document.getElementById('questionText').value = question;
        }

        function submitQuestion() {
            var questionText = document.getElementById('questionText').value;
            var answerPanel = document.getElementById('answerPanel');
            answerPanel.style.display = 'block';
            answerPanel.scrollIntoView();
        }

        function submitAnswer() {
            var question = document.getElementById('questionText').value;
            var answer = document.getElementById('answerText').value;

            qaPairs.push({ question: question, answer: answer });
            localStorage.setItem('qaPairs', JSON.stringify(qaPairs));

            var chat = document.getElementById('chat');
            var botMessage = '<div class="chat-message bot-message">Obrigado por me ensinar!</div>';
            chat.innerHTML += botMessage;

            var questionPanel = document.getElementById('questionPanel');
            var answerPanel = document.getElementById('answerPanel');
            questionPanel.style.display = 'none';
            answerPanel.style.display = 'none';
        }

        function consultarRespostas() {
            var respostasList = document.getElementById('respostasList');
            respostasList.innerHTML = '';

            if (qaPairs.length > 0) {
                for (var i = 0; i < qaPairs.length; i++) {
                    var qaPair = qaPairs[i];
                    var listItem = document.createElement('li');
                    listItem.innerText = 'Pergunta: ' + qaPair.question + ' - Resposta: ' + qaPair.answer;
                    respostasList.appendChild(listItem);
                }

                var respostasSalvas = document.getElementById('respostasSalvas');
                respostasSalvas.style.display = 'block';
            } else {
                alert('Nenhuma resposta salva ainda.');
            }
        }
    </script>
</body>
</html>


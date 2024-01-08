<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Jogo de Acertar a Palavra</title>
  <style>
    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      text-align: center;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      height: 100vh;
      margin: 0;
      background-color: #282c34; /* Cor de fundo alterada para um tom escuro */
      color: #61dafb; /* Cor do texto alterada para azul claro */
    }

    input {
      width: 200px;
      text-align: center;
      margin-top: 30px;
      padding: 10px;
      font-size: 16px;
      background-color: #61dafb; /* Cor do fundo do input */
      color: #282c39; /* Cor do texto do input */
      border: none;
      border-radius: 20px;
      transition: background-color 0.3s ease;
    }

    input:focus {
      outline: none;
      background-color: #91a7ff; /* Cor do fundo do input ao focar */
    }

    #word-container {
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
    }

    .letter-box {
      width: 30px;
      height: 30px;
      margin: 5px;
      border: 1px solid #888;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 18px;
      color: #555;
      background-color: #ddd;
      transition: background-color 0.3s ease;
    }

    .letter-reveal {
      background-color: #61dafb; /* Cor do fundo quando a letra é revelada */
    }

    #chances-container {
      margin-top: 20px;
      font-size: 20px;
      color: #555;
    }

    #message {
      font-size: 20px;
      margin-top: 20px;
      display: none;
      position: relative;
      z-index: 2;
      color: #ff3333;
    }

    #congrats-message, #lose-message {
      display: none;
      font-size: 20px;
      color: white;
      padding: 20px;
      border-radius: 10px;
      position: fixed;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      z-index: 1;
      width: 80%; /* Ajuste a largura conforme necessário */
      max-width: 600px; /* Adicione um valor máximo para evitar que fique muito largo */
      background-color: #4CAF50; /* Cor de fundo para o parabéns */
    }

    #lose-message {
      background-color: #000; /* Cor de fundo para o perdeu */
    }

    #next-button {
      margin-top: 20px;
      padding: 10px 20px;
      font-size: 18px;
      cursor: pointer;
      background-color: #555;
      color: white;
      border: none;
      border-radius: 8px;
      display: none;
      transition: background-color 0.3s ease;
    }

    #next-button:hover {
      background-color: #777;
    }

    button {
      margin-top: 20px;
      padding: 15px 30px;
      font-size: 18px;
      cursor: pointer;
      background-color: #61dafb;
      color: #282c34;
      border: none;
      border-radius: 8px;
      transition: background-color 0.3s ease;
    }

    button:hover {
      background-color: #91a7ff;
    }
  </style>
</head>
<body>

<h1>Jogo de Acertar a Palavra</h1>

<div id="hint-container"></div>
<div id="word-container"></div>
<div id="chances-container"></div>
<input type="text" id="guessInput" placeholder="Digite uma letra">
<button id="guessButton" onclick="makeGuess()">Enviar Palpite</button>
<div id="message"></div>
<div id="congrats-message">
  <p>Parabéns, resposta correta!</p>
  <p>A palavra correta era: <span id="correct-word"></span></p>
</div>
<div id="lose-message">
  <p>Você perdeu! A palavra era: <span id="lost-word"></span></p>
</div>
<button id="next-button" onclick="nextWord()">Próximo</button>

<audio id="correct-sound" src="caminho/para/correct-sound.mp3"></audio>
<audio id="incorrect-sound" src="caminho/para/incorrect-sound.mp3"></audio>

<script>
  var wordData = [
    { word: "banana", hint: "Fruta amarela alongada" },
    { word: "abacaxi", hint: "Fruta tropical com casca espinhosa" },
    { word: "morango", hint: "Fruta vermelha pequena e doce" },
    { word: "uva", hint: "Fruta pequena e arredondada, geralmente usada para fazer vinho" }
  ];

  function shuffleArray(array) {
    for (var i = array.length - 1; i > 0; i--) {
      var j = Math.floor(Math.random() * (i + 1));
      var temp = array[i];
      array[i] = array[j];
      array[j] = temp;
    }
    return array;
  }

  wordData = shuffleArray(wordData);

  var currentWordIndex = 0;
  var wordInfo = wordData[currentWordIndex];
  var word = wordInfo.word;

  var wordLetters = word.split('');
  var guessedLetters = new Array(word.length).fill('_');
  var chances = 6;
  var maxRows = 6;

  document.getElementById('hint-container').innerHTML = 'Dica: ' + wordInfo.hint;

  function displayWordAndChances() {
    var wordContainer = document.getElementById('word-container');
    wordContainer.innerHTML = '';

    for (var i = 0; i < guessedLetters.length; i++) {
      var letterBox = document.createElement('div');
      letterBox.classList.add('letter-box');
      if (guessedLetters[i] !== '_') {
        letterBox.classList.add('letter-reveal');
      }
      letterBox.innerText = guessedLetters[i];
      wordContainer.appendChild(letterBox);
    }

    var remainingRows = maxRows - Math.ceil(guessedLetters.length / 6);
    for (var j = 0; j < remainingRows; j++) {
      var row = document.createElement('div');
      row.classList.add('letter-row');
      wordContainer.appendChild(row);
    }

    document.getElementById('chances-container').innerHTML = 'Chances restantes: ' + chances;
  }

  function displayMessage(message, isError = false) {
    var messageElement = document.getElementById('message');
    messageElement.innerHTML = message;
    messageElement.style.color = isError ? '#ff3333' : '#61dafb';
    messageElement.style.display = 'block';
  }

  function displayCongratsMessage() {
    document.getElementById('congrats-message').style.display = 'block';
    document.getElementById('correct-word').innerText = word;
  }

  function displayLoseMessage() {
    document.getElementById('lose-message').style.display = 'block';
    document.getElementById('lost-word').innerText = word;
  }

  function hideCongratsMessage() {
    document.getElementById('congrats-message').style.display = 'none';
  }

  function showNextButton() {
    document.getElementById('next-button').style.display = 'block';
  }

  function makeGuess() {
    document.getElementById('guessInput').disabled = true;
    document.getElementById('guessButton').disabled = true;

    var guess = document.getElementById('guessInput').value.toLowerCase();

    if (guess === word) {
      displayCongratsMessage();
      showNextButton();
      document.getElementById('message').style.display = 'none';
      document.getElementById('correct-sound').play();
      return;
    } else if (guess.length !== 1 || !/[a-z]/.test(guess)) {
      displayMessage('Por favor, digite uma letra válida.');
      document.getElementById('guessInput').disabled = false;
      document.getElementById('guessButton').disabled = false;
      return;
    } else if (guessedLetters.includes(guess)) {
      displayMessage('Você já tentou essa letra. Tente outra.');
      document.getElementById('guessInput').disabled = false;
      document.getElementById('guessButton').disabled = false;
      return;
    }

    var correctGuess = false;
    for (var i = 0; i < wordLetters.length; i++) {
      if (wordLetters[i] === guess) {
        guessedLetters[i] = guess;
        correctGuess = true;
      }
    }

    if (!correctGuess) {
      chances--;
      document.getElementById('incorrect-sound').play();
    } else {
      document.getElementById('correct-sound').play();
    }

    if (chances === 0) {
      displayLoseMessage();
      showNextButton();
    }

    if (guessedLetters.join('') === word) {
      displayCongratsMessage();
      showNextButton();
      document.getElementById('message').style.display = 'none';
      return;
    }

    document.getElementById('guessInput').disabled = false;
    document.getElementById('guessButton').disabled = false;

    displayWordAndChances();
    document.getElementById('guessInput').value = '';
  }

  function nextWord() {
    hideCongratsMessage();
    document.getElementById('lose-message').style.display = 'none';
    document.getElementById('next-button').style.display = 'none';

    document.getElementById('guessInput').disabled = false;
    document.getElementById('guessButton').disabled = false;

    guessedLetters = new Array(word.length).fill('_');
    chances = 6;

    currentWordIndex++;

    if (currentWordIndex >= wordData.length) {
      currentWordIndex = 0;
      wordData = shuffleArray(wordData);
    }

    wordInfo = wordData[currentWordIndex];
    word = wordInfo.word;
    wordLetters = word.split('');
    guessedLetters = new Array(word.length).fill('_');
    chances = 6;

    document.getElementById('hint-container').innerHTML = 'Dica: ' + wordInfo.hint;

    displayWordAndChances();
    document.getElementById('guessInput').value = '';
    document.getElementById('message').style.display = 'none';
  }

  displayWordAndChances();
</script>

</body>
</html>
# ding

<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">

  <title>ULTRON V0.1</title>

  <style>
    * {
      box-sizing: border-box;
    }

    body {
      margin: 0;
      min-height: 100vh;
      background: #050505;
      color: #00ff88;
      font-family: Arial, sans-serif;

      display: flex;
      justify-content: center;
      align-items: center;
    }

    .container {
      width: 90%;
      max-width: 500px;
      text-align: center;
    }

    .core {
      width: 130px;
      height: 130px;
      margin: 0 auto 30px;

      border: 3px solid #00ff88;
      border-radius: 50%;

      display: flex;
      justify-content: center;
      align-items: center;

      font-size: 20px;
      font-weight: bold;

      box-shadow:
        0 0 15px #00ff88,
        0 0 40px rgba(0, 255, 136, 0.4);

      transition: 0.3s;
    }

    .core.active {
      transform: scale(1.15);

      box-shadow:
        0 0 25px #00ff88,
        0 0 70px rgba(0, 255, 136, 0.8);
    }

    h1 {
      letter-spacing: 5px;
      margin-bottom: 10px;
    }

    #status {
      color: #aaa;
      margin-bottom: 25px;
    }

    #heard {
      min-height: 60px;
      padding: 15px;

      border: 1px solid #222;
      border-radius: 10px;

      background: #0b0b0b;
      color: white;

      margin-bottom: 20px;
    }

    button {
      width: 100%;
      padding: 15px;

      border: none;
      border-radius: 10px;

      background: #00ff88;
      color: #00150b;

      font-size: 16px;
      font-weight: bold;

      cursor: pointer;
    }

    button:active {
      transform: scale(0.98);
    }

    .detected {
      color: #00ff88 !important;
      font-weight: bold;
    }
  </style>
</head>

<body>

  <div class="container">

    <div id="core" class="core">
      ULTRON
    </div>

    <h1>ULTRON</h1>

    <div id="status">
      Sistema desligado
    </div>

    <div id="heard">
      Clique em iniciar e diga "ULTRON".
    </div>

    <button id="startButton">
      ATIVAR MICROFONE
    </button>

  </div>

  <script>

    // ==========================================
    // ULTRON V0.1
    // Detector da palavra "ULTRON"
    // ==========================================

    const SpeechRecognition =
      window.SpeechRecognition ||
      window.webkitSpeechRecognition;

    const status = document.getElementById("status");
    const heard = document.getElementById("heard");
    const button = document.getElementById("startButton");
    const core = document.getElementById("core");

    if (!SpeechRecognition) {

      status.textContent =
        "Seu navegador não suporta reconhecimento de voz.";

      button.disabled = true;

    } else {

      const recognition = new SpeechRecognition();

      // Português do Brasil
      recognition.lang = "pt-BR";

      // Continua tentando reconhecer novas frases
      recognition.continuous = true;

      // Queremos resultados enquanto a pessoa fala
      recognition.interimResults = true;

      recognition.maxAlternatives = 1;


      // ==========================================
      // BOTÃO
      // ==========================================

      button.addEventListener("click", () => {

        try {

          recognition.start();

          status.textContent =
            "ULTRON está ouvindo...";

          button.textContent =
            "MICROFONE ATIVO";

        } catch (error) {

          console.log(error);

        }

      });


      // ==========================================
      // QUANDO COMEÇA A ESCUTAR
      // ==========================================

      recognition.onstart = () => {

        status.textContent =
          "🟢 Ouvindo... diga ULTRON";

      };


      // ==========================================
      // QUANDO RECEBE VOZ
      // ==========================================

      recognition.onresult = (event) => {

        let texto = "";

        for (
          let i = event.resultIndex;
          i < event.results.length;
          i++
        ) {

          texto +=
            event.results[i][0].transcript + " ";

        }

        texto = texto.trim();

        heard.textContent =
          texto || "Ouvindo...";


        // ==========================================
        // DETECTOR
        // ==========================================

        const textoNormalizado =
          texto
            .normalize("NFD")
            .replace(/[\u0300-\u036f]/g, "")
            .toLowerCase();


        if (
          textoNormalizado.includes("ultron") ||
          textoNormalizado.includes("ultrão")
        ) {

          ativarUltron(texto);

        }

      };


      // ==========================================
      // ULTRON DETECTADO
      // ==========================================

      function ativarUltron(texto) {

        status.textContent =
          "⚡ ULTRON DETECTADO!";

        heard.textContent =
          "COMANDO DETECTADO: " + texto;

        heard.classList.add("detected");

        core.classList.add("active");


        // Vibração do celular, se disponível
        if (navigator.vibrate) {

          navigator.vibrate([
            100,
            50,
            100
          ]);

        }


        // Fala
        falar("Estou ouvindo.");


        setTimeout(() => {

          core.classList.remove("active");

          heard.classList.remove("detected");

          status.textContent =
            "🟢 ULTRON está ouvindo...";

        }, 2500);

      }


      // ==========================================
      // VOZ
      // ==========================================

      function falar(texto) {

        if (!("speechSynthesis" in window)) {
          return;
        }

        window.speechSynthesis.cancel();

        const voz =
          new SpeechSynthesisUtterance(texto);

        voz.lang = "pt-BR";

        voz.rate = 1;

        voz.pitch = 0.8;

        window.speechSynthesis.speak(voz);

      }


      // ==========================================
      // ERROS
      // ==========================================

      recognition.onerror = (event) => {

        console.log(
          "Erro:",
          event.error
        );

        if (event.error === "not-allowed") {

          status.textContent =
            "❌ Permissão do microfone negada.";

        }

        else {

          status.textContent =
            "⚠️ Erro no reconhecimento.";

        }

      };


      // ==========================================
      // SE O RECONHECIMENTO PARAR
      // ==========================================

      recognition.onend = () => {

        status.textContent =
          "Microfone parado.";

        button.textContent =
          "ATIVAR MICROFONE";

      };

    }

  </script>

</body>
</html>
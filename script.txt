// Variables globales
let input = document.getElementById("input"); // Elemento de entrada de texto
let summary = document.getElementById("summary"); // Elemento de salida de resumen
let labels = document.getElementById("labels"); // Elemento de salida de etiquetas
let improvedSummary = document.getElementById("improved-summary"); // Elemento de salida de resumen mejorado
let improvedLabels = document.getElementById("improved-labels"); // Elemento de salida de etiquetas mejoradas
let record = document.getElementById("record"); // Botón de iniciar grabación
let stop = document.getElementById("stop"); // Botón de detener grabación
let timer = document.getElementById("timer"); // Elemento de tiempo transcurrido
let generate = document.getElementById("generate"); // Botón de generar
let improve = document.getElementById("improve"); // Botón de mejorar

// Claves de acceso a las APIs
let chatgptKey = "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"; // Clave de acceso a la API de ChatGPT
let whisperKey = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiI1YWRiZjNkNTAyN2IxY2JkNDE4NzNiMjRlNzlhNjE5ZTQxODhiYTVlM2M5ZmM3ZDM1N2NmNjNkOGYxYjg3YmEzZjBjZmMyMDJiOTM2NTdhOGJkMWQ0MmZlNWYwNWY3N2Q5YzExYjZkNjE2NzE2NDE0ZWM1YWU2ODVhNjE0MjQxMCIsInNjb3BlIjoiY29uZmlnIiwiaXNzIjoiaHR0cHM6Ly9hcGkuc3BlZWNobHkuY29tLyIsImF1ZCI6Imh0dHBzOi8vYXBpLnNwZWVjaGx5LmNvbS8ifQ.EcziURlyqDVnoV2WdBd9SqEGhDT94Y8AlWkaqrlichM"; // Clave de acceso a la API de Whisper

// URLs de los prompts de ChatGPT en español
let summaryURL = "https://platform.openai.com/playground/p/GKBxw4DtPWS4D1rV6aQ5g3U9?model=text-davinci-003"; // URL del prompt para generar resúmenes
let labelsURL = "https://platform.openai.com/playground/p/03hVCuo4XUVYTAXfQf6CLADB?model=text-davinci-003"; // URL del prompt para generar etiquetas
let improvedSummaryURL = "https://platform.openai.com/playground/p/SA79pnbCoMIlnhr4vWUMPPyQ?model=text-davinci-003"; // URL del prompt para generar resúmenes mejorados
let improvedLabelsURL = "https://platform.openai.com/playground/p/ce7hev53lhsuPrZjs0rgZ4t2?model=text-davinci-003"; // URL del prompt para generar etiquetas mejoradas

// Variables para la grabación y el tiempo
let recorder; // Objeto para grabar audio
let startTime; // Tiempo inicial de la grabación
let interval; // Intervalo para actualizar el tiempo transcurrido

// Función para iniciar la grabación
function record() {
  // Crear un objeto MediaRecorder con el dispositivo de audio predeterminado
  navigator.mediaDevices.getUserMedia({ audio: true }).then(stream => {
    recorder = new MediaRecorder(stream);
    recorder.start(); // Iniciar la grabación

    // Cuando se inicia la grabación, deshabilitar el botón de iniciar y habilitar el botón de detener
    record.disabled = true;
    stop.disabled = false;

    // Obtener el tiempo inicial y crear un intervalo para actualizar el tiempo transcurrido cada segundo
    startTime = Date.now();
    interval = setInterval(updateTimer, 1000);

    // Cuando se recibe un fragmento de audio, enviarlo a la API de Whisper para obtener la transcripción en tiempo real
    recorder.ondataavailable = event => {
      let blob = event.data; // Fragmento de audio en formato blob
      let reader = new FileReader(); // Objeto para leer el blob como un array buffer
      reader.readAsArrayBuffer(blob); // Leer el blob como un array buffer
      reader.onloadend = () => {
        let buffer = reader.result; // Array buffer con los datos del blob
        let data = new Uint8Array(buffer); // Array con los bytes del array buffer

        // Crear un objeto con los parámetros para la solicitud a la API de Whisper
        let params = {
          method: "POST", // Método POST
          headers: {
            "Authorization": `Bearer ${whisperKey}`, // Clave de acceso a la API de Whisper
            "Content-Type": "application/octet-stream" // Tipo de contenido binario
          },
          body: data // Cuerpo con los datos del audio
        };

        // Enviar la solicitud a la API de Whisper y obtener la respuesta como JSON
        fetch("https://api.speechly.com/api/v1/speech/utterance", params)
          .then(response => response.json())
          .then(json => {
            let transcript = json.transcript; // Transcripción del audio en texto
            input.value = transcript; // Asignar el texto al elemento de entrada
          })
          .catch(error => console.error(error)); // Manejar posibles errores
        };
    };

    // Cuando se detiene la grabación, limpiar el intervalo y el objeto MediaRecorder
    recorder.onstop = () => {
      clearInterval(interval);
      recorder = null;
    };
  }).catch(error => console.error(error)); // Manejar posibles errores
}

// Función para detener la grabación
function stop() {
  if (recorder) { // Si hay un objeto MediaRecorder activo
    recorder.stop(); // Detener la grabación

    // Cuando se detiene la grabación, habilitar el botón de iniciar y deshabilitar el botón de detener
    record.disabled = false;
    stop.disabled = true;
  }
}

// Función para actualizar el tiempo transcurrido
function updateTimer() {
  let elapsedTime = Date.now() - startTime; // Tiempo transcurrido en milisegundos
  let minutes = Math.floor(elapsedTime / 60000); // Minutos transcurridos
  let seconds = Math.floor((elapsedTime % 60000) / 1000); // Segundos transcurridos
  minutes = minutes < 10 ? "0" + minutes : minutes; // Añadir un cero si es menor que 10
  seconds = seconds < 10 ? "0" + seconds : seconds; // Añadir un cero si es menor que 10
  timer.textContent = `${minutes}:${seconds}`; // Mostrar el tiempo en el elemento de tiempo
}

// Función para generar el resumen y las etiquetas a partir de la entrada
function generate() {
    let text = input.value; // Obtener el texto de la entrada
    if (text) { // Si el texto no está vacío
      // Crear un objeto con los parámetros para la solicitud al prompt de resumen
      let summaryParams = {
        method: "POST", // Método POST
        headers: {
          "Authorization": `Bearer ${chatgptKey}`, // Clave de acceso a la API de ChatGPT
          "Content-Type": "application/json" // Tipo de contenido JSON
        },
        body: JSON.stringify({ // Cuerpo con el código del prompt y la entrada
          prompt: `// Generar un resumen de 20 palabras a partir de un texto
  ###
  Texto: ${text}
  ###
  Resumen:`,
          max_tokens: 20, // Número máximo de palabras a generar
          stop: "\n" // Carácter para detener la generación
        })
      };
  
      // Enviar la solicitud al prompt de resumen y obtener la respuesta como JSON
      fetch(summaryURL, summaryParams)
        .then(response => response.json())
        .then(json => {
          let summaryText = json.choices[0].text; // Texto generado por el prompt de resumen
          summary.textContent = summaryText; // Asignar el texto al elemento de salida de resumen
        })
        .catch(error => console.error(error)); // Manejar posibles errores
  
      // Crear un objeto con los parámetros para la solicitud al prompt de etiquetas
      let labelsParams = {
        method: "POST",
        headers: {
          "Authorization": `Bearer ${chatgptKey}`,
          "Content-Type": "application/json"
        },
        body: JSON.stringify({ // Cuerpo con el código del prompt y la entrada
          prompt: `// Generar etiquetas de ideas e intenciones a partir de un texto
  ###
  Texto: ${text}
  ###
  Etiquetas:`,
          max_tokens: 20, // Número máximo de palabras a generar
          stop: "\n" // Carácter para detener la generación
        })
      };
  
      // Enviar la solicitud al prompt de etiquetas y obtener la respuesta como JSON
      fetch(labelsURL, labelsParams)
        .then(response => response.json())
        .then(json => {
          let labelsText = json.choices[0].text; // Texto generado por el prompt de etiquetas
          labels.textContent = labelsText; // Asignar el texto al elemento de salida de etiquetas
        })
        .catch(error => console.error(error)); // Manejar posibles errores
  
    } else { // Si el texto está vacío
      alert("Por favor, escribe o graba algo en la entrada."); // Mostrar un mensaje de alerta
    }
  }
  
  // Función para mejorar el resumen y las etiquetas a partir del resumen y las etiquetas generados
  function improve() {
    let summaryText = summary.textContent; // Obtener el texto del resumen generado
    let labelsText = labels.textContent; // Obtener el texto de las etiquetas generadas
  
    if (summaryText && labelsText) { // Si los textos no están vacíos
  
      // Crear un objeto con los parámetros para la solicitud al prompt de resumen mejorado
      let improvedSummaryParams = {
        method: "POST", // Método POST
        headers: {
          "Authorization": `Bearer ${chatgptKey}`, // Clave de acceso a la API de ChatGPT
          "Content-Type": "application/json" // Tipo de contenido JSON
        },
        body: JSON.stringify({ // Cuerpo con el código del prompt y la entrada
          prompt: `// Generar un resumen mejorado de 40 palabras a partir de un resumen y unas etiquetas
  ###
  Resumen: ${summaryText}
  Etiquetas: ${labelsText}
  ###
  Resumen mejorado:`,
          max_tokens: 40, // Número máximo de palabras a generar
          stop: "\n" // Carácter para detener la generación
        })
    };

    // Enviar la solicitud al prompt de resumen mejorado y obtener la respuesta como JSON
    fetch(improvedSummaryURL, improvedSummaryParams)
      .then(response => response.json())
      .then(json => {
        let improvedSummaryText = json.choices[0].text; // Texto generado por el prompt de resumen mejorado
        improvedSummary.textContent = improvedSummaryText; // Asignar el texto al elemento de salida de resumen mejorado
      })
      .catch(error => console.error(error)); // Manejar posibles errores

    // Crear un objeto con los parámetros para la solicitud al prompt de etiquetas mejoradas
    let improvedLabelsParams = {
      method: "POST",
      headers: {
        "Authorization": `Bearer ${chatgptKey}`,
        "Content-Type": "application/json"
      },
      body: JSON.stringify({ // Cuerpo con el código del prompt y la entrada
        prompt: `// Generar etiquetas de ideas e intenciones mejoradas a partir de un resumen mejorado
###
Resumen mejorado: ${improvedSummaryText}
###
Etiquetas mejoradas:`,
        max_tokens: 20, // Número máximo de palabras a generar
        stop: "\n" // Carácter para detener la generación
      })
    };

    // Enviar la solicitud al prompt de etiquetas mejoradas y obtener la respuesta como JSON
    fetch(improvedLabelsURL, improvedLabelsParams)
      .then(response => response.json())
      .then(json => {
        let improvedLabelsText = json.choices[0].text; // Texto generado por el prompt de etiquetas mejoradas
        improvedLabels.textContent = improvedLabelsText; // Asignar el texto al elemento de salida de etiquetas mejoradas
      })
      .catch(error => console.error(error)); // Manejar posibles errores

  } else { // Si los textos están vacíos
    alert("Por favor, genera primero el resumen y las etiquetas."); // Mostrar un mensaje de alerta
  }
}
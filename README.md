<!doctype html>
<html lang="es">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Generador ICFES - Demo (Offline)</title>
<style>
  body{font-family:Inter,system-ui,-apple-system,Segoe UI,Roboto,"Helvetica Neue",Arial;line-height:1.4;margin:0;background:#f5f7fa;color:#122; padding:24px}
  h1{margin:0 0 8px;font-size:22px}
  p.lead{margin:0 0 18px;color:#445}
  .grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(280px,1fr));gap:12px;margin-bottom:16px}
  .card{background:white;border-radius:8px;padding:12px;box-shadow:0 6px 18px rgba(20,30,50,0.06)}
  label{font-weight:600;font-size:13px;display:block;margin-bottom:6px}
  input[type="text"], input[type="number"], select, textarea{width:100%;padding:8px;border:1px solid #d7dde6;border-radius:6px}
  button{background:#0b63d6;color:white;border:none;padding:10px 14px;border-radius:8px;cursor:pointer}
  button.secondary{background:#2d3748}
  .row{display:flex;gap:8px;align-items:center;margin-top:10px}
  pre{background:#0f1724;color:#dbeafe;padding:10px;border-radius:6px;overflow:auto}
  footer{margin-top:18px;color:#567;font-size:13px}
  .small{font-size:13px;color:#556}
</style>
</head>
<body>

<h1>📘 Generador ICFES — Demo (offline)</h1>
<p class="lead">Rellena tema + número por cada área. Esto genera preguntas de ejemplo (plantilla) con competencia y componente. Descarga JSON/CSV para usar en tu demo o como banco de preguntas.</p>

<div class="card">
  <div class="small">El flujo: completa los temas → Generar (local) → revisar → Descargar</div>
</div>

<form id="formAreas">
  <div class="grid" id="areasGrid">
    <!-- bloque de áreas generado por JS -->
  </div>

  <div style="display:flex;gap:8px;margin-top:8px">
    <button id="genBtn" type="button">✨ Generar preguntas (local)</button>
    <button id="downloadJsonBtn" type="button" class="secondary" disabled>📥 Descargar JSON</button>
    <button id="downloadCsvBtn" type="button" class="secondary" disabled>📥 Descargar CSV</button>
    <button id="clearBtn" type="button" class="secondary">🧹 Limpiar</button>
  </div>
</form>

<h3 style="margin-top:18px">Salida (vista previa)</h3>
<div class="card">
  <pre id="output" style="min-height:140px">Aquí verás el JSON generado...</pre>
</div>

<footer>
  <div class="small">Áreas incluidas: Matemáticas, Lectura crítica (Lenguaje), Ciencias Naturales, Ciencias Sociales y Ciudadanas, Inglés. Esta versión usa plantillas locales para demo.</div>
</footer>

<script>
/*
  icfes_generator.html
  - Genera preguntas de ejemplo (offline) para las 5 áreas ICFES.
  - Estructura de salida: [{id, area, enunciado, opciones:{A,B,C,D}, respuesta, justificacion, competencia, componente}, ...]
  - Descarga JSON / CSV
*/

const AREAS = [
  {
    id: "matematicas",
    nombre: "Matemáticas",
    competencias: ["Razonamiento cuantitativo", "Resolución de problemas", "Modelación matemática"],
    componentes: ["Álgebra", "Geometría", "Probabilidad y Estadística", "Funciones"]
  },
  {
    id: "lecturacritica",
    nombre: "Lectura crítica (Lenguaje)",
    competencias: ["Comprensión lectora", "Interpretación de textos", "Inferencia"],
    componentes: ["Comprensión de textos literarios", "Comprensión de textos expositivos", "Interpretación de argumentos"]
  },
  {
    id: "cienciasnaturales",
    nombre: "Ciencias Naturales",
    competencias: ["Comprensión científica", "Razonamiento científico", "Aplicación de conocimientos"],
    componentes: ["Biología", "Física", "Química", "Ciencias de la Tierra"]
  },
  {
    id: "sociales",
    nombre: "Ciencias Sociales y Ciudadanas",
    competencias: ["Pensamiento social", "Competencias ciudadanas", "Interpretación histórica"],
    componentes: ["Historia", "Geografía", "Constitución y democracia", "Economía básica"]
  },
  {
    id: "ingles",
    nombre: "Inglés",
    competencias: ["Comprensión lectora en inglés", "Uso funcional del idioma", "Comprensión auditiva (si aplica)"],
    componentes: ["Reading", "Vocabulary & Grammar", "Functional English"]
  }
];

function createAreaBlocks() {
  const grid = document.getElementById('areasGrid');
  grid.innerHTML = '';
  AREAS.forEach(area => {
    const div = document.createElement('div');
    div.className = 'card';
    div.innerHTML = `
      <label>${area.nombre}</label>
      <input type="text" data-area="${area.id}" placeholder="Tema (p. ej. Revolución Francesa / Probabilidad / Ósmosis)" />
      <label style="margin-top:8px">Número de preguntas</label>
      <input type="number" min="1" max="10" value="2" data-num="${area.id}" />
      <div class="small" style="margin-top:8px;color:#445">Competencias (ejemplos): ${area.competencias.join(' · ')}</div>
      <div class="small" style="color:#667;margin-top:6px">Componentes (ejemplos): ${area.componentes.join(' · ')}</div>
    `;
    grid.appendChild(div);
  });
}

createAreaBlocks();

/* Util: genera un id incremental */
let globalId = 1;

/* Función que crea una pregunta plantilla (offline) */
function generarPreguntaPlantilla(areaObj, tema, indice) {
  // seleccionar competencia y componente de forma cíclica
  const competencia = areaObj.competencias[indice % areaObj.competencias.length];
  const componente = areaObj.componentes[indice % areaObj.componentes.length];

  // Enunciado plantilla (varía según área para parecer más real)
  let enunciado;
  if (areaObj.id === 'matematicas') {
    enunciado = `En relación con ${tema}, resuelva: Si una cantidad aumenta proporcionalmente en un 20% y luego disminuye en 20%, ¿cuál es el % neto aproximado de cambio?`;
  } else if (areaObj.id === 'lecturacritica') {
    enunciado = `Lee el siguiente extracto sobre ${tema}: "…". ¿Cuál es la idea principal implícita en el texto?`;
  } else if (areaObj.id === 'cienciasnaturales') {
    enunciado = `Considerando un experimento relacionado con ${tema}, ¿qué variable es la más apropiada para medir el efecto descrito?`;
  } else if (areaObj.id === 'sociales') {
    enunciado = `Respecto a ${tema}, ¿qué hecho histórico o social permitió el cambio descrito en el enunciado?`;
  } else { // inglés
    enunciado = `Read the following short sentence about ${tema}: "...". Which option best summarizes the sentence?`;
  }

  // Opciones (templated). Marcaremos la opción C como "correcta" por plantilla y crearemos justificación coherente.
  const opciones = {
    A: "Opción A (distractor).",
    B: "Opción B (distractor).",
    C: "Opción C (correcta).",
    D: "Opción D (distractor)."
  };

  const respuesta = "C";
  const justificacion = `La opción C es la correcta porque, en el contexto de ${tema}, representa la interpretación/resultado coherente con la competencia "${competencia}" y el componente "${componente}".`;

  return {
    id: globalId++,
    area: areaObj.nombre,
    enunciado,
    opciones,
    respuesta,
    justificacion,
    competencia,
    componente
  };
}

/* Genera preguntas para todas las áreas según entradas del formulario */
function generarTodasLasPreguntas() {
  const resultados = [];
  AREAS.forEach(area => {
    const temaInput = document.querySelector(`input[data-area="${area.id}"]`);
    const numInput = document.querySelector(`input[data-num="${area.id}"]`);
    const tema = temaInput ? temaInput.value.trim() : '';
    const num = numInput ? parseInt(numInput.value) || 0 : 0;
    if (tema && num > 0) {
      for (let i = 0; i < num; i++) {
        resultados.push(generarPreguntaPlantilla(area, tema, i));
      }
    }
  });
  return resultados;
}

/* Descargas */
function downloadObjectAsJson(exportObj, exportName){
  const dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(exportObj, null, 2));
  const dlAnchorElem = document.createElement('a');
  dlAnchorElem.setAttribute("href", dataStr);
  dlAnchorElem.setAttribute("download", exportName + ".json");
  dlAnchorElem.click();
}

function convertToCSV(objArray) {
  const array = typeof objArray !== 'object' ? JSON.parse(objArray) : objArray;
  let str = '';
  // encabezados
  const headers = ["id","area","enunciado","opciones_A","opciones_B","opciones_C","opciones_D","respuesta","justificacion","competencia","componente"];
  str += headers.join(',') + '\r\n';

  for (let i = 0; i < array.length; i++) {
    const row = array[i];
    const line = [
      row.id,
      `"${row.area.replace(/"/g,'""')}"`,
      `"${(row.enunciado||'').replace(/"/g,'""')}"`,
      `"${(row.opciones?.A||'').replace(/"/g,'""')}"`,
      `"${(row.opciones?.B||'').replace(/"/g,'""')}"`,
      `"${(row.opciones?.C||'').replace(/"/g,'""')}"`,
      `"${(row.opciones?.D||'').replace(/"/g,'""')}"`,
      row.respuesta,
      `"${(row.justificacion||'').replace(/"/g,'""')}"`,
      `"${(row.competencia||'').replace(/"/g,'""')}"`,
      `"${(row.componente||'').replace(/"/g,'""')}"`
    ];
    str += line.join(',') + '\r\n';
  }
  return str;
}

document.getElementById('genBtn').addEventListener('click', () => {
  globalId = 1; // reiniciar ids para cada generación (opcional)
  const datos = generarTodasLasPreguntas();
  const output = document.getElementById('output');
  if (datos.length === 0) {
    output.textContent = 'No se generaron preguntas. Completa al menos un tema y número > 0.';
    document.getElementById('downloadJsonBtn').disabled = true;
    document.getElementById('downloadCsvBtn').disabled = true;
    return;
  }
  output.textContent = JSON.stringify(datos, null, 2);
  // habilitar descargas
  document.getElementById('downloadJsonBtn').disabled = false;
  document.getElementById('downloadCsvBtn').disabled = false;

  // Almacenamos en window para que los botones de descarga lo usen
  window._icfes_last_generated = datos;
});

document.getElementById('downloadJsonBtn').addEventListener('click', () => {
  const datos = window._icfes_last_generated || [];
  if (datos.length === 0) return alert('No hay datos para descargar. Genera primero.');
  const now = new Date().toISOString().slice(0,19).replace(/[:T]/g,'-');
  downloadObjectAsJson(datos, `preguntas_icfes_${now}`);
});

document.getElementById('downloadCsvBtn').addEventListener('click', () => {
  const datos = window._icfes_last_generated || [];
  if (datos.length === 0) return alert('No hay datos para descargar. Genera primero.');
  const csv = convertToCSV(datos);
  const dataStr = "data:text/csv;charset=utf-8," + encodeURIComponent(csv);
  const dlAnchor = document.createElement('a');
  dlAnchor.setAttribute("href", dataStr);
  const now = new Date().toISOString().slice(0,19).replace(/[:T]/g,'-');
  dlAnchor.setAttribute("download", `preguntas_icfes_${now}.csv`);
  dlAnchor.click();
});

document.getElementById('clearBtn').addEventListener('click', () => {
  // limpiar inputs
  AREAS.forEach(area => {
    const temaInput = document.querySelector(`input[data-area="${area.id}"]`);
    const numInput = document.querySelector(`input[data-num="${area.id}"]`);
    if (temaInput) temaInput.value = '';
    if (numInput) numInput.value = 2;
  });
  document.getElementById('output').textContent = 'Aquí verás el JSON generado...';
  document.getElementById('downloadJsonBtn').disabled = true;
  document.getElementById('downloadCsvBtn').disabled = true;
  window._icfes_last_generated = [];
});
</script>

</body>
</html>

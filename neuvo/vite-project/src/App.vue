<script setup>
import axios from "axios";
import { ref, onMounted } from "vue";
import { defineProps } from "vue";

// Modelo de Gemini
const MODEL = "gemini-2.0-flash";

// Input donde el usuario escribe la excusa
const excusaBase = ref("");
const cargando = ref("");
// Array de excusas generadas
const excusas = ref([]);
const guardarEnLocalStorage1 = () => {
  localStorage.setItem("excusas", JSON.stringify(excusas.value));
};
const mostrarOne = ref(true)
onMounted(() => {
  const data = localStorage.getItem("mostrarInfo")

  if (data === "false") {
    mostrarOne.value = false
  }
})
const borrarDiv = (index) => {
   excusas.value.splice(index, 1);
     guardarEnLocalStorage1();
  mostrarOne.value = false 
 

}


// Cargar datos del localStorage al montar el componente
onMounted(() => {
  const savedExcusas = localStorage.getItem("excusas");
  if (savedExcusas) {
    excusas.value = JSON.parse(savedExcusas);
    
    // Reiniciar contadores para cada excusa
    excusas.value.forEach(excusa => {
      if (excusa.inicio) {
        // Calcular el tiempo transcurrido desde el inicio guardado
        const tiempoTranscurrido = Math.floor((Date.now() - excusa.inicio) / 1000);
        excusa.contador = tiempoTranscurrido;
        
        // Calcular el nivel basado en el tiempo transcurrido
        excusa.nivel = Math.min(Math.floor(tiempoTranscurrido / 30) + 1, 4);
        
        // Iniciar el contador nuevamente
        iniciarContador(excusa);
      }
    });
  }
});

// Guardar excusas en localStorage
const guardarEnLocalStorage = () => {
  localStorage.setItem("excusas", JSON.stringify(excusas.value));
};



const generarExcusa = async () => {
  cargando.value = true;

  // Esperar 2 segundos ANTES de generar la excusa
  await new Promise(resolve => setTimeout(resolve, 2000));

  try {
    const keys = [import.meta.env.VITE_API_KEY];

    for (const key of keys) {
      try {
        const contenido = [
          {
            role: "user",
            parts: [
              {
                text: `Eres un generador experto de excusas cómicas, absurdas...
                Quiero que construyas una excusa inventada explicando por qué llegué tarde debido a: **${excusaBase.value}**.`
              }
            ]
          }
        ];

        const res = await axios.post(
          `https://generativelanguage.googleapis.com/v1beta/models/${MODEL}:generateContent?key=${key}`,
          { contents: contenido },
          { headers: { "Content-Type": "application/json" } }
        );

        const texto =
          res.data.candidates?.[0]?.content?.parts?.[0]?.text || "";

        excusas.value.push({
          texto,
          contador: 0,
          inicio: Date.now(),
          nivel: 0
        });

        console.log("🎭 Excusa generada:", texto);

        guardarEnLocalStorage();
        break; // se sale después de funcionar con una key
      } catch (error) {
        console.warn("⚠️ Falló esta API key, probando la siguiente...");
      }
    }

  } catch (err) {
    console.error("❌ Error generando excusa:", err);
  }

  cargando.value = false;
};



const iniciarContador = (excusa) => {
  // Guardar el tiempo de inicio
  excusa.inicio = Date.now();
  
  // Guardar en localStorage al iniciar
  guardarEnLocalStorage();

  const intervalo = setInterval(() => {
    // Calcular los segundos transcurridos
    excusa.contador = Math.floor((Date.now() - excusa.inicio) / 1000);

    console.log(`Segundos transcurridos: ${excusa.contador}`);

    // Cada 30 segundos, cambiar imagen
    if (excusa.contador > 0 && excusa.contador % 30 === 0) {
      console.log("🔥 30 segundos completados, cambiar imagen");

      // Aquí puedes poner la lógica de cambiar la imagen
      // Por ejemplo, aumentar un índice de imagen dentro del excusa
      if (!excusa.nivel) excusa.nivel = 0;
      excusa.nivel = Math.min(excusa.nivel + 1, 4); // 4 imágenes: junior, oficial, master, jefe
      
      // Guardar en localStorage al cambiar nivel
      guardarEnLocalStorage();

      // Reiniciar el contador para volver a contar otros 30 segundos
      excusa.inicio = Date.now();
      excusa.contador = 0;
    }
  }, 1000);
  
  // Guardar el intervalo para poder limpiarlo si es necesario
  excusa.intervaloId = intervalo;
};

const props = defineProps({
  color: {  
    type: String,
    default: "red"
  },
  textColor: {
    type: String,
    default: "yellow"
  }
});
</script>

<template>
  <div class="app-container">
    <div class="division">
      <div  class="one">
        <h2 class="title">BIENVENIDO A TU POSTERGACIÓN DE TAREAS</h2>

        <!-- Input -->
        <input
          v-model="excusaBase"
          placeholder="Ej: Perdí mi tarea, el gato se la comió."
          class="excusa-input"
          @keyup.enter="generarExcusa"
        />

        <!-- Botón generar -->
        <button @click="generarExcusa" class="generate-btn" :style="{ backgroundColor: props.color, color: props.textColor }">
          Generar Excusa
        </button>
  <p v-if="cargando" > cargando excusa espero unos segundos</p>

  <div class="info" v-if="mostrarOne">
      <!-- Mostrar excusas -->
        <div   v-for="(excusa, index) in excusas" :key="index" class="excusa-card">
          <p  class="excusa-text"><strong>Excusa {{ index + 1 }}:</strong> {{ excusa.texto }}</p>
       
<q-btn 
  @click="borrarDiv"
  label="Eliminar bienvenida"
/>

          <div class="excusa-info">
            <p class="timer">Tiempo postergando: {{ excusa.contador }} segundos</p>
            <q-btn 
              @click="iniciarContador(excusa)" 
              :style="{ backgroundColor: props.color, color: props.textColor }" 
              class="timer-btn"
              label="Iniciar Postergación" 
            />

          
            
          </div>
          
          <div class="level-indicator" v-if="excusa.nivel > 0">
            <img
              v-if="excusa.nivel === 1"
              src="./junior.png"
              alt="Junior"
              class="level-img"
            />
            <img
              v-if="excusa.nivel === 2"
              src="./oficial.png"
              alt="Oficial"
              class="level-img"
            />
            <img
              v-if="excusa.nivel === 3"
              src="./master.png"
              alt="Master"
              class="level-img"
            />
            <img
              v-if="excusa.nivel === 4"
              src="./jefe.png"
              alt="Jefe"
              class="level-img"
            />
            
            <p class="level-up" v-if="excusa.contador >= 30 && excusa.contador % 30 === 0">
              ¡Felicidades, has subido de nivel!
            </p>
          </div>
        </div>
      </div>
  </div>
  



      <div class="logros">
        <h2 class="logros-title">TUS LOGROS DE POSTERGACIÓN</h2>
        <div class="img-container">
          <div class="content">
            <img src="./junior.png" alt="Postergador Junior" class="achievement-img">
            <h2>Postergador Junior</h2>
            <p>30 segundos de postergación</p>
          </div>
          <div class="content">
            <img src="./oficial.png" alt="Postergador Oficial" class="achievement-img">
            <h2>Postergador Oficial</h2>
            <p>1 minuto de postergación</p>
          </div>
          <div class="content">
            <img src="./master.png" alt="Postergador Master" class="achievement-img">
            <h2>Postergador Master</h2>
            <p>1.5 minutos de postergación</p>
          </div>
          <div class="content">
            <img src="./jefe.png" alt="Postergador Jefe" class="achievement-img">
            <h2>Postergador Jefe</h2>
            <p>2 minutos de postergación</p>

          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.app-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background-color: #f5f7fa;
  min-height: 100vh;
}

.division {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 30px;
}

.title {
  text-align: center;
  color: #2c3e50;
  margin-bottom: 25px;
  font-size: 1.8rem;
  text-shadow: 1px 1px 2px rgba(0,0,0,0.1);
}

.excusa-input {
  width: 100%;
  padding: 12px 15px;
  border: 2px solid #ddd;
  border-radius: 8px;
  margin-bottom: 15px;
  font-size: 1rem;
  transition: border-color 0.3s;
}

.excusa-input:focus {
  border-color: #3498db;
  outline: none;
  box-shadow: 0 0 0 2px rgba(52, 152, 219, 0.2);
}

.generate-btn {
  width: 100%;
  padding: 12px;
  border: none;
  border-radius: 8px;
  font-size: 1rem;
  font-weight: bold;
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
  margin-bottom: 25px;
}

.generate-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0,0,0,0.1);
}

.excusa-card {
  background-color: white;
  border-radius: 10px;
  padding: 20px;
  margin-bottom: 20px;
  box-shadow: 0 4px 6px rgba(0,0,0,0.05);
  transition: transform 0.2s, box-shadow 0.2s;
  border-left: 4px solid #3498db;
}

.excusa-card:hover {
  transform: translateY(-3px);
  box-shadow: 0 6px 12px rgba(0,0,0,0.1);
}

.excusa-text {
  margin-bottom: 15px;
  line-height: 1.5;
  color: #34495e;
}

.excusa-info {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
}

.timer {
  font-weight: bold;
  color: #e74c3c;
}

.timer-btn {
  padding: 8px 15px;
  border: none;
  border-radius: 5px;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.2s;
}

.timer-btn:hover {
  opacity: 0.9;
  transform: scale(1.05);
}

.level-indicator {
  text-align: center;
  margin-top: 15px;
  padding-top: 15px;
  border-top: 1px dashed #ddd;
}

.level-img {
  height: auto;
  width: 120px;
  margin-bottom: 10px;
  border-radius: 10px;
  box-shadow: 0 3px 6px rgba(0,0,0,0.1);
}

.level-up {
  color: #27ae60;
  font-weight: bold;
  margin-top: 10px;
  animation: pulse 1.5s infinite;
}

@keyframes pulse {
  0% { transform: scale(1); }
  50% { transform: scale(1.05); }
  100% { transform: scale(1); }
}

.logros {
  background-color: white;
  border-radius: 10px;
  padding: 25px;
  box-shadow: 0 4px 6px rgba(0,0,0,0.05);
  height: fit-content;
}

.logros-title {
  text-align: center;
  color: #2c3e50;
  margin-bottom: 25px;
  font-size: 1.5rem;
}

.img-container {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.content {
  display: flex;
  align-items: center;
  gap: 15px;
  padding: 15px;
  border-radius: 8px;
  transition: background-color 0.2s;
}

.content:hover {
  background-color: #f8f9fa;
}

.achievement-img {
  height: auto;
  width: 80px;
  border-radius: 8px;
}

.content h2 {
  margin: 0;
  font-size: 1.1rem;
  color: #2c3e50;
}

.content p {
  margin: 0;
  color: #7f8c8d;
  font-size: 0.9rem;
}

/* Responsive */
@media (max-width: 768px) {
  .division {
    grid-template-columns: 1fr;
  }
  
  .excusa-info {
    flex-direction: column;
    gap: 10px;
    align-items: flex-start;
  }
  
  .content {
    flex-direction: column;
    text-align: center;
  }
}
</style>
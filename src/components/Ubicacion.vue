<script setup lang="ts">
import { ref, onMounted } from 'vue';
import axios from 'axios';

const empresas = ref(null);
const loading = ref(true);
const error = ref(null);

const fetchUbicacion = async () => {
  try {
    const response = await axios.get('/empresas/1');
    //console.log('Respuesta del servidor:', response);
    empresas.value = response.data.empresa || {};
    loading.value = false;
  } catch (err) {
    console.error('Error al obtener Ubicación:', err);
    error.value =
      err.response && err.response.status
        ? `Error ${err.response.status}: ${err.response.statusText}`
        : 'No se pudo conectar al servidor.';
    loading.value = false;
  }
};

onMounted(() => {
  fetchUbicacion();
});
</script>

<template>
  <section id="ubicacion" class="py-5 bg-pattern">
    <div class="container mb-5">
      <h2 class="text-center mb-5 section-title">
        Nuestra Ubicación
      </h2>
      <div v-if="loading" class="text-center loading-spinner">
        <div class="spinner"></div>
        <p>Cargando...</p>
      </div>
      <div v-else-if="error" class="text-center text-danger error-message">
        <i class="fas fa-exclamation-circle"></i>
        <p>{{ error }}</p>
      </div>
      <div v-else class="row align-items-center content-wrapper">
        <div class="col-md-6 map-container">
          <div class="ratio ratio-16x9 map-frame">
            <iframe src="https://www.google.com/maps/embed?pb=!1m18!1m12!1m3!1d782.2618851872418!2d-71.4177988!3d-34.517429899999996!2m3!1f0!2f0!3f0!3m2!1i1024!2i768!4f13.1!3m3!1m2!1s0x96647747e7d6bd35%3A0xd5e8cbad3c392bbe!2sConstructora%20Jara!5e1!3m2!1ses-419!2scl!4v1737598953319!5m2!1ses-419!2scl" width="600" height="450" style="border:0;" allow="fullscreen" loading="lazy" referrerpolicy="no-referrer-when-downgrade"></iframe>
          </div>
        </div>
        <div class="col-md-6 info-container">
          <div class="info-card">
            <h3 class="mb-4">
              Visítanos en nuestro sitio de exhibición
            </h3>
            <p v-if="empresas && empresas.ubicacion_empresa" class="info-item">
              <i class="fas fa-map-marker-alt text-primary me-2"></i>
              {{ empresas.ubicacion_empresa }}
            </p>
            <p v-else class="info-item loading">
              <i class="fas fa-spinner fa-spin text-primary"></i> Cargando ubicación...
            </p>
            <p v-if="empresas && empresas.celular_empresa" class="info-item">
              <i class="fas fa-phone-alt text-primary me-2"></i>{{ empresas.celular_empresa }}
            </p>
            <p v-if="empresas && empresas.email_empresa" class="info-item">
              <i class="fas fa-envelope text-primary me-2"></i>{{ empresas.email_empresa }}
            </p>
            <p>
              De Lunes a Viernes de 09:00 a 18:00 hrs.
            </p>
            <p>
              Sábados de 09:00 a 14:00 hrs.
            </p>
            <p>
              Domingos y feriados Cerrado
            </p>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<style scoped>
/* :root {
  --primary-color: #f7b500;
  --secondary-color: #1e1f21;
  --accent-color: #ff6b35;
  --text-color: #333333;
  --light-bg: #f8f9fa;
  --dark-bg: #2c2d30;
} */

.bg-pattern {
  background: linear-gradient(135deg, var(--secondary-color), var(--dark-bg));
  color: var(--light-bg);
}

.section-title {
  font-size: 2.5rem;
  font-weight: 700;
  color: var(--primary-color);
  margin-bottom: 2rem;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.2);
}

.content-wrapper {
  background-color: rgba(255, 255, 255, 0.05);
  border-radius: 15px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
  overflow: hidden;
}

.map-container {
  padding: 20px;
}

.map-frame {
  border-radius: 10px;
  overflow: hidden;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.2);
}

.info-container {
  padding: 20px;
}

.info-card {
  background-color: rgba(255, 255, 255, 0.1);
  border-radius: 10px;
  padding: 20px;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
}

.info-item {
  margin-bottom: 15px;
  font-size: 1.1rem;
  color: var(--light-bg);
  transition: all 0.3s ease;
}

.info-item:hover {
  transform: translateX(5px);
  color: var(--primary-color);
}

.info-item i {
  width: 25px;
  text-align: center;
  color: var(--primary-color);
}

.loading-spinner {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 200px;
}

.spinner {
  border: 4px solid rgba(247, 181, 0, 0.1);
  border-left-color: var(--primary-color);
  border-radius: 50%;
  width: 40px;
  height: 40px;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.error-message {
  background-color: rgba(255, 107, 53, 0.2);
  border: 1px solid var(--accent-color);
  border-radius: 5px;
  padding: 20px;
  margin-top: 20px;
}

.error-message i {
  font-size: 2rem;
  color: var(--accent-color);
  margin-bottom: 10px;
}

.text-primary {
  color: var(--primary-color) !important;
}

.text-danger {
  color: var(--accent-color) !important;
}

h3 {
  color: var(--primary-color);
}
</style>
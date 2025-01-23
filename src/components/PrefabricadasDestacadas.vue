<script setup>
import { ref, computed, onMounted, onUnmounted } from "vue";
import axios from "axios";
import { useRouter } from "vue-router";

const prefabricadas = ref([]);
const featuredPrefabricadas = ref([]);
const loading = ref(true);
const error = ref(null);
const progressValue = ref(0);

const baseURL = "https://v1-backend-sebastian-nadal-production.up.railway.app";
const router = useRouter();

const formatPrice = (price) => {
  return price.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ".");
};

const getPrefabricadas = async () => {
  try {
    loading.value = true;
    const { data: prefabricadasData } = await axios.get(`${baseURL}/empresas/1/prefabricadas`);
    
    if (prefabricadasData?.prefabricadas) {
      prefabricadas.value = prefabricadasData.prefabricadas
        .filter(prefab => prefab.destacada)
        .map((prefab) => ({
          ...prefab,
          id: prefab?.id,
          nombre_prefabricada: prefab?.nombre_prefabricada || 'Casa sin nombre',
          m2: prefab?.m2 || 'N/A',
          garantia: prefab?.garantia || 'No especificada',
          image: prefab?.imagenes_prefabricadas?.[0]?.image || '/img/placeholder.png',
          precios: Array.isArray(prefab?.precios) ? prefab.precios.map(precio => ({
            id: precio?.id,
            nombre_precio: precio?.nombre_precio || 'Precio base',
            valor_prefabricada: precio?.valor_prefabricada || 0
          })) : []
        }));
      updateFeaturedPrefabricadas();
    }
  } catch (err) {
    error.value = "Error al cargar las casas prefabricadas";
    console.error("Error al cargar las prefabricadas:", err);
  } finally {
    loading.value = false;
  }
};

const updateFeaturedPrefabricadas = () => {
  const shuffled = [...prefabricadas.value].sort(() => 0.5 - Math.random());
  featuredPrefabricadas.value = shuffled.slice(0, 3);
};

const verTodasPrefabricadas = () => {
  router.push({ name: 'prefabricadas' });    
};

const navigateToDetailPrefabricada = (id) => {
  if (id) {
    router.push({ name: "prefabricada-detalle", params: { id } });
  }
};

let intervalId;
let progressIntervalId;

const startIntervals = () => {
  intervalId = setInterval(() => {
    updateFeaturedPrefabricadas();
    progressValue.value = 0;
  }, 10000);

  progressIntervalId = setInterval(() => {
    progressValue.value += 1;
  }, 100);
};

onMounted(async () => {
  await getPrefabricadas();
  startIntervals();
});

onUnmounted(() => {
  clearInterval(intervalId);
  clearInterval(progressIntervalId);
});

const progressPercentage = computed(() => {
  return (progressValue.value / 100) * 100;
});
</script>

<template>
  <div class="animated-gradient py-5">
    <header class="text-white text-center py-5 animate__animated animate__fadeIn">
      <div class="container">
        <h1 class="display-4 fw-bold mb-3">Casas Prefabricadas Destacadas</h1>
        <p class="lead mb-4">Descubre nuestras mejores opciones en casas prefabricadas</p>
      </div>
    </header>
    
    <!-- Loading State -->
    <div v-if="loading" class="text-center">
      <i class="fas fa-circle-notch fa-spin fa-3x text-white"></i>
      <p class="mt-3 text-white">Cargando casas prefabricadas destacadas...</p>
    </div>

    <!-- Error State -->
    <div v-else-if="error" class="container">
      <div class="alert alert-danger text-center" role="alert">
        <i class="fas fa-exclamation-triangle me-2"></i>
        {{ error }}
      </div>
    </div>

    <!-- Empty State -->
    <div v-else-if="!featuredPrefabricadas.length" class="text-center text-white">
      <i class="fas fa-home fa-3x mb-3"></i>
      <p>No hay casas prefabricadas destacadas disponibles en este momento.</p>
    </div>

    <!-- Content -->
    <div v-else class="container">
      <div class="row justify-content-center g-4">
        <div
          v-for="prefabricada in featuredPrefabricadas"
          :key="prefabricada?.id"
          class="col-12 col-md-6 col-lg-4 d-flex justify-content-center"
        >
          <div
            class="card h-100 shadow-lg hover-card"
            @click="navigateToDetailPrefabricada(prefabricada?.id)"
          >
            <!-- Listones destacados y oferta -->
            <div class="ribbons">
              <span class="ribbon ribbon-dark text-light">Destacada</span>
              <span v-if="prefabricada?.oferta" class="ribbon ribbon-red">Oferta</span>
            </div>

            <div class="position-relative">
              <img
                :src="prefabricada?.image"
                class="card-img-top"
                :alt="prefabricada?.nombre_prefabricada"
              />
              <div class="card-img-overlay d-flex align-items-end">
                <span v-if="prefabricada?.m2" class="badge bg-primary mb-2">
                  {{ prefabricada.m2 }} m²
                </span>
              </div>
            </div>

            <div class="card-body">
              <h5 class="card-title text-dark">
                {{ prefabricada?.nombre_prefabricada }}
              </h5>
              <p class="card-text">
                <i class="fas fa-shield-alt me-2"></i>
                Garantía: {{ prefabricada?.garantia }}
              </p>
              
              <template v-if="prefabricada?.precios?.length">
                <ul class="list-group list-group-flush">
                  <li
                    v-for="precio in prefabricada.precios"
                    :key="precio?.id"
                    class="list-group-item d-flex justify-content-between align-items-center"
                  >
                    {{ precio?.nombre_precio }}
                    <span class="badge bg-secondary rounded-pill">
                      ${{ formatPrice(precio?.valor_prefabricada) }}
                    </span>
                  </li>
                </ul>
              </template>
            </div>
          </div>
        </div>
      </div>

      <div class="text-center mt-4">
        <button @click="verTodasPrefabricadas" class="btn btn-modern btn-lg">
          Ver Catálogo Completo
        </button>
      </div>

      <!-- Progress Bar -->
      <div class="progress mt-4">
        <div 
          class="progress-bar" 
          role="progressbar" 
          :style="{ width: progressPercentage + '%' }" 
          :aria-valuenow="progressValue" 
          aria-valuemin="0" 
          aria-valuemax="100"
        ></div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.animated-gradient {
  background: linear-gradient(
    -45deg,
    #003f5c, /* Azul oscuro profundo */
    #16222a, /* Azul verdoso oscuro */
    #2f4b7c, /* Azul grisáceo */
    #1e1f21, /* Negro carbón */
    #2a3d45 /* Verde oliva oscuro */
  );
  background-size: 400% 400%;
  animation: gradient 10s ease infinite;
  min-height: 100vh;
}

@keyframes gradient {
  0% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
  100% { background-position: 0% 50%; }
}

.card {
  transition: all 0.3s ease;
  overflow: hidden;
  width: 100%;
  max-width: 30rem;
  background-color: rgba(255, 255, 255, 0.95);
  border: none;
}

.hover-card {
  cursor: pointer;
}

.hover-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 10px 20px rgba(0, 0, 0, 0.2) !important;
}

.card-img-top {
  height: 250px;
  object-fit: cover;
  transition: transform 0.5s ease;
}

.hover-card:hover .card-img-top {
  transform: scale(1.1);
}

.btn-modern {
  background-color: var(--primary-color);
  border: none;
  color: var(--text-color);
  font-weight: 600;
  font-size: 1rem;
  padding: 0.75rem 2rem;
  border-radius: 8px;
  transition: all 0.4s ease;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  letter-spacing: 0.5px;
  cursor: pointer;
}

.btn-modern:hover {
  background-color: var(--secondary-color);
  color: var(--primary-color);
  transform: scale(1.05);
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.3);
}

.btn-modern:active {
  transform: scale(0.98);
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.2);
}

.badge {
  background-color: var(--accent-color);
  color: var(--text-color);
  font-weight: bold;
  font-size: 0.85rem;
  padding: 0.4em 0.8em;
  border-radius: 20px;
}

.ribbons {
  position: absolute;
  top: 10px;
  left: 10px;
  display: flex;
  flex-direction: column;
  gap: 5px;
  z-index: 10;
}

.ribbon {
  font-size: 0.85rem;
  padding: 0.3rem 1.2rem;
  background-color: var(--primary-color);
  color: var(--text-color);
  font-weight: bold;
  text-transform: uppercase;
  border-radius: 0 8px 8px 0;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.25);
}

.ribbon-dark {
  background-color:blueviolet;
}

.ribbon-red {
  background-color: red;
}

.progress {
  height: 6px;
  background-color: rgba(255, 255, 255, 0.5);
  border-radius: 3px;
  overflow: hidden;
}

.progress-bar {
  background-color: var(--primary-color);
  transition: width 0.3s linear;
}
</style>
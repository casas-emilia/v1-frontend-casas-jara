<script setup>
import { ref, onMounted, computed } from "vue";
import { useRoute, useRouter } from "vue-router";
import axios from "axios";
import EjecutivosVentas from "@/components/EjecutivosVentas.vue";


const prefabricada = ref(null);
const loading = ref(true);
const error = ref(null);
const lightboxOpen = ref(false);
const currentImageIndex = ref(0);
const currentPlanoIndex = ref(0);
const isPlanoLightbox = ref(false); 
const touchStartX = ref(0);
const touchEndX = ref(0);

const route = useRoute();
const router = useRouter();
const baseURL = "https://v1-backend-sebastian-nadal-production.up.railway.app/empresas/1";

const regularImages = computed(() => 
  prefabricada.value?.imagenes_prefabricadas?.filter(img => !img.plano) || []
);

const planoImages = computed(() => 
  prefabricada.value?.imagenes_prefabricadas?.filter(img => img.plano) || []
);

const hasRegularImages = computed(() => regularImages.value.length > 0);
const hasPlanoImages = computed(() => planoImages.value.length > 0);

const currentRegularImage = computed(() => 
  hasRegularImages.value ? regularImages.value[currentImageIndex.value].image : null
);

const currentPlanoImage = computed(() => 
  hasPlanoImages.value ? planoImages.value[currentPlanoIndex.value].image : null
);

const handleImageError = (event) => {
  event.target.src = '/img/placeholder.png';
};

const openAccordions = ref(new Set());

const toggleAccordion = (id) => {
  if (openAccordions.value.has(id)) {
    openAccordions.value.delete(id);
  } else {
    openAccordions.value.add(id);
  }
};

const isAccordionOpen = (id) => openAccordions.value.has(id);

onMounted(async () => {
  const prefabricadaId = route.params.id;
  try {
    loading.value = true;
    const { data } = await axios.get(`${baseURL}/prefabricadas/${prefabricadaId}`);
    
    if (data?.prefabricada) {
      prefabricada.value = data.prefabricada;
    } else {
      throw new Error('No se encontró la casa prefabricada');
    }
  } catch (err) {
    error.value = "No se pudo cargar la información de la casa prefabricada.";
    console.error('Error:', err);
  } finally {
    loading.value = false;
  }
});

const navigateCarousel = (direction, isPlano = false) => {
  const images = isPlano ? planoImages.value : regularImages.value;
  const currentIndex = isPlano ? currentPlanoIndex : currentImageIndex;
  
  if (images.length === 0) return;
  
  const length = images.length;
  if (direction === 'next') {
    currentIndex.value = (currentIndex.value + 1) % length;
  } else {
    currentIndex.value = (currentIndex.value - 1 + length) % length;
  }
};

const prevImage = () => navigateCarousel('prev');
const nextImage = () => navigateCarousel('next');
const prevPlano = () => navigateCarousel('prev', true);
const nextPlano = () => navigateCarousel('next', true);

const handleTouchStart = (event) => {
  touchStartX.value = event.touches[0].clientX;
};

const handleTouchEnd = (event, isPlano = false) => {
  touchEndX.value = event.changedTouches[0].clientX;
  const swipeDistance = touchEndX.value - touchStartX.value;
  
  if (Math.abs(swipeDistance) > 50) {
    if (isPlano) {
      swipeDistance > 0 ? prevPlano() : nextPlano();
    } else {
      swipeDistance > 0 ? prevImage() : nextImage();
    }
  }
};

const openLightbox = (index, isPlano = false) => {
  if (isPlano && hasPlanoImages.value) {
    currentPlanoIndex.value = index;
    isPlanoLightbox.value = true;
    lightboxOpen.value = true;
  } else if (!isPlano && hasRegularImages.value) {
    currentImageIndex.value = index;
    isPlanoLightbox.value = false;
    lightboxOpen.value = true;
  }
  document.body.style.overflow = 'hidden';
};

const closeLightbox = () => {
  lightboxOpen.value = false;
  isPlanoLightbox.value = false;
  document.body.style.overflow = '';
};

const lightboxNext = () => {
  isPlanoLightbox.value ? nextPlano() : nextImage();
};

const lightboxPrev = () => {
  isPlanoLightbox.value ? prevPlano() : prevImage();
};

const getFeatureIcon = (clave) => {
  const iconMap = {
    'Dormitorios': 'fas fa-bed',
    'dormitorios': 'fas fa-bed',
    'Baños': 'fas fa-bath',
    'baños': 'fas fa-bath',
    'Cocina': 'fas fa-utensils',
    'cocina': 'fas fa-utensils',
    'Sala': 'fas fa-couch',
    'sala': 'fas fa-couch',
    'Living': 'fas fa-couch',
    'Living-Comedor': 'fas fa-couch',
    'living': 'fas fa-couch',
    'Sala de estar': 'fas fa-couch',
    'sala de estar': 'fas fa-couch',
    'Comedor': 'fas fa-chair',
    'comedor': 'fas fa-chair',
    'Eficiencia energética': 'fas fa-leaf',
    'Materiales': 'fas fa-hard-hat',
    'materiales': 'fas fa-hard-hat',
    'Material': 'fas fa-hard-hat',
    'material': 'fas fa-hard-hat',
    'Diseño': 'fas fa-pencil-ruler',
    'Bodega' : 'fas fa-warehouse',
    'bodega' : 'fas fa-warehouse'
  };
  return iconMap[clave] || 'fas fa-check';
};

const goBack = () => {
    router.go(-1);
};
</script>

<template>
  <div class="prefabricada-details">
    <header class="detail-header">
      <div class="container">
        <nav class="breadcrumb-nav">
          <button class="back-button" @click="goBack">
            <span class="back-icon">←</span>
            <span>Volver</span>
          </button>
        </nav>
      </div>
    </header>

    <div v-if="loading" class="text-center py-5">
      <i class="fas fa-circle-notch fa-spin fa-3x"></i>
      <p class="mt-3">Cargando detalles...</p>
    </div>

    <div v-else-if="error" class="alert alert-danger m-4">
      <i class="fas fa-exclamation-triangle me-2"></i>
      {{ error }}
    </div>
    
    <div v-else-if="prefabricada" class="animate__animated animate__fadeIn">
      <section class="hero-section py-5 text-dark">
        <div class="container">
          <div class="row align-items-center">
            <div class="col-lg-6">
              <h1 class="display-4 fw-bold mb-3">
                {{ prefabricada.nombre_prefabricada }}
              </h1>
              <p class="lead mb-4 animate__animated animate__fadeInLeft animate__delay-1s">
                {{ prefabricada.eslogan }}
              </p>
              <div class="d-flex align-items-center mb-4">
                <i class="fas fa-ruler-combined fa-2x me-3"></i>
                <div>
                  <h5 class="mb-0">Espacio</h5>
                  <p class="mb-0">{{ prefabricada.m2 }} m2</p>
                </div>
              </div>
              <div class="d-flex align-items-center mb-4">
                <i class="fas fa-shield-alt fa-2x me-3"></i>
                <div>
                  <h5 class="mb-0">Garantía</h5>
                  <p class="mb-0">{{ prefabricada.garantia }}</p>
                </div>
              </div>
              <a v-if="prefabricada.precios?.length" 
                 href="#precios" 
                 class="btn btn-primary btn-lg">
                Ver Opciones de Precios
              </a>
            </div>

            <div v-if="hasRegularImages" class="col-lg-6">
              <div id="imageCarousel" 
                   class="carousel slide shadow-lg rounded" 
                   @touchstart="handleTouchStart"
                   @touchend="handleTouchEnd">
                <div class="carousel-indicators">
                  <button v-for="(_, index) in regularImages"
                          :key="index"
                          type="button"
                          :class="{ active: index === currentImageIndex }"
                          :aria-current="index === currentImageIndex"
                          @click="currentImageIndex = index">
                  </button>
                </div>

                <div class="carousel-inner">
                  <div v-for="(imagen, index) in regularImages" 
                       :key="index"
                       class="carousel-item"
                       :class="{ active: currentImageIndex === index }">
                    <img :src="imagen.image" 
                         :alt="`Vista ${index + 1}`"
                         class="d-block w-100"
                         @error="handleImageError"
                         @click="openLightbox(index)">
                  </div>
                </div>

                <button class="carousel-control-prev" @click.prevent="prevImage">
                  <span class="carousel-control-prev-icon"></span>
                  <span class="visually-hidden">Anterior</span>
                </button>
                <button class="carousel-control-next" @click.prevent="nextImage">
                  <span class="carousel-control-next-icon"></span>
                  <span class="visually-hidden">Siguiente</span>
                </button>
              </div>
            </div>
          </div>
        </div>
      </section>

      <section v-if="prefabricada.descripcion?.trim()" class="py-5 bg-light">
        <div class="container">
          <div class="row">
            <div class="col-lg-6">
              <h2 class="mb-4 fw-bold">Descripción</h2>
              <p class="text-muted descripcion">{{ prefabricada.descripcion }}</p>
            </div>
            
            <div v-if="hasPlanoImages" class="col-lg-6">
              <h2 class="mb-4 fw-bold">Planos</h2>
              <div id="planosCarousel" 
                   class="carousel slide" 
                   @touchstart="handleTouchStart"
                   @touchend="(e) => handleTouchEnd(e, true)">
                <div class="carousel-indicators">
                  <button v-for="(_, index) in planoImages"
                          :key="index"
                          type="button"
                          :class="{ active: index === currentPlanoIndex }"
                          :aria-current="index === currentPlanoIndex"
                          @click="currentPlanoIndex = index">
                  </button>
                </div>

                <div class="carousel-inner">
                  <div v-for="(plano, index) in planoImages" 
                       :key="index"
                       class="carousel-item"
                       :class="{ active: currentPlanoIndex === index }">
                    <img :src="plano.image" 
                         :alt="`Plano ${index + 1}`"
                         class="d-block w-100"
                         @error="handleImageError"
                         @click="openLightbox(index, true)">
                  </div>
                </div>

                <button class="carousel-control-prev" @click.prevent="prevPlano">
                  <span class="carousel-control-prev-icon"></span>
                  <span class="visually-hidden">Anterior</span>
                </button>
                <button class="carousel-control-next" @click.prevent="nextPlano">
                  <span class="carousel-control-next-icon"></span>
                  <span class="visually-hidden">Siguiente</span>
                </button>
              </div>
            </div>
          </div>
        </div>
      </section>

      <section v-if="prefabricada.caracteristicas?.length" class="features-section py-5">
        <div class="container">
          <h2 class="text-center mb-5 fw-bold">Características Destacadas</h2>
          <div class="row row-cols-2 row-cols-md-3 g-4">
            <div v-for="(caracteristica, index) in prefabricada.caracteristicas" 
                :key="index"
                class="col">
              <div class="card h-100 border-0 shadow-sm">
                <div class="card-body">
                  <div class="feature-icon mb-3">
                    <i :class="getFeatureIcon(caracteristica.clave)" class="fa-2x"></i>
                  </div>
                  <h5 class="card-title">{{ caracteristica.clave }}</h5>
                  <p class="card-text">{{ caracteristica.valor }}</p>
                </div>
              </div>
            </div>
          </div>
        </div>
      </section>

      <section v-if="prefabricada.precios?.length"
               id="precios" 
               class="pricing-section py-5">
        <div class="container">
          <h2 class="text-center mb-5 fw-bold section-title">Precios Disponibles</h2>
          <div class="pricing-cards">
            <div v-for="precio in prefabricada.precios" 
                 :key="precio.id" 
                 class="pricing-card-wrapper mb-4">
              <div class="card h-100 pricing-card">
                <div class="card-header text-center py-4">
                  <h3 class="card-title mb-0">{{ precio.nombre_precio }}</h3>
                </div>
                <div class="card-body d-flex flex-column">
                  <div class="price-container mb-4">
                    <span class="period text-light">Desde: </span>
                    <span class="currency">$</span>
                    <span class="price">{{ Math.floor(precio.valor_prefabricada).toLocaleString() }}</span>
                  </div>
                  <p class="text-center mb-4 text-light">{{ precio.descripcion_precio }}</p>
                  <ul v-if="precio.incluyes?.length" class="list-unstyled mb-4 flex-grow-1">
                    <li v-for="incluye in precio.incluyes" 
                        :key="incluye.id" 
                        class="mb-3 text-light">
                      <i class="fas fa-minus me-2 text-primary"></i>
                      {{ incluye.nombre_incluye }}
                    </li>
                  </ul>
                </div>
                <div class="accordion mt-4" :id="'accordion-' + precio.id">
                  <div class="accordion-item" v-if="precio.adicionales?.length">
                    <h2 class="accordion-header" :id="'heading-' + precio.id">
                      <button 
                        class="accordion-button collapsed w-100 text-left text-dark"
                        type="button" 
                        @click="toggleAccordion(precio.id)"
                        :aria-expanded="isAccordionOpen(precio.id)"
                        :aria-controls="'collapse-' + precio.id">
                        <i class="fas fa-plus-circle me-2"></i>
                        Adicionales que puedes agregar
                      </button>
                    </h2>
                    <div 
                      :id="'collapse-' + precio.id" 
                      class="accordion-collapse collapse"
                      :class="{ 'show': isAccordionOpen(precio.id) }"
                      :aria-labelledby="'heading-' + precio.id">
                      <div class="accordion-body">
                        <div v-for="adicional in precio.adicionales" 
                             :key="adicional.id"
                             class="adicional-item d-flex justify-content-between align-items-center">
                          <span>{{ adicional.nombre_adicional }}</span>
                          <span class="fw-bold text-success">${{ adicional.valor_adicional.toLocaleString() }}</span>
                        </div>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </section>

      <EjecutivosVentas />

      <!-- <section class="footer-banner">
        <div class="banner-overlay">
          <h2 class="banner-text animate__animated animate__fadeIn animate__delay-1s">
            Casas Prefabricadas Jara
          </h2>
        </div>
      </section> -->

      <Transition name="fade">
        <div v-if="lightboxOpen" 
             class="lightbox"
             @click="closeLightbox"
             @touchstart="handleTouchStart"
             @touchend="(e) => handleTouchEnd(e, isPlanoLightbox)">
          <button class="close-button" @click.stop="closeLightbox">
            <i class="fas fa-times"></i>
          </button>
          <button class="nav-button prev" @click.stop="lightboxPrev">
            <i class="fas fa-chevron-left"></i>
          </button>
          <button class="nav-button next" @click.stop="lightboxNext">
            <i class="fas fa-chevron-right"></i>
          </button>
          <img :src="isPlanoLightbox ? currentPlanoImage : currentRegularImage"
               :alt="`Vista ampliada ${isPlanoLightbox ? 'plano' : 'imagen'}`"
               class="lightbox-image"
               @error="handleImageError">
        </div>
      </Transition>
    </div>
  </div>
</template>

<style scoped>
/* :root {
  --primary-color: #f7b500;
  --secondary-color: #1e1f21;
  --accent-color: #ff6b35;
  --text-color: #ffffff;
  --light-bg: #f8f9fa;
  --dark-bg: #2c2d30;
} */

.prefabricada-details {
  font-family: 'Arial', sans-serif;
  color: var(--secondary-color);
}

.hero-section {
  background: linear-gradient(135deg, var(--primary-color) 0%, var(--secondary-color) 100%);
  color: var(--text-color);
}

.pricing-section {
  background: linear-gradient(135deg, var(--secondary-color) 0%, var(--dark-bg) 100%);
  color: var(--text-color);
}

.feature-icon {
  width: 60px;
  height: 60px;
  border-radius: 50%;
  background-color: var(--primary-color);
  color: var(--secondary-color);
  display: flex;
  align-items: center;
  justify-content: center;
}

.card {
  transition: transform 0.3s ease-in-out, box-shadow 0.3s ease-in-out;
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  border: none;
}

.card:hover {
  transform: translateY(-5px);
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
}

.btn-primary {
  background-color: var(--primary-color);
  border-color: var(--primary-color);
  color: var(--secondary-color);
  transition: all 0.3s ease;
}

.btn-primary:hover {
  background-color: var(--accent-color);
  border-color: var(--accent-color);
  transform: translateY(-3px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.lightbox {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.9);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.lightbox-image {
  max-width: 90%;
  max-height: 90%;
  object-fit: contain;
}

.close-button,
.nav-button {
  position: absolute;
  background: none;
  border: none;
  color: var(--text-color);
  font-size: 2rem;
  cursor: pointer;
  transition: color 0.3s ease;
  padding: 1rem;
  z-index: 1001;
}

.close-button:hover,
.nav-button:hover {
  color: var(--primary-color);
}

.close-button {
  top: 20px;
  right: 20px;
}

.nav-button {
  top: 50%;
  transform: translateY(-50%);
}

.prev { left: 20px; }
.next { right: 20px; }

.carousel {
  touch-action: pan-y pinch-zoom;
}

.carousel-indicators {
  margin-bottom: 0;
  bottom: -2.5rem;
}

.carousel-indicators button {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  background-color: var(--primary-color);
  opacity: 0.5;
  transition: all 0.3s ease;
  margin: 0 5px;
  border: none;
}

.carousel-indicators button.active {
  background-color: var(--accent-color);
  opacity: 1;
  transform: scale(1.2);
}

#planosCarousel {
  background: white;
  border-radius: 8px;
  overflow: hidden;
}

#planosCarousel .carousel-inner {
  border-radius: 8px;
}

#planosCarousel img {
  max-height: 400px;
  object-fit: contain;
  background-color: white;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.descripcion {
  overflow-wrap: break-word;
  word-break: break-word;
}

.pricing-section {
  padding: 4rem 0;
}

.section-title {
  font-size: 2.5rem;
  font-weight: 700;
  color: var(--primary-color);
  margin-bottom: 2rem;
}

.pricing-cards {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.pricing-card-wrapper {
  width: 100%;
  max-width: 800px;
}

.pricing-card {
  background-color: rgba(255, 255, 255, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 10px;
  overflow: hidden;
  transition: all 0.3s ease;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
}

.pricing-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.2);
}

.pricing-card .card-header {
  background-color: rgba(255, 255, 255, 0.05);
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
  padding: 1.5rem;
}

.pricing-card .card-title {
  color: var(--primary-color);
  font-size: 1.5rem;
  font-weight: 600;
}

.pricing-card .card-body {
  padding: 2rem;
}

.price-container {
  text-align: center;
  color: var(--primary-color);
}

.currency {
  font-size: 1.5rem;
  font-weight: 600;
  vertical-align: top;
  line-height: 1;
}

.price {
  font-size: 3rem;
  font-weight: 700;
  line-height: 1;
}

.period {
  font-size: 1rem;
  color: var(--text-color);
}

.pricing-card ul li {
  color: var(--text-color);
  font-size: 1rem;
}

.accordion-item {
  border: none;
  margin-bottom: 0.5rem;
}

.accordion-button {
  background-color: rgba(255, 255, 255, 0.05);
  color: var(--text-color);
  font-weight: 600;
  padding: 1rem 1.25rem;
  border-radius: 0.5rem;
  transition: all 0.3s ease;
}

.accordion-button:not(.collapsed) {
  background-color: var(--primary-color);
  color: var(--secondary-color);
  box-shadow: none;
}

.accordion-button:focus {
  box-shadow: none;
  border-color: rgba(255, 255, 255, 0.1);
}

.accordion-button::after {
  background-size: 1rem;
  transition: transform 0.3s ease;
}

.accordion-button:not(.collapsed)::after {
  background-image: url("data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 16 16' fill='%231e1f21'%3e%3cpath fill-rule='evenodd' d='M1.646 4.646a.5.5 0 0 1 .708 0L8 10.293l5.646-5.647a.5.5 0 0 1 .708.708l-6 6a.5.5 0 0 1-.708 0l-6-6a.5.5 0 0 1 0-.708z'/%3e%3c/svg%3e");
  transform: rotate(-180deg);
}

.accordion-body {
  background-color: rgba(255, 255, 255, 0.05);
  border-bottom-left-radius: 0.5rem;
  border-bottom-right-radius: 0.5rem;
  padding: 1rem 1.25rem;
}

.adicional-item {
  margin-bottom: 0.5rem;
  padding: 0.5rem;
  background-color: rgba(255, 255, 255, 0.1);
  border-radius: 0.25rem;
  transition: background-color 0.3s ease;
}

.adicional-item:hover {
  background-color: rgba(255, 255, 255, 0.15);
}

.adicional-item .fw-bold {
  font-size: 0.9rem;
}

.footer-banner {
  position: relative;
  width: 100%;
  height: 300px;
  background-image: url('/img/casas_charlotte_3.jpg');
  background-size: cover;
  background-position: center;
}

.banner-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.banner-text {
  color: #ffffff;
  font-size: 3.4rem;
  font-weight: bold;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
}

.detail-header {
  background: var(--secondary-color);
  padding: 1.5rem 0;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.breadcrumb-nav {
  display: flex;
  align-items: center;
}

.back-button {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.5rem 1rem;
  border: none;
  background: transparent;
  color: var(--primary-color);
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
}

.back-button:hover {
  color: var(--accent-color);
  transform: translateX(-4px);
}

.back-icon {
  font-size: 1.2rem;
}

@media (max-width: 991px) {
  .pricing-card-wrapper {
    padding: 0 1rem;
  }
}

@media (max-width: 768px) {
  .hero-section {
    text-align: center;
  }

  .hero-section .btn {
    width: 100%;
    margin-top: 2rem;
  }

  .carousel {
    margin-top: 2rem;
  }

  .nav-button {
    font-size: 1.5rem;
    padding: 0.5rem;
  }

  .lightbox-image {
    max-width: 95%;
    max-height: 95%;
  }

  .section-title {
    font-size: 2rem;
  }
  
  .pricing-card .card-body {
    padding: 1.5rem;
  }
  
  .price {
    font-size: 2.5rem;
  }

  .footer-banner {
    height: 200px;
  }
  
  .banner-text {
    font-size: 1.5rem;
  }
}
</style>
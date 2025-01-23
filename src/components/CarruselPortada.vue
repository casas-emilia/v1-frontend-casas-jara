<script setup>
import { ref, onMounted, watch, nextTick } from 'vue'
import axios from 'axios'
import { Carousel } from 'bootstrap'

const images = ref([])
const loading = ref(true)
const error = ref(null)
let carouselInstance = null
const progressValue = ref(0)
let progressInterval = null

const fetchImages = async () => {
    try {
        const response = await axios.get('/empresas/1/portadas/')
        images.value = response.data.portadas
    } catch (err) {
        console.error('Error al obtener las imágenes:', err)
        error.value = 'Hubo un error al cargar las imágenes. Por favor, intenta de nuevo más tarde.'
    } finally {
        loading.value = false
    }
}

const initCarousel = () => {
    const carouselElement = document.getElementById('mainCarousel')
    if (carouselElement) {
        if (carouselInstance) {
            carouselInstance.dispose()
        }
        carouselInstance = new Carousel(carouselElement, {
            interval: 5000,
            wrap: true,
            pause: 'hover'
        })
        carouselElement.addEventListener('slide.bs.carousel', resetProgress)
        startProgress()
    }
}

const startProgress = () => {
    clearInterval(progressInterval)
    progressValue.value = 0
    const duration = 5000 // Debe coincidir con el intervalo del carrusel
    const step = 100 / (duration / 50) // Actualiza cada 50ms
    progressInterval = setInterval(() => {
        progressValue.value += step
        if (progressValue.value >= 100) {
            clearInterval(progressInterval)
            nextTick(() => {
                if (carouselInstance) {
                    carouselInstance.next()
                }
            })
        }
    }, 50)
}

const resetProgress = () => {
    clearInterval(progressInterval)
    progressValue.value = 0
    nextTick(startProgress)
}

const handleImageClick = (link) => {
  if (link && link !== 'null' && link !== 'undefined' && link.trim() !== '') {
    window.location.href = link;
  }
}

onMounted(() => {
    fetchImages()
})

watch(images, () => {
    if (images.value.length > 0) {
        nextTick(initCarousel)
    }
})
</script>

<template>
    <div class="carousel-container">
        <div v-if="loading" class="text-center p-5">
            <div class="spinner-border text-primary" role="status">
                <span class="visually-hidden">Cargando...</span>
            </div>
        </div>
        
        <div v-else-if="error" class="alert alert-danger" role="alert">
            {{ error }}
        </div>
        
        <div v-else-if="images.length > 0" 
             id="mainCarousel" 
             class="carousel slide" 
             data-bs-ride="carousel">
            <div class="carousel-progress-bar">
                <div class="progress-inner" :style="{ width: `${progressValue}%` }"></div>
            </div>
            
            <div class="carousel-indicators">
                <button
                    v-for="(img, index) in images"
                    :key="'indicator-' + img.id"
                    type="button"
                    :data-bs-target="'#mainCarousel'"
                    :data-bs-slide-to="index"
                    :class="{ active: index === 0 }"
                    :aria-current="index === 0"
                    :aria-label="'Slide ' + (index + 1)">
                </button>
            </div>
            
            <div class="carousel-inner">
                <div 
                    v-for="(img, index) in images" 
                    :key="img.id" 
                    class="carousel-item"
                    :class="{ 
                        active: index === 0, 
                        'cursor-pointer': img.link && img.link !== 'null' && img.link !== 'undefined' && img.link.trim() !== '' 
                    }"
                    @click="handleImageClick(img.link)"
                >
                    <img 
                        :src="img.image"
                        class="d-block w-100" 
                        :alt="img.alt || img.nombre_portada"
                    >
                    <div class="carousel-gradient"></div>
                </div>
            </div>
            
            <button class="carousel-control-prev" type="button" data-bs-target="#mainCarousel" data-bs-slide="prev">
                <span class="carousel-control-prev-icon" aria-hidden="true"></span>
                <span class="visually-hidden">Anterior</span>
            </button>
            <button class="carousel-control-next" type="button" data-bs-target="#mainCarousel" data-bs-slide="next">
                <span class="carousel-control-next-icon" aria-hidden="true"></span>
                <span class="visually-hidden">Siguiente</span>
            </button>
        </div>
        
        <div v-else class="alert alert-info" role="alert">
            No hay imágenes disponibles para mostrar.
        </div>
    </div>
</template>

<style scoped>
.carousel-container {
    width: 100%;
    max-width: 100vw;
    position: relative;
    overflow: hidden;
}

.carousel-item {
    position: relative;
    width: 100%;
    height: calc(100vw * 9 / 16);
    background-color: #f8f9fa;
}

.carousel-item img {
    width: 100%;
    height: 100%;
    object-fit: cover;
    object-position: center;
}

.carousel-progress-bar {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 5px;
    background-color: rgba(0, 0, 0, 0.2);
    z-index: 10;
}

.progress-inner {
    height: 100%;
    background-color: #ff6b6b;
    transition: width 0.05s linear;
}

.carousel-indicators {
    margin-bottom: 1rem;
    z-index: 15;
}

.carousel-indicators button {
    width: 10px;
    height: 10px;
    border-radius: 50%;
    background-color: rgba(255, 255, 255, 0.5);
    border: none;
    margin: 0 5px;
}

.carousel-indicators button.active {
    background-color: #ffffff;
}

.carousel-gradient {
    position: absolute;
    bottom: 0;
    left: 0;
    right: 0;
    height: 150px;
    background: linear-gradient(to top, rgba(0,0,0,0.7) 0%, rgba(0,0,0,0) 100%);
    pointer-events: none;
}

.cursor-pointer {
    cursor: pointer;
}
</style>


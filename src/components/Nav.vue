<script setup>
import { ref } from 'vue'
import { RouterLink } from 'vue-router'

const navItems = ref([
    { text: 'Nosotros', route: 'nosotros', icon: 'fas fa-info-circle' },
    { text: 'Prefabricadas', route: 'prefabricadas', icon: 'fas fa-home', highlight: true },
    { text: 'Contacto', route: 'contactos', icon: 'fas fa-envelope' },
    { text: 'Portafolio', route: 'blog', icon: 'fas fa-briefcase' }
])


const isNavbarOpen = ref(false)

const toggleNavbar = () => {
    isNavbarOpen.value = !isNavbarOpen.value
}

const closeNavbar = () => {
    isNavbarOpen.value = false
}
</script>

<template>
    <nav class="navbar navbar-expand-lg" style="background-color: #1e1f21;">
        <div class="container">
            <RouterLink :to="{ name: 'home' }" class="navbar-brand fw-light fs-6 text-light"
                style="color: #0f2b5e; font-family: 'Cabin', sans-serif;">
                <img src="/img/logo_casas_jara.png" alt="home" width="73" height="70">
                Casas Prefabricadas Jara
            </RouterLink>
            <button class="navbar-toggler custom-toggler" type="button" @click="toggleNavbar"
                aria-controls="navbarSupportedContent" aria-expanded="false"
                aria-label="Toggle navigation">
                <span class="navbar-toggler-icon"></span>
            </button>
            <div :class="['collapse navbar-collapse', { 'show': isNavbarOpen }]" id="navbarSupportedContent" style="font-family: 'Cabin', sans-serif;">
                <ul class="navbar-nav ms-auto mb-2 mb-lg-0">
                    <li v-for="(item, index) in navItems" :key="index" class="nav-item mx-2">
                        <RouterLink :to="{ name: item.route }" 
                            :class="['nav-link active fw-bold nav-item-hover d-flex align-items-center nav-item', 
                                     {'highlight-nav-item': item.highlight}]"
                            @click="closeNavbar">
                            <i :class="[item.icon, 'me-2']"></i>
                            {{ item.text }}
                        </RouterLink>
                    </li>
                </ul>
            </div>
        </div>
    </nav>
</template>

<style scoped>
.nav-item {
    color:#fefefe !important;
}

.nav-item-hover {
    transition: all 0.3s ease;
    padding: 8px 15px;
}

.nav-item-hover:hover {
    background-color: var(--primary-color);
    color: #000000 !important;
    border-radius: 5px;
}

.highlight-nav-item {
  /* Gradiente con los colores definidos */
  background: linear-gradient(
    -45deg,
    #f7b500, /* Amarillo dorado (Principal) */
    #ff5a5f, /* Rojo coral (Contraste) */
    #0a84ff, /* Azul vibrante (Complemento) */
    #20212b  /* Gris oscuro (Profundidad) */
  );
  background-size: 300% 300%;
  animation: gradient 10s ease infinite;

  /* Texto */
  color: #fefefe !important; /* Blanco para contraste */
  font-weight: bold;
  font-size: 14px;
  text-transform: uppercase;

  /* Espaciado y bordes */
  padding: 10px 20px;
  border-radius: 30px;

  /* Sombras para dar profundidad */
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.2), 
              0 2px 4px rgba(255, 255, 255, 0.1) inset;

  /* Efecto inicial */
  transform: scale(1);
  transition: all 0.3s ease;
}

.highlight-nav-item:hover {
  /* Acelera la animación en hover */
  animation: gradient 6s ease infinite;

  /* Transformaciones al interactuar */
  transform: scale(1.1);
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.3), 
              0 2px 6px rgba(255, 255, 255, 0.15) inset;
}
@keyframes gradient {
    0% {
        background-position: 0% 50%;
    }
    50% {
        background-position: 100% 50%;
    }
    100% {
        background-position: 0% 50%;
    }
}

.custom-toggler .navbar-toggler-icon {
    background-image: url("data:image/svg+xml;charset=utf8,%3Csvg viewBox='0 0 32 32' xmlns='http://www.w3.org/2000/svg'%3E%3Cpath stroke='rgba(0, 123, 255, 1)' stroke-width='2' stroke-linecap='round' stroke-miterlimit='10' d='M4 8h24M4 16h24M4 24h24'/%3E%3C/svg%3E");
}

.custom-toggler.navbar-toggler {
    border-color: #20212b;
}

@media (max-width: 991.98px) {
    .navbar-collapse {
        transition: max-height 0.3s ease-in-out;
        max-height: 0;
        overflow: hidden;
    }

    .navbar-collapse.show {
        max-height: 100vh;
    }
}
</style>


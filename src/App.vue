<template>
  <div class="app-container">
    <!-- الهيدر -->
    <header class="header-container">
      <div class="left-section">
        <div
          class="hamburger-button"
          @click="toggleMenu"
        >
          <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-6">
            <path stroke-linecap="round" stroke-linejoin="round" d="M3.75 6.75h16.5M3.75 12h16.5m-16.5 5.25h16.5" />
          </svg>
        </div>
      </div>
      <div class="center-section">
        <nav class="nav-bar">
          <router-link to="/">Home</router-link>
          <router-link to="/history">History</router-link>
          <router-link to="/about">About</router-link>
        </nav>
      </div>
      <div class="right-section">
        <!-- <div class="mode-switcher" @click="toggleTheme">
          <svg v-if="!isLightMode" xmlns="http://www.w3.org/2000/svg" width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-sun-icon lucide-sun"><circle cx="12" cy="12" r="4"/><path d="M12 2v2"/><path d="M12 20v2"/><path d="m4.93 4.93 1.41 1.41"/><path d="m17.66 17.66 1.41 1.41"/><path d="M2 12h2"/><path d="M20 12h2"/><path d="m6.34 17.66-1.41 1.41"/><path d="m19.07 4.93-1.41 1.41"/></svg>
          <svg v-else xmlns="http://www.w3.org/2000/svg" width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-moon-star-icon lucide-moon-star"><path d="M18 5h4"/><path d="M20 3v4"/><path d="M20.985 12.486a9 9 0 1 1-9.473-9.472c.405-.022.617.46.402.803a6 6 0 0 0 8.268 8.268c.344-.215.825-.004.803.401"/></svg>
        </div> -->
        <div class="center-section">
  <a href="/"><img src="/src/assets/photos/logo.png" alt="Logo" class="mobile-logo" /></a>
</div>

      </div>
    </header>
    <!-- قائمة الجوال -->
    <transition name="menu-slide">
      <div class="menu-mobile" v-if="isMobileMenuOpen">
        <nav>
        <router-link to="/" @click="closeMenu">Home</router-link>
          <hr>
          <router-link to="/history" @click.native="closeMenu">History</router-link>
          <hr>
          <router-link to="/about" @click.native="closeMenu">About</router-link>
        </nav>
      </div>
    </transition>
    <div v-if="isMobileMenuOpen" class="menu-overlay" @click="closeMenu"></div>
    <!-- المحتوى الرئيسي -->
    <main class="main-container">
      <router-view />
    </main>
  </div>
</template>

<script>
export default {
  name: "App",
  data() {
    return {
      isMobileMenuOpen: false,
      isLightMode: false,
    };
  },
  mounted() {
    const root = document.documentElement;
    if (!root.hasAttribute('data-theme')) {
      root.removeAttribute('data-theme');
    }
    this.isLightMode = root.getAttribute('data-theme') === 'light';
  },
  methods: {
    toggleMenu() {
      this.isMobileMenuOpen = !this.isMobileMenuOpen;
    },
    closeMenu() {
      this.isMobileMenuOpen = false;
    },
    toggleTheme() {
      this.isLightMode = !this.isLightMode;
      const root = document.documentElement;
      if (this.isLightMode) {
        root.setAttribute('data-theme', 'light');
      } else {
        root.removeAttribute('data-theme');
      }
    },
  },
};
</script>

<style >
:root {
  --main-color: #111827;
  --White: #fbfbfb;
  --secend-color: #3B82F6;
    --gray:#1b2b4d;
    --hr:#505760;
    --support-color:#18233b;
}

html[data-theme="light"] {
  --main-color: #f6f0df;  
  --White:      #0F1724;    
  --secend-color: #2563EB;
  --gray:         #eae7df;  
  --support-color:#e5ddc7;  
  --hr: #D1D5DB;            
}

body, html {
  font-family: 'Open Sans', system-ui, sans-serif;
  height: 100%;
  margin: 0;
  padding: 0;
  background-color: var(--main-color);
  color: var(--White);
    overflow-x: hidden;
}

#app {
  height: 100vh;
}
.mobile-logo {
  display: none;
}
.app-container {
  height: 100vh;
  width: 100vw;
  display: flex;
  flex-direction: column;
}

.header-container {
  width: 100%;
  height: 5.25rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
  background-color: var(--support-color);
  position: fixed;
  z-index: 1000;
  top: 0;
  left: 0;
  padding: 0 2rem;
  box-sizing: border-box;
}

.left-section {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: flex-start;
  min-width: 80px;
}

.center-section {
  flex: 2;
  display: flex;
  justify-content: center;
  align-items: center;
}

.right-section {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: flex-end;
  min-width: 80px;
}

.nav-bar {
  display: flex;
  gap: 2rem;
}

.nav-bar a {
  color: var(--White);
  text-decoration: none;
  font-size: 1.1rem;
  position: relative;
  padding: 0.5rem 1rem;
  transition: color 0.3s cubic-bezier(.4,0,.2,1);
  overflow: hidden;
    letter-spacing: 1px;
}

.nav-bar a::after {
  content: '';
  position: absolute;
  left: 50%;
  bottom: 0;
  transform: translateX(-50%) scaleX(0);
  width: 80%;
  height: 3px;
  background: var(--secend-color);
  border-radius: 2px;
  transition: transform 0.4s cubic-bezier(.4,0,.2,1);
}

.nav-bar a:hover {
  color: var(--secend-color);

}

.nav-bar a:hover::after {
  transform: translateX(-50%) scaleX(1);
}

.hamburger-button{
  display: none; /* keep default hidden on desktop */
  width: 40px;
  height: 40px;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  background: transparent;
  border: none;
  box-sizing: border-box;
  padding: 0;
}
.mode-switcher {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: .5rem;
  border-radius: 8px;
  cursor: pointer;
  transition: background 0.2s;
  margin-left: 0.5rem;
}

.main-container {
  flex: 1;
  width: 100%;
  min-height: calc(100% - 5.25rem);
  background-color: var(--main-color);
  margin-top: 5.25rem;

}

/* قائمة الجوال */
.menu-mobile {
  position: fixed;
  top: 5.25rem;
  left: 0;
  height: calc(100vh - 5.25rem); 
  width: 60vw;                 
  max-width: 420px;           
  background:var(--gray);
  z-index: 1200;
  will-change: transform, opacity;
  touch-action: pan-y;

}

.menu-mobile nav {
  display: flex;
  flex-direction: column;
  align-items: stretch;
  text-align: center;
}

.menu-mobile nav a {
  color: var(--White);
  text-decoration: none;
  padding: 1.2rem 0;
  font-size: 1.2rem;
  background: none;
}

.menu-mobile nav hr {
  display: block;
  width: 100%;
  height: 1px;               /* اجعلها مرئية */
  margin: 0;
  border: none;              /* تأكد من عدم وجود حدود متضاربة */
  background-color: var(--hr); /* لون الخط */
  opacity: 1;
}

.menu-overlay {
  position: fixed;
  top: 5.25rem; 
  left: 0;
  width: 100vw;
  height: calc(100vh - 5.25rem);
  background: rgba(0,0,0,0.32); 
  z-index: 1100; 
  transition: opacity 260ms ease;
  opacity: 1;
}
/* transition classes */
.menu-slide-enter-from,
.menu-slide-leave-to {
  transform: translateX(-100%);
  opacity: 0;
}

.menu-slide-enter-to,
.menu-slide-leave-from {
  transform: translateX(0);
  opacity: 1;
}

.menu-slide-enter-active,
.menu-slide-leave-active {
  transition: transform 0.4s ease, opacity 0.4s ease;
}

@media (max-width: 1024px) { 
    .mobile-logo {
    display: block;
    height: 30px;   /* adjust as needed */
    width: auto;    /* keeps aspect ratio */
    margin-right: .3rem; /* optional spacing */
  }
.nav-bar {
 display: none;

}
.hamburger-button {
 display: flex;
}
.main-container {
 margin-top: 5.25rem;
}
.header-container{
 padding: 0 1rem;    
}

}
</style>
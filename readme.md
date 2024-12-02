# **Proiect Svelte: Galerie de Imagini cu Animații**

---

## **Introducere**

Am creat un proiect în **Svelte** care prezintă o galerie de imagini interactivă cu animații elegante. Proiectul îndeplinește următoarele cerințe:

- **Pagina principală** cu trei categorii reprezentate prin imagini mici.
- **Animații** la interacțiunea cu mouse-ul (hover și click).
- **Galerie de imagini** pentru fiecare categorie, cu navigare orizontală controlată prin scroll vertical.
- **Vizualizare full-screen** a imaginilor, cu posibilitatea de a naviga între ele și de a închide vizualizarea.

---

## **Structura Proiectului**

- **`App.svelte`**: Componenta principală care afișează categoriile pe pagina principală.
- **`Gallery.svelte`**: Componenta care gestionează afișarea imaginilor dintr-o categorie selectată.
- **Imaginile**: Organizate în foldere pentru fiecare categorie în directorul `public/images`.

---

## **Detalii de Implementare**

### **1. Pagina Principală (`App.svelte`)**

#### **Categoriile**

- Trei categorii reprezentate prin imagini:

    1. **Anime**
    2. **Games**
    3. **IT**

- Imaginile au dimensiunea de **450x450 pixeli**, colțuri rotunjite și sunt aranjate orizontal, centrate pe pagină.

#### **Interacțiune și Animații**

- **Animație la Hover**:

    - Imaginea se ridică ușor (10px).
    - Apare un text sub imagine cu numele categoriei, animat în jos.
    - Se adaugă o umbră pentru un efect vizual plăcut.

- **Click pe Categorie**:

    - Deschide galeria corespunzătoare categoriei selectate.

#### **Codul Sursa**

```svelte
<script>
  import Gallery from './Gallery.svelte';

  let showGallery = false;
  let selectedCategory = '';
  let categories = [
    { name: 'Anime', image: '/images/categorie1/thumb.jpg', folder: 'categorie1' },
    { name: 'Games', image: '/images/categorie2/thumb.jpg', folder: 'categorie2' },
    { name: 'IT', image: '/images/categorie3/thumb.jpg', folder: 'categorie3' },
  ];

  function openGallery(category) {
    selectedCategory = category;
    showGallery = true;
  }

  function closeGallery() {
    showGallery = false;
  }
</script>

<style>
  /* Fundal cu gradient roșu-negru */
  :global(body) {
    margin: 0;
    padding: 0;
    background: linear-gradient(to bottom, red, black);
  }

  .container {
    position: relative;
    width: 100%;
    height: 100vh;
  }

  .categories {
    display: flex;
    justify-content: center;
    align-items: center;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
  }

  .category-wrapper {
    position: relative;
    margin: 0 10px;
    text-align: center;
    cursor: pointer;
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .category {
    transition: transform 0.3s, box-shadow 0.3s;
    width: 450px;
    height: 450px;
    object-fit: cover;
    border-radius: 15px;
  }

  .category-wrapper:hover .category {
    transform: translateY(-10px);
    box-shadow: 0 10px 20px rgba(0, 0, 0, 0.3);
  }

  .category-name {
    margin-top: 10px;
    opacity: 0;
    transform: translateY(-10px);
    transition: opacity 0.3s, transform 0.3s;
    color: #fff;
    font-size: 24px;
    font-weight: bold;
  }

  .category-wrapper:hover .category-name {
    opacity: 1;
    transform: translateY(0);
  }

  .overlay {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-color: rgba(0, 0, 0, 0.8);
    display: flex;
    justify-content: center;
    align-items: center;
    animation: fadeIn 0.5s;
  }

  @keyframes fadeIn {
    from { opacity: 0; }
    to { opacity: 1; }
  }
</style>

<div class="container">
  <div class="categories">
    {#each categories as category}
      <div class="category-wrapper" on:click={() => openGallery(category.folder)}>
        <img
          src="{category.image}"
          alt="{category.name}"
          class="category"
        />
        <div class="category-name">{category.name}</div>
      </div>
    {/each}
  </div>
</div>

{#if showGallery}
  <div class="overlay" on:click={closeGallery}>
    <Gallery {selectedCategory} on:closeGallery={closeGallery} />
  </div>
{/if}
```

---

### **2. Galeria (`Gallery.svelte`)**

#### **Afișarea Imaginilor**

- Imaginile din categoria selectată sunt afișate într-un rând orizontal.
- Dimensiunile imaginilor: **300x200 pixeli**.
- Colțuri rotunjite și efecte de umbră la hover.

#### **Interacțiune și Animații**

- **Scroll Vertical**:

    - Controlează mișcarea orizontală a imaginilor.
    - Scroll în sus: imaginile se mișcă spre dreapta.
    - Scroll în jos: imaginile se mișcă spre stânga.

- **Click pe Imagine**:

    - Deschide imaginea în modul full-screen.

#### **Modul Full-Screen**

- **Afișarea Imaginii Mărite**:

    - Imaginea selectată este afișată pe întreg ecranul.

- **Navigare între Imagini**:

    - Scroll vertical, tastele săgeți sau butoane de navigare.

- **Închiderea Vizualizării**:

    - Click pe butonul de închidere sau apăsarea tastei `Escape`.

#### **Codul Sursa**

```svelte
<script>
  import { onMount } from 'svelte';
  import { createEventDispatcher } from 'svelte';
  export let selectedCategory;

  const dispatch = createEventDispatcher();
  let images = [];
  let translateX = 0;
  let showFullScreen = false;
  let currentImageIndex = 0;

  onMount(() => {
    fetchImages();
    window.addEventListener('keydown', handleKeyDown);
  });

  function handleKeyDown(event) {
    if (event.key === 'Escape') {
      if (showFullScreen) {
        showFullScreen = false;
      } else {
        closeGallery();
      }
    } else if (event.key === 'ArrowRight' && showFullScreen) {
      nextImage();
    } else if (event.key === 'ArrowLeft' && showFullScreen) {
      prevImage();
    }
  }

  function closeGallery() {
    dispatch('closeGallery');
    window.removeEventListener('keydown', handleKeyDown);
  }

  function fetchImages() {
    const numberOfImages = 10; // Modificați în funcție de numărul de imagini
    images = [];
    for (let i = 1; i <= numberOfImages; i++) {
      images.push(`/images/${selectedCategory}/${i}.jpg`);
    }
  }

  function handleWheel(event) {
    const maxTranslateX = -(images.length * 310 - window.innerWidth * 0.8);
    if (event.deltaY > 0) {
      translateX -= 30;
      if (translateX < maxTranslateX) translateX = maxTranslateX;
    } else {
      translateX += 30;
      if (translateX > 0) translateX = 0;
    }
  }

  function openFullScreen(index) {
    currentImageIndex = index;
    showFullScreen = true;
  }

  function nextImage() {
    if (currentImageIndex < images.length - 1) {
      currentImageIndex += 1;
    }
  }

  function prevImage() {
    if (currentImageIndex > 0) {
      currentImageIndex -= 1;
    }
  }

  function handleFullScreenWheel(event) {
    if (event.deltaY > 0) {
      nextImage();
    } else {
      prevImage();
    }
  }
</script>

<style>
  .gallery-overlay {
    width: 80%;
    height: 60%;
    overflow: hidden;
    border-radius: 10px;
    position: relative;
  }

  .horizontal-gallery {
    display: flex;
    align-items: center;
    height: 100%;
    transition: transform 0.1s ease-out;
  }

  .horizontal-gallery img {
    margin-right: 10px;
    width: 300px;
    height: 200px;
    object-fit: cover;
    border-radius: 10px;
    flex-shrink: 0;
    transition: transform 0.3s, box-shadow 0.3s;
    cursor: pointer;
  }

  .horizontal-gallery img:hover {
    transform: translateY(-10px);
    box-shadow: 0 10px 20px rgba(0, 0, 0, 0.3);
  }

  /* Stiluri pentru modul full-screen */
  .full-screen-overlay {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-color: rgba(0, 0, 0, 0.9);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 1000;
  }

  .full-screen-image {
    max-width: 90%;
    max-height: 90%;
    border-radius: 10px;
    transition: transform 0.3s;
  }

  .full-screen-image:hover {
    transform: scale(1.02);
  }

  .close-button {
    position: absolute;
    top: 20px;
    right: 30px;
    font-size: 30px;
    color: #fff;
    cursor: pointer;
  }

  .nav-button {
    position: absolute;
    top: 50%;
    color: #fff;
    font-size: 50px;
    cursor: pointer;
    user-select: none;
    transform: translateY(-50%);
  }

  .nav-button.left {
    left: 20px;
  }

  .nav-button.right {
    right: 20px;
  }
</style>

<!-- Galerie de imagini mici -->
<div class="gallery-overlay" on:wheel|preventDefault={handleWheel} on:click|stopPropagation>
  <div
    class="horizontal-gallery"
    style="transform: translateX({translateX}px);"
  >
    {#each images as image, index}
      <img src="{image}" alt="Image" on:click={() => openFullScreen(index)} />
    {/each}
  </div>
</div>

<!-- Vizualizare full-screen -->
{#if showFullScreen}
  <div class="full-screen-overlay" on:wheel|preventDefault={handleFullScreenWheel} on:click|stopPropagation>
    <span class="close-button" on:click={() => (showFullScreen = false)}>&times;</span>
    <img src="{images[currentImageIndex]}" alt="Full Screen Image" class="full-screen-image" />
    {#if currentImageIndex > 0}
      <span class="nav-button left" on:click={prevImage}>&#10094;</span>
    {/if}
    {#if currentImageIndex < images.length - 1}
      <span class="nav-button right" on:click={nextImage}>&#10095;</span>
    {/if}
  </div>
{/if}
```

---

## **Cum să Rulați Proiectul**

1. **Clonați proiectul**:

   ```bash
   npx degit sveltejs/template galerie-imagini
   cd galerie-imagini
   ```

2. **Instalați dependențele**:

   ```bash
   npm install
   ```

3. **Rulați proiectul**:

   ```bash
   npm run dev
   ```

4. **Deschideți în browser**:

    - Accesați `http://localhost:5000` pentru a vedea aplicația în acțiune.

---

## **Structura Fişierelor**

- **`src/App.svelte`**: Componenta principală a aplicației.
- **`src/Gallery.svelte`**: Componenta care gestionează galeria de imagini.
- **`public/images/`**: Directorul care conține imaginile pentru fiecare categorie:
    - `categorie1/` (Anime)
    - `categorie2/` (Games)
    - `categorie3/` (IT)

---
[Video Proiect](1.mp4)
## Autor: Zama Andrei
Grupa: IAFR2101(RO)
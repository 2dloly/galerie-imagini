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
        const maxTranslateX = -(images.length * 310 - window.innerWidth * 0.8); // Calculăm limita maximă
        if (event.deltaY > 0) {
            // Scroll în jos - mutăm imaginile spre stânga
            translateX -= 30;
            if (translateX < maxTranslateX) translateX = maxTranslateX;
        } else {
            // Scroll în sus - mutăm imaginile spre dreapta
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
        /* Fundalul este transparent */
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

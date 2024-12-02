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
	/* Aplicăm gradientul de fundal la nivel global */
	:global(body) {
		margin: 0;
		padding: 0;
		background: linear-gradient(to bottom, red, black);
	}

	.container {
		position: relative;
		width: 100%;
		height: 100vh; /* Înălțimea ecranului */
	}

	.categories {
		display: flex;
		justify-content: center;
		align-items: center;
		position: absolute;
		top: 50%; /* La jumătatea verticală */
		left: 50%; /* La jumătatea orizontală */
		transform: translate(-50%, -50%); /* Centrare perfectă */
		/* Am eliminat background-color pentru a face fundalul transparent */
		padding: 20px;
		border-radius: 10px;
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
		color: #fff; /* Am păstrat culoarea albă pentru contrast */
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

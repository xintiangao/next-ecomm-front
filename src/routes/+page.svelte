<script>
	import { uploadMedia } from '../utils/s3-uploader.js';
	import { isAuthenticated, getTokenFromLocalStorage } from '../utils/auth.js';
	import { PUBLIC_BACKEND_BASE_URL } from '$env/static/public';
	export let data;

	let formErrors = {};
	// let photos = [];

	// async function fetchPhotos() {
	// 	let token = getTokenFromLocalStorage();
	// 	const resp = await fetch(PUBLIC_BACKEND_BASE_URL + '/photos', {
	// 		method: 'GET',
	// 		mode: 'cors',
	// 		headers: {
	// 			'Content-Type': 'application/json',
	// 			Authorization: `Bearer ${token}`
	// 		}
	// 	});
	// 	if (resp.ok) {
	// 		photos = await resp.json();
	// 	} else {
	// 		console.log('Failed to fetch photos');
	// 	}
	// }

	// fetchPhotos();

	

	let showModal = false;

	function toggleModal() {
		showModal = true;
	}

	function closeModal() {
		showModal = false;
	}

	document.addEventListener('click', (event) => {
		if (
			document.querySelector('.popup') &&
			!document.querySelector('.popup').contains(event.target) &&
			!event.target.closest('.btn-ghost')
		) {
			closeModal();
		}
	});

	async function uploadImage(evt) {
		evt.preventDefault();
		const [fileName, fileUrl] = await uploadMedia(evt.target['file'].files[0]);
		const photoData = {
			file: fileUrl,
			name: fileName,
			title: evt.target['title'].value,
			price: evt.target['price'].value,
			description: evt.target['description'].value
		};

		let token = getTokenFromLocalStorage();

		const resp = await fetch(PUBLIC_BACKEND_BASE_URL + '/photos', {
			method: 'POST',
			mode: 'cors',
			headers: {
				'Content-Type': 'application/json',
				Authorization: `Bearer ${token}`
			},

			body: JSON.stringify(photoData)
		}).catch((error) => {
			console.error('Error:', error);
		});

		if (resp && resp.status == 200) {
			await fetchPhotos();
			closeModal();
		} else {
			console.log('Failed to create photo');
		}
	}

	async function checkout(evt) {
		evt.preventDefault();
		const price = evt.target.dataset.price;
		const id = evt.target.dataset.id;

		const orderData = {
			id: id,
			price: price
		};

		try {
			const response = await fetch(PUBLIC_BACKEND_BASE_URL + '/checkout', {
				method: 'POST',
				mode: 'cors',
				headers: {
					'Content-Type': 'application/json'
				},
				body: JSON.stringify(orderData)
			});

			if (response.status == 200) {
				const res = await response.json();
				window.location.replace(res);
			} else {
				console.log('Failed to initiate checkout');
			}
		} catch (error) {
			console.error('Error:', error);
		}
	}
</script>

<svelte:head>
	<script src="/aws-sdk-s3.min.js"></script>
</svelte:head>

{#if $isAuthenticated}
	<div class="flex justify-end">
		<div class="btn btn-ghost gap-2">
			<svg
				xmlns="http://www.w3.org/2000/svg"
				class="h-6 w-6"
				fill="none"
				viewBox="0 0 24 24"
				stroke="currentColor"
				stroke-width="1.5"
			>
				<path stroke-linecap="round" stroke-linejoin="round" d="M12 6v12m6-6H6" />
			</svg>
			<button on:click={toggleModal}> Upload Image </button>
		</div>
	</div>
{/if}

{#if showModal}
	<div class="fixed inset-0 flex items-center justify-center bg-black bg-opacity-50 z-10">
		<div class="z-10 fixed rounded-xl flex justify-center w-[50%] bg-base-100 p-10">
			<div class="popup w-full rounded-xl">
				<form on:submit|preventDefault={uploadImage} class="w-full">
					<div class="flex flex-col lg:flex-row">
						<div class="form-control w-full mt-2">
							<input class="file-input max-w-xs rounded-xl" type="file" name="file" />

							{#if 'file' in formErrors}
								<label class="label" for="file">
									<span class="label-text-alt text-red-500">{formErrors['file']}</span>
								</label>
							{/if}
						</div>
					</div>

					<div class="form-control w-full">
						<label class="label" for="price">
							<span class="label-text">Price</span>
						</label>
						<input
							type="text"
							name="price"
							placeholder="99.99"
							class="input input-bordered w-full rounded-xl"
						/>
					</div>
					<div class="form-control w-full">
						<label class="label" for="title">
							<span class="label-text">Title</span>
						</label>
						<input
							type="text"
							name="title"
							placeholder="Beautiful Mountains"
							class="input input-bordered w-full rounded-xl"
						/>
					</div>
					<div class="form-control w-full">
						<label class="label" for="description">
							<span class="label-text">Description</span>
						</label>
						<textarea
							name="description"
							class="textarea textarea-bordered rounded-xl"
							placeholder="Mountains are awesome"
						/>
					</div>
					<div class="form-control w-full mt-4">
						<button type="submit" class="btn btn-md rounded-xl">Upload</button>
					</div>
				</form>
			</div>
		</div>
	</div>
{/if}

<div class="container mt-3 lg:mt-10 mx-auto px-2 lg:px-0">
	<div class="grid grid-rows-1 lg:grid-cols-3 gap-4">
		{#each data.photos as photo}
			<div class="card bg-base-100 shadow-xl">
				<figure class="h-60">
					<img src={photo.file} alt={photo.name} />
				</figure>
				<div class="card-body">
					<h2 class="card-title">
						{photo.title}
						<div class="badge badge-secondary">NEW</div>
					</h2>
					<p>{photo.description}</p>
					<div class="card-actions items-end justify-end">
						<h3 class="text-xl font-thin mr-4">USD {photo.price}</h3>
						<button on:click={checkout} data-price={photo.price} data-id={photo.id} class="btn"
							>Buy Now</button
						>
					</div>
				</div>
			</div>
		{/each}
	</div>
</div>

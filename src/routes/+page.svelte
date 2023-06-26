<script>
	import { uploadMedia } from '../utils/s3-uploader.js';
	import { isAuthenticated, getTokenFromLocalStorage } from '../utils/auth.js';
	import { PUBLIC_BACKEND_BASE_URL } from '$env/static/public';
	export let data;

	let formErrors = {};
	let loading=false;

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
		loading=true;
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
			location.reload()
			closeModal();
			loading=false;
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
				stroke-width="1.5">
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

					{#if loading}
					<div class ="flex justify-center shadow-l">
						<svg aria-hidden="true" class="w-8 h-8 mr-2 text-gray-200 animate-spin dark:text-gray-600 fill-blue-600" viewBox="0 0 100 101" fill="none" xmlns="http://www.w3.org/2000/svg">
							<path d="M100 50.5908C100 78.2051 77.6142 100.591 50 100.591C22.3858 100.591 0 78.2051 0 50.5908C0 22.9766 22.3858 0.59082 50 0.59082C77.6142 0.59082 100 22.9766 100 50.5908ZM9.08144 50.5908C9.08144 73.1895 27.4013 91.5094 50 91.5094C72.5987 91.5094 90.9186 73.1895 90.9186 50.5908C90.9186 27.9921 72.5987 9.67226 50 9.67226C27.4013 9.67226 9.08144 27.9921 9.08144 50.5908Z" fill="currentColor"/>
							<path d="M93.9676 39.0409C96.393 38.4038 97.8624 35.9116 97.0079 33.5539C95.2932 28.8227 92.871 24.3692 89.8167 20.348C85.8452 15.1192 80.8826 10.7238 75.2124 7.41289C69.5422 4.10194 63.2754 1.94025 56.7698 1.05124C51.7666 0.367541 46.6976 0.446843 41.7345 1.27873C39.2613 1.69328 37.813 4.19778 38.4501 6.62326C39.0873 9.04874 41.5694 10.4717 44.0505 10.1071C47.8511 9.54855 51.7191 9.52689 55.5402 10.0491C60.8642 10.7766 65.9928 12.5457 70.6331 15.2552C75.2735 17.9648 79.3347 21.5619 82.5849 25.841C84.9175 28.9121 86.7997 32.2913 88.1811 35.8758C89.083 38.2158 91.5421 39.6781 93.9676 39.0409Z" fill="currentFill"/>
						</svg>
					</div>
					{/if}
					<div class="form-control w-full mt-4">
						<button type="submit" class="btn btn-md rounded-xl" disabled={loading}>Upload</button>
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

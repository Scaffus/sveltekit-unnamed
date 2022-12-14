<script lang="ts">
	import {
		getAuth,
		createUserWithEmailAndPassword,
		signInWithEmailAndPassword,
		signInWithPopup
	} from 'firebase/auth';
	import fb from '$lib/firebase';
	import { goto } from '$app/navigation';
	import { fade } from 'svelte/transition';
	import Separator from '$lib/Separator.svelte';

	// 0 => Sign In
	// 1 => Sign Up
	let currentForm = 0;

	let emailregex = new RegExp(
		'^[a-zA-Z0-9.!#$%&’*+/=?^_`{|}~-]+@[a-zA-Z0-9-]+(?:.[a-zA-Z0-9-]+)*$'
	);
	let incorrectEmailFormat = false;
	let email = '';
	let password = '';

	let passwordInput: HTMLInputElement;
	let showPassword = false;
	const toggleShowPassword = () => {
		showPassword = !showPassword;
		if (showPassword) passwordInput.type = 'text';
		else passwordInput.type = 'password';
	};

	function signInWithGoogle() {
		const auth = getAuth();
		signInWithPopup(auth, fb.googleAuthProvider)
			.then((result) => {
				const user = result.user;
				if (user) console.info('Logged in successfully');
				goto('/');
			})
			.catch((error) => {
				console.error(error);
			});
	}

	function auth() {
		if (email == '' || password == '') return;

		if (password.length < 8) return;

		if (currentForm == 0) {
			signInWithEmailAndPassword(fb.auth(), email, password)
				.then(() => {
					goto('/');
					return 0;
				})
				.catch((e) => {
					switch (e.code) {
						case 'auth/wrong-password':
							alert('Wrong password');
					}
					console.error(e);
					return 1;
				});
		} else if (currentForm == 1) {
			createUserWithEmailAndPassword(fb.auth(), email, password)
				.then(() => {
					goto('/');
					return 0;
				})
				.catch((e) => {
					switch (e.code) {
						case 'auth/email-already-in-use':
							alert('Email already in use');
					}
					return 1;
				});
		}
	}

	function checkEmail() {
		if (email == '') {
			incorrectEmailFormat = false;
			return;
		}

		if (email.match(emailregex)) incorrectEmailFormat = false;
		else incorrectEmailFormat = true;
	}
</script>

<div in:fade={{ duration: 350 }} class="pointer-events-none flex h-full w-full">
	<div
		class="pointer-events-auto mx-auto w-fit self-center rounded-lg border border-zinc-700 bg-white px-5 py-4 shadow-lg dark:bg-zinc-900 md:w-1/3"
	>
		<div class="flex justify-between">
			<button
				class="mx-auto py-2 px-2 text-center text-xl font-bold transition duration-300 ease-in-out hover:text-black dark:hover:text-gray-300"
				on:click={() => (currentForm = 0)}
				class:opacity-50={currentForm == 1}
			>
				<h1>Sign In</h1>
				<hr class="h-px border-none bg-zinc-600 dark:bg-white" class:invisible={currentForm == 1} />
			</button>
			<button
				class="mx-auto py-2 px-2 text-center text-xl font-bold transition duration-300 ease-in-out hover:text-black dark:hover:text-gray-300"
				on:click={() => (currentForm = 1)}
				class:opacity-50={currentForm == 0}
			>
				<h1>Sign Up</h1>
				<hr class="h-px border-none bg-zinc-600 dark:bg-white" class:invisible={currentForm == 0} />
			</button>
		</div>

		<div id="inputs" class="px-2 py-3">
			<div id="email">
				<span class="text-lg"> Email </span>
				{#if incorrectEmailFormat}
					<span transition:fade={{ duration: 200 }} class="text-sm text-red-600"> Bad format </span>
				{/if}
				<input
					type="text"
					name="email"
					id="email"
					placeholder="jhondoe@mail.com"
					class="block w-full rounded-md border border-zinc-700 px-2 py-1 text-lg outline-none outline-offset-0 focus:outline-blue-500 dark:bg-zinc-800 sm:rounded-lg sm:px-3"
					bind:value={email}
					on:input={checkEmail}
				/>
			</div>
			<br />
			<div id="password">
				<span class="block text-lg"> Password </span>
				<span class="block text-zinc-500"> 8 characters minimum </span>
				<div class="flex">
					<input
						type="password"
						name="password"
						id="password"
						placeholder="••••••••"
						bind:this={passwordInput}
						class="w-full rounded-md border border-zinc-700 px-2 py-1 text-lg outline-none outline-offset-0 focus:outline-blue-500 dark:bg-zinc-800 sm:rounded-lg sm:px-3"
						bind:value={password}
					/>
					<button on:click={toggleShowPassword} class="relative right-10 z-20 w-0 p-2 pr-0">
						{#if showPassword}
							<svg
								xmlns="http://www.w3.org/2000/svg"
								fill="none"
								viewBox="0 0 24 24"
								stroke-width="1.5"
								stroke="currentColor"
								class="h-6 w-6 transition"
							>
								<path
									stroke-linecap="round"
									stroke-linejoin="round"
									d="M3.98 8.223A10.477 10.477 0 001.934 12C3.226 16.338 7.244 19.5 12 19.5c.993 0 1.953-.138 2.863-.395M6.228 6.228A10.45 10.45 0 0112 4.5c4.756 0 8.773 3.162 10.065 7.498a10.523 10.523 0 01-4.293 5.774M6.228 6.228L3 3m3.228 3.228l3.65 3.65m7.894 7.894L21 21m-3.228-3.228l-3.65-3.65m0 0a3 3 0 10-4.243-4.243m4.242 4.242L9.88 9.88"
								/>
							</svg>
						{:else}
							<svg
								xmlns="http://www.w3.org/2000/svg"
								fill="none"
								viewBox="0 0 24 24"
								stroke-width="1.5"
								stroke="currentColor"
								class="h-6 w-6 transition"
							>
								<path
									stroke-linecap="round"
									stroke-linejoin="round"
									d="M2.036 12.322a1.012 1.012 0 010-.639C3.423 7.51 7.36 4.5 12 4.5c4.638 0 8.573 3.007 9.963 7.178.07.207.07.431 0 .639C20.577 16.49 16.64 19.5 12 19.5c-4.638 0-8.573-3.007-9.963-7.178z"
								/>
								<path
									stroke-linecap="round"
									stroke-linejoin="round"
									d="M15 12a3 3 0 11-6 0 3 3 0 016 0z"
								/>
							</svg>
						{/if}
					</button>
				</div>
			</div>
		</div>

		<button
			class="my-2 mx-auto block rounded-md border border-zinc-700 px-3 py-2 hover:bg-zinc-200 dark:hover:bg-zinc-800 sm:rounded-lg"
			on:click={auth}
		>
			Sign {currentForm == 0 ? 'In' : 'Up'}
		</button>
		<Separator />
		<button
			class="mx-auto mt-3 block rounded-md border border-zinc-700 px-3 py-2 hover:bg-zinc-200 dark:hover:bg-zinc-800 sm:rounded-lg"
			on:click={signInWithGoogle}
		>
			Sign In with Google</button
		>
	</div>
</div>

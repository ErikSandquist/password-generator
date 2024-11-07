<script>
	import { Button } from '$lib/components/ui/button';
	import {
		Card,
		CardContent,
		CardDescription,
		CardFooter,
		CardHeader,
		CardTitle
	} from '$lib/components/ui/card';
	import { Checkbox } from '$lib/components/ui/checkbox';
	import { Input } from '$lib/components/ui/input';
	import { Label } from '$lib/components/ui/label';
	import * as Slider from '$lib/components/ui/slider';
	import { Progress } from '$lib/components/ui/progress';
	import { toast } from 'svelte-sonner';
	import { Copy } from 'lucide-svelte';
	import { fade } from 'svelte/transition';

	let password = '';
	let length = 12;
	let includeUppercase = true;
	let includeLowercase = true;
	let includeNumbers = true;
	let includeSpecialChars = true;
	let strengthScore = 0;

	function getSecureRandomNumber(max) {
		const randomBuffer = new Uint8Array(1);
		window.crypto.getRandomValues(randomBuffer);
		return randomBuffer[0] % max;
	}

	function calculatePasswordStrength(pwd) {
		if (!pwd) return 0;

		let score = 0;

		// Length checks (max 3 points)
		if (pwd.length >= 8) score += 1;
		if (pwd.length >= 14) score += 1; // Increased from 12
		if (pwd.length >= 20) score += 1; // Increased from 16

		// Character variety checks (max 4 points)
		const hasUppercase = /[A-Z]/.test(pwd);
		const hasLowercase = /[a-z]/.test(pwd);
		const hasNumbers = /[0-9]/.test(pwd);
		const hasSpecial = /[^A-Za-z0-9]/.test(pwd);

		if (hasUppercase) score += 1;
		if (hasLowercase) score += 1;
		if (hasNumbers) score += 1;
		if (hasSpecial) score += 1;

		// Additional complexity checks (max 3 points)
		if (hasUppercase && hasLowercase && hasNumbers && hasSpecial) score += 1; // All char types
		if (/(\d.*[A-Za-z])|([A-Za-z].*\d)/.test(pwd)) score += 1; // Mix of letters and numbers
		if (pwd.length >= 12 && hasSpecial && hasNumbers) score += 1; // Long with special chars and numbers

		// Calculate percentage (max score is 10)
		return Math.min(100, Math.round((score / 10) * 100));
	}

	function getStrengthText(score) {
		if (score < 30) return 'Very Weak';
		if (score < 50) return 'Weak';
		if (score < 70) return 'Medium';
		if (score < 90) return 'Strong';
		return 'Very Strong';
	}

	function getStrengthColor(score) {
		if (score < 30) return 'bg-red-500';
		if (score < 50) return 'bg-orange-500';
		if (score < 70) return 'bg-yellow-500';
		if (score < 90) return 'bg-green-500';
		return 'bg-emerald-500';
	}

	function generatePassword() {
		let charset = '';
		if (includeUppercase) charset += 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
		if (includeLowercase) charset += 'abcdefghijklmnopqrstuvwxyz';
		if (includeNumbers) charset += '0123456789';
		if (includeSpecialChars) charset += '!@#$%^&*()_+{}[]|:;<>,.?/~';

		if (charset === '') {
			toast.error('Error', {
				description: 'Please select at least one character type'
			});
			return;
		}

		let result = '';
		for (let i = 0; i < length; i++) {
			result += charset[getSecureRandomNumber(charset.length)];
		}

		password = result;
		strengthScore = calculatePasswordStrength(password);
	}

	async function copyToClipboard() {
		await navigator.clipboard.writeText(password);
		toast.success('Copied!', {
			description: 'Password copied to clipboard'
		});
	}
</script>

<main
	class="flex h-screen items-center justify-center bg-cover bg-center bg-no-repeat"
	style="background-image: url(/images/bg.avif);"
>
	<Card
		class="mx-auto w-full max-w-md rounded-xl border border-white/20 bg-white/[0.07] p-6 px-3 backdrop-blur-lg"
	>
		<CardHeader>
			<CardTitle>Password Generator</CardTitle>
			<CardDescription>Customize your password settings below.</CardDescription>
		</CardHeader>

		<CardContent class="space-y-4 transition-all">
			<div class="space-y-2">
				<Label for="length">Password Length: {length}</Label>
				<Slider.Root
					value={[length]}
					onValueChange={(value) => (length = value[0])}
					max={32}
					min={8}
					step={1}
					class="relative flex w-full touch-none select-none items-center"
				>
					<Slider.Track
						class="bg-primary/20 relative h-1.5 w-full grow overflow-hidden rounded-full"
					>
						<Slider.Range class="bg-primary absolute h-full" />
					</Slider.Track>
					<Slider.Thumb
						class="border-primary/50 bg-background focus-visible:ring-ring block h-4 w-4 rounded-full border shadow transition-colors focus-visible:outline-none focus-visible:ring-1 disabled:pointer-events-none disabled:opacity-50"
					/>
				</Slider.Root>
			</div>

			<div class="space-y-2">
				<Label>Include:</Label>
				<div class="flex flex-col space-y-2">
					<div class="flex items-center space-x-2">
						<Checkbox id="uppercase" bind:checked={includeUppercase} />
						<Label for="uppercase">Uppercase Letters</Label>
					</div>
					<div class="flex items-center space-x-2">
						<Checkbox id="lowercase" bind:checked={includeLowercase} />
						<Label for="lowercase">Lowercase Letters</Label>
					</div>
					<div class="flex items-center space-x-2">
						<Checkbox id="numbers" bind:checked={includeNumbers} />
						<Label for="numbers">Numbers</Label>
					</div>
					<div class="flex items-center space-x-2">
						<Checkbox id="special" bind:checked={includeSpecialChars} />
						<Label for="special">Special Characters</Label>
					</div>
				</div>
			</div>

			<Button on:click={generatePassword} class="w-full">Generate Password</Button>

			{#if password}
				<div class="mt-4 space-y-2" transition:fade>
					<Label for="generated-password">Generated Password:</Label>
					<div class="flex space-x-2">
						<Input id="generated-password" value={password} readonly class="flex-grow" />
						<Button
							variant="outline"
							size="icon"
							on:click={copyToClipboard}
							aria-label="Copy password to clipboard"
						>
							<Copy class="h-4 w-4" />
						</Button>
					</div>

					<div class="space-y-2">
						<div class="flex justify-between text-sm">
							<span>Password Strength:</span>
							<span>{getStrengthText(strengthScore)}</span>
						</div>
						<Progress
							value={strengthScore}
							class={`relative h-4 w-full overflow-hidden rounded-full bg-opacity-20 ${getStrengthColor(strengthScore)}`}
						/>
					</div>
				</div>
			{/if}
		</CardContent>

		<CardFooter class="text-muted-foreground text-sm">
			Your password is generated locally and never stored or transmitted.
		</CardFooter>
	</Card>
</main>

<style>
	:global(.bg-red-500) {
		background-color: rgb(239 68 68 / 0.2) !important;
	}
	:global(.bg-red-500 div) {
		background-color: rgb(239 68 68) !important;
	}

	:global(.bg-orange-500) {
		background-color: rgb(249 115 22 / 0.2) !important;
	}
	:global(.bg-orange-500 div) {
		background-color: rgb(249 115 22) !important;
	}

	:global(.bg-yellow-500) {
		background-color: rgb(234 179 8 / 0.2) !important;
	}
	:global(.bg-yellow-500 div) {
		background-color: rgb(234 179 8) !important;
	}

	:global(.bg-green-500) {
		background-color: rgb(34 197 94 / 0.2) !important;
	}
	:global(.bg-green-500 div) {
		background-color: rgb(34 197 94) !important;
	}

	:global(.bg-emerald-500) {
		background-color: rgb(16 185 129 / 0.2) !important;
	}
	:global(.bg-emerald-500 div) {
		background-color: rgb(16 185 129) !important;
	}
</style>

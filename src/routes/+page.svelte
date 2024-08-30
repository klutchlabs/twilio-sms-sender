<script lang="ts">
	import { browser } from '$app/environment'

	let isLoading = false
	const form = {
		accountSid: '',
		keySid: '',
		keySecret: '',
		to: '',
		from: '',
		message: '',
		mediaUrl: ''
	}

	$: if (browser) {
		form.accountSid = localStorage.getItem('accountSid') ?? ''
		form.keySid = localStorage.getItem('keySid') ?? ''
		form.keySecret = localStorage.getItem('keySecret') ?? ''
		form.to = localStorage.getItem('to') ?? ''
		form.from = localStorage.getItem('from') ?? ''
		form.message = localStorage.getItem('message') ?? ''
		form.mediaUrl = localStorage.getItem('mediaUrl') ?? ''
	}

	async function send(): Promise<void> {
		isLoading = true

		try {
			if (browser) {
				localStorage.setItem('accountSid', form.accountSid)
				localStorage.setItem('keySid', form.keySid)
				localStorage.setItem('keySecret', form.keySecret)
				localStorage.setItem('to', form.to)
				localStorage.setItem('from', form.from)
				localStorage.setItem('message', form.message)
				localStorage.setItem('mediaUrl', form.mediaUrl)
			}

			const data = new FormData()
			data.append('From', form.from)
			data.append('To', form.to)

			if (form.message) {
				data.append('Body', form.message)
			}

			if (form.mediaUrl) {
				data.append('MediaUrl', form.mediaUrl)
			}

			const resp = await fetch(
				`https://api.twilio.com/2010-04-01/Accounts/${form.accountSid}/Messages.json`,
				{
					method: 'POST',
					headers: {
						'Authorization': `Basic ${btoa(`${form.keySid}: ${form.keySecret}`)}`
					},
					body: data
				}
			)

			const json = await resp.json()
			console.log(json)

			if (!json.error_message) {
				alert('Sent')
			} else {
				alert(`Error: ${json.error_message}\n\nPlease see console.`)
			}
		} finally {
			isLoading = false
		}
	}
</script>

<div class="p-2 flex flex-col gap-4">
	<div class="flex gap-4">
		<span>Twilio Account SID:</span>
		<input type="text" class="rounded border border-gray-600 px-2 py-1 w-[350px]" bind:value={form.accountSid} />
	</div>
	<div class="flex gap-4">
		<span>Twilio Key SID:</span>
		<input type="text" class="rounded border border-gray-600 px-2 py-1 w-[350px]" bind:value={form.keySid} />
	</div>
	<div class="flex gap-4">
		<span>Twilio Key Secret:</span>
		<input type="text" class="rounded border border-gray-600 px-2 py-1 w-[350px]" bind:value={form.keySecret} />
	</div>
	<div class="flex gap-4">
		<span>From:</span>
		<input type="text" class="rounded border border-gray-600 px-2 py-1" bind:value={form.from} />
	</div>
	<div class="flex gap-4">
		<span>To:</span>
		<input type="text" class="rounded border border-gray-600 px-2 py-1" bind:value={form.to} />
	</div>
	<div class="flex gap-4">
		<span>Message:</span>
		<textarea class="rounded border border-gray-600 w-[300px] h-[80px] px-2 py-1" bind:value={form.message}></textarea>
	</div>
	<div class="flex gap-4">
		<span>Media URL:</span>
		<textarea class="rounded border border-gray-600 w-[300px] h-[80px] px-2 py-1" bind:value={form.mediaUrl}></textarea>
	</div>
	<div>
		<button 
			class="rounded bg-blue-500 border border-blue-800 cursor-pointer px-2 py-1 text-white" 
			disabled={isLoading}
			on:click={() => { void send() }}
		>
		  {#if isLoading} 
			  Loading...
			{:else}
				Send
			{/if}
		</button>
	</div>
</div>

<style>
</style>

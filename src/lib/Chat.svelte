<script lang="ts">
    // Import necessary modules and components
    import { afterUpdate, onMount } from 'svelte';
    import { ProgressRadial } from '@skeletonlabs/skeleton';
    import snarkdown from 'snarkdown';
    import { afterNavigate } from '$app/navigation';
    import type { Chat } from '$misc/shared';
    import { chatStore, isLoadingAnswerStore, liveAnswerStore } from '$misc/stores';
    import ChatMessages from './ChatMessages.svelte';

    // Exported props
    export let slug: string; 						// Slug for identifying specific chat
    export let chat: Chat | undefined = undefined; 			// Chat object, initially undefined

    // Reactive statement to update chat based on slug
    $: if ($chatStore[slug]) {
        chat = $chatStore[slug];
    }

    // Autoscroll setup
    let div: HTMLElement | null | undefined; // Setup for auto scroll

    // Mount lifecycle hook: called when component is mounted to DOM
    onMount(() => {
        // Get a reference to scrollable element with id 'page'
        div = document.getElementById('page');
    });

    // Function to scroll to bottom of chat messages
    function scrollToBottom() {
        setTimeout(() => {
            // Scroll div element to the bottom with smooth behavior
            div?.scrollTo({ top: div.scrollHeight + 999, behavior: 'smooth' });
        }, 50);
    }

    // After update lifecycle hook: called after component updates
    afterUpdate(() => {
        scrollToBottom(); // Scroll to bottom of chat messages
    });

    // After navigate hook: called after navigation events
    afterNavigate(() => {
        scrollToBottom(); // Scroll to bottom of chat messages after navigation
    });
</script>

{#if chat} 	// checks if chat is fine and wokring
	<div
		class="flex flex-col container justify-end h-full mx-auto px-4 md:px-8 gap-6" <!-- Flexbox layout in column form, container meaning fixed width, alligns at the bottom, full height of parent, centres, pads gaps -->
	>
		<slot name="additional-content-top" />  <!-- allows external entries -->

		{#if chat.messages.length > 0 || $isLoadingAnswerStore} <!-- length is longer than 0 and exists -->
			<div class="flex flex-col max-w-4xl md:mx-auto space-y-6"> <!-- Just designs -->
				<!-- Message history --> 
				<ChatMessages {slug} siblings={chat.messages} on:editMessage /> <!-- Sets chat.messages as sibling function to ChatMessages meaning it anytime one changes the other does too -->

				<!-- Live Message -->
				{#if $isLoadingAnswerStore}
					<div class="place-self-start">
						<div class="p-5 rounded-2xl variant-ghost-tertiary rounded-tl-none">
							{@html snarkdown($liveAnswerStore.content)} <!-- renders HTML content directly -->
						</div>
					</div>
				{/if}
			</div>
		{/if}

		<slot name="additional-content-bottom" /> <!-- another slot to put stuff in -->

		<!-- Progress indicator -->
		{#if $isLoadingAnswerStore} 
			<div class="animate-pulse md:w-12 self-center py-2 md:py-6"> <!-- pulse effect to make it look smooth & other effects -->
				<ProgressRadial
					class="w-8"
					stroke={120}
					meter="stroke-tertiary-500"
					track="stroke-tertiary-500/30"
				/>
			</div>
		{/if}
	</div>
{/if}

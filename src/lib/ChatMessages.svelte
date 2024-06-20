<script lang="ts">
	// Imports modules
	import type { ChatMessage as ChatMessageModel } from '$misc/shared';
	import { TabGroup, Tab } from '@skeletonlabs/skeleton';
	import { chatStore } from '$misc/stores';
	import ChatMessage from './ChatMessage.svelte';

	// Exports components for external use
	export let slug: string;
	export let siblings: ChatMessageModel[];

	// Intializes tabSet as a sibling component to getActiveIndex
	let tabSet = getActiveIndex(siblings);

	// Determines the active tab index based on whether a message is selected
	function getActiveIndex(messages: ChatMessageModel[]) {
		let result = 0;
		if (messages.length > 1) {
			result = messages.findIndex((message) => message.isSelected);
		}

		// fallback to the first item if none is selected (shouldn't happen)
		return Math.max(result, 0);
	}
	// Handles Tab changes based off and changes the sibling chat messages based on the selected Tab
	function handleChangeTab(e: any): void {
		const id = siblings[e.target?.value].id!;
		chatStore.selectSibling(slug, id);
	}
</script>

{#if siblings.length === 1}
	<!-- This TypeScript error is nonsense... -->
	<ChatMessage {slug} message={siblings[0]} renderChildren on:editMessage />
{:else}
	<!-- siblings are modified outside, so we need to trigger an update  -->
	{#key siblings}
		<TabGroup regionPanel="flex flex-col space-y-4">
			{#each siblings as sibling, index}
				<Tab
					bind:group={tabSet}
					name={sibling.id || 'tab' + index}
					value={index}
					on:change={handleChangeTab}
				>
					{index + 1}
				</Tab>
			{/each}
			<!-- Tab Panels --->
			<svelte:fragment slot="panel">
				{#each siblings as sibling, index}
					{#if tabSet === index}
						<ChatMessage {slug} message={sibling} renderChildren on:editMessage />
					{/if}
				{/each}
			</svelte:fragment>
		</TabGroup>
	{/key}
{/if}

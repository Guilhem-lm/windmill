<script lang="ts">
	import type { Schema } from '$lib/common'
	import { twMerge } from 'tailwind-merge'
	import EditableSchemaForm from '../EditableSchemaForm.svelte'
	import AddProperty from './AddProperty.svelte'
	import { createEventDispatcher } from 'svelte'

	export let schema: Schema | undefined | any
	export let offset: number = 0
	export let uiOnly: boolean = false
	export let noPreview: boolean = false
	export let fullHeight: boolean = true
	export let lightweightMode: boolean = false

	let addProperty: AddProperty | undefined = undefined

	const dispatch = createEventDispatcher()
</script>

<div class={twMerge(fullHeight ? 'h-full' : 'h-80', 'border overflow-y-auto rounded-md')}>
	<div class="p-4 border-b">
		<AddProperty on:change={() => dispatch('change', schema)} bind:schema bind:this={addProperty} />
	</div>
	<EditableSchemaForm
		bind:schema
		on:change={() => dispatch('change', schema)}
		isFlowInput
		on:edit={(e) => {
			addProperty?.openDrawer(e.detail)
		}}
		on:delete={(e) => {
			addProperty?.handleDeleteArgument([e.detail])
		}}
		{offset}
		{uiOnly}
		{noPreview}
		{lightweightMode}
	/>
</div>

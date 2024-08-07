<script lang="ts">
	import AppRow from '../common/table/AppRow.svelte'
	import FlowRow from '../common/table/FlowRow.svelte'
	import RawAppRow from '../common/table/RawAppRow.svelte'
	import ScriptRow from '../common/table/ScriptRow.svelte'
	import ConfirmationModal from '../common/confirmationModal/ConfirmationModal.svelte'
	import { Alert } from '$lib/components/common'
	import DeployWorkspaceDrawer from '../DeployWorkspaceDrawer.svelte'
	import MoveDrawer from '../MoveDrawer.svelte'
	import ShareModal from '../ShareModal.svelte'
	import { createEventDispatcher } from 'svelte'
	import { ArrowBigUp } from 'lucide-svelte'
	import { enterpriseLicense, workspaceStore } from '$lib/stores'
	import { WorkspaceService, type WorkspaceDeployUISettings } from '$lib/gen'
	import { ALL_DEPLOYABLE } from '$lib/utils'

	export let item
	export let depth: number = 0

	const dispatch = createEventDispatcher()

	let deleteConfirmedCallback: (() => void) | undefined = undefined
	let shareModal: ShareModal
	let moveDrawer: MoveDrawer
	let deploymentDrawer: DeployWorkspaceDrawer

	let menuOpen: boolean = false
	export let showCode: (path: string, summary: string) => void

	let deployUiSettings: WorkspaceDeployUISettings | undefined = undefined

	async function getDeployUiSettings() {
		if (!$enterpriseLicense) {
			deployUiSettings = ALL_DEPLOYABLE
			return
		}
		let settings = await WorkspaceService.getSettings({ workspace: $workspaceStore! })
		deployUiSettings = settings.deploy_ui ?? ALL_DEPLOYABLE
	}
	getDeployUiSettings()

</script>

{#if item.type == 'script'}
	<ScriptRow
		bind:deleteConfirmedCallback
		starred={item.starred ?? false}
		marked={item.marked}
		on:change={() => dispatch('scriptChanged')}
		script={item}
		errorHandlerMuted={item.ws_error_handler_muted === undefined ||
		item.ws_error_handler_muted === null
			? false
			: item.ws_error_handler_muted}
		{shareModal}
		{moveDrawer}
		{deploymentDrawer}
		{depth}
		bind:menuOpen
		{showCode}
		{deployUiSettings}
	/>
{:else if item.type == 'flow'}
	<FlowRow
		bind:deleteConfirmedCallback
		starred={item.starred ?? false}
		marked={item.marked}
		on:change={() => dispatch('flowChanged')}
		flow={item}
		errorHandlerMuted={item.ws_error_handler_muted === undefined ||
		item.ws_error_handler_muted === null
			? false
			: item.ws_error_handler_muted}
		{shareModal}
		{moveDrawer}
		{deploymentDrawer}
		{depth}
		bind:menuOpen
		{deployUiSettings}
	/>
{:else if item.type == 'app'}
	<AppRow
		bind:deleteConfirmedCallback
		starred={item.starred ?? false}
		marked={item.marked}
		on:change={() => dispatch('appChanged')}
		app={item}
		{moveDrawer}
		{shareModal}
		{deploymentDrawer}
		{depth}
		bind:menuOpen
		{deployUiSettings}
	/>
{:else if item.type == 'raw_app'}
	<RawAppRow
		bind:deleteConfirmedCallback
		starred={item.starred ?? false}
		marked={item.marked}
		on:change={() => dispatch('rawAppChanged')}
		app={item}
		{moveDrawer}
		{shareModal}
		{deploymentDrawer}
		{depth}
		bind:menuOpen
		{deployUiSettings}
	/>
{/if}

{#if menuOpen}
	<ConfirmationModal
		open={Boolean(deleteConfirmedCallback)}
		title="Remove"
		confirmationText="Remove"
		on:canceled={() => {
			deleteConfirmedCallback = undefined
		}}
		on:confirmed={() => {
			if (deleteConfirmedCallback) {
				deleteConfirmedCallback()
			}
			deleteConfirmedCallback = undefined
		}}
	>
		<div class="flex flex-col w-full space-y-4">
			<span>Are you sure you want to remove it?</span>
			<Alert type="info" title="Bypass confirmation">
				<div>
					You can press
					<span
						class="inline-flex border rounded-md p-1 bg-blue-100 border-blue-200 dark:bg-blue-800 dark:border-blue-900 text-xs"
					>
						<ArrowBigUp size={18} />
					</span>
					while removing to bypass confirmation.
				</div>
			</Alert>
		</div>
	</ConfirmationModal>

	<ShareModal
		bind:this={shareModal}
		on:change={() => {
			dispatch('reload')
		}}
	/>

	<DeployWorkspaceDrawer bind:this={deploymentDrawer} />
	<MoveDrawer
		bind:this={moveDrawer}
		on:update={() => {
			dispatch('reload')
		}}
	/>
{/if}

<script lang="ts">
	import { createEventDispatcher, getContext } from 'svelte'
	import type { App, AppViewerContext } from '../types'
	import { BG_PREFIX, allItems } from '../utils'
	import { findComponentSettings, findGridItem } from './appUtils'
	import PanelSection from './settingsPanel/common/PanelSection.svelte'
	import ComponentPanel from './settingsPanel/ComponentPanel.svelte'
	import InputsSpecsEditor from './settingsPanel/InputsSpecsEditor.svelte'
	import BackgroundScriptSettings from './settingsPanel/script/BackgroundScriptSettings.svelte'
	import EventHandlerItem from './settingsPanel/EventHandlerItem.svelte'

	const { selectedComponent, app, stateId, runnableComponents } =
		getContext<AppViewerContext>('AppViewerContext')

	let firstComponent = $selectedComponent?.[0]

	$: $selectedComponent?.[0] != firstComponent && (firstComponent = $selectedComponent?.[0])

	$: hiddenInlineScript = $app?.hiddenInlineScripts
		?.map((x, i) => ({ script: x, index: i }))
		.find(({ script, index }) => $selectedComponent?.includes(BG_PREFIX + index))

	$: componentSettings = findComponentSettings($app, firstComponent)
	$: tableActionSettings = findTableActionSettings($app, firstComponent)
	$: menuItemsSettings = findMenuItemsSettings($app, firstComponent)

	function findTableActionSettings(app: App, id: string | undefined) {
		return allItems(app.grid, app.subgrids)
			.map((x) => {
				if (x?.data?.type === 'tablecomponent') {
					if (x?.data?.actionButtons) {
						const tableAction = x.data.actionButtons.find((x) => x.id === id)
						if (tableAction) {
							return { item: { data: tableAction, id: tableAction.id }, parent: x.data.id }
						}
					}
				} else if (
					x?.data?.type === 'aggridcomponent' ||
					x?.data?.type === 'aggridcomponentee' ||
					x?.data?.type === 'dbexplorercomponent' ||
					x?.data?.type === 'aggridinfinitecomponent' ||
					x?.data?.type === 'aggridinfinitecomponentee'
				) {
					if (x?.data?.actions) {
						const tableAction = x.data.actions.find((x) => x.id === id)
						if (tableAction) {
							return { item: { data: tableAction, id: tableAction.id }, parent: x.data.id }
						}
					}
				}
			})
			.find((x) => x)
	}

	function findMenuItemsSettings(app: App, id: string | undefined) {
		return allItems(app.grid, app.subgrids)
			.map((x) => {
				if (x?.data?.type === 'menucomponent') {
					if (x?.data?.menuItems) {
						const menuItem = x.data.menuItems.find((x) => x.id === id)
						if (menuItem) {
							return { item: { data: menuItem, id: menuItem.id }, parent: x.data.id }
						}
					}
				}
			})
			.find((x) => x)
	}

	const dispatch = createEventDispatcher()
</script>

{#if componentSettings}
	{#key componentSettings?.item?.id}
		<ComponentPanel
			bind:componentSettings
			onDelete={() => {
				dispatch('delete')
			}}
		/>
	{/key}
{:else if tableActionSettings}
	{#key tableActionSettings?.item?.data?.id}
		<ComponentPanel
			noGrid
			bind:componentSettings={tableActionSettings}
			duplicateMoveAllowed={false}
			onDelete={() => {
				if (tableActionSettings) {
					const parent = findGridItem($app, tableActionSettings.parent)
					if (!parent) return

					if (parent.data.type === 'tablecomponent') {
						parent.data.actionButtons = parent.data.actionButtons.filter(
							(x) => x.id !== tableActionSettings?.item.id
						)
					}

					if (
						(parent.data.type === 'aggridcomponent' ||
							parent.data.type === 'aggridcomponentee' ||
							parent.data.type === 'dbexplorercomponent' ||
							parent.data.type === 'aggridinfinitecomponent' ||
							parent.data.type === 'aggridinfinitecomponentee') &&
						Array.isArray(parent.data.actions)
					) {
						parent.data.actions = parent.data.actions.filter(
							(x) => x.id !== tableActionSettings?.item.id
						)
					}
				}
			}}
		/>
	{/key}
{:else if menuItemsSettings}
	{#key menuItemsSettings?.item?.id}
		<ComponentPanel
			noGrid
			bind:componentSettings={menuItemsSettings}
			duplicateMoveAllowed={false}
			onDelete={() => {
				if (menuItemsSettings) {
					const parent = findGridItem($app, menuItemsSettings.parent)
					if (!parent) return

					if (parent.data.type === 'menucomponent') {
						parent.data.menuItems = parent.data.menuItems.filter(
							(x) => x.id !== menuItemsSettings?.item.id
						)
					}
				}
			}}
		/>
	{/key}
{:else if hiddenInlineScript}
	{@const id = BG_PREFIX + hiddenInlineScript.index}
	{#key id}
		<BackgroundScriptSettings bind:runnable={hiddenInlineScript.script} {id} />

		{#if Object.keys(hiddenInlineScript.script.fields ?? {}).length > 0}
			<div class="mb-8">
				<PanelSection title={`Inputs`}>
					{#key $stateId}
						<InputsSpecsEditor
							displayType
							{id}
							shouldCapitalize={false}
							bind:inputSpecs={hiddenInlineScript.script.fields}
							userInputEnabled={false}
							recomputeOnInputChanged={hiddenInlineScript.script.recomputeOnInputChanged}
							showOnDemandOnlyToggle
						/>
					{/key}
				</PanelSection>
			</div>
		{/if}
		<PanelSection
			title={`Event handlers`}
			fullHeight={false}
			tooltip="Event handlers are used to trigger actions on other components when a specific event occurs. For example, you can trigger a recompute on a component when a script has successfully run."
		>
			<EventHandlerItem
				title="on success"
				tooltip="This event is triggered when the script runs successfully."
				items={Object.keys($runnableComponents).filter((_id) => _id !== id)}
				bind:value={hiddenInlineScript.script.recomputeIds}
			/>
		</PanelSection>
		<div class="grow shrink" />
	{/key}
{/if}

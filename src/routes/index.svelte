<script>
	import Workspace from '../components/Workspace.svelte';
	import Panel from '../components/Panel.svelte';
	import Preview from '../components/Preview.svelte';
</script>

<div id="Designer">
	<div id="PixelPanel" class="flex-row top-panels">
		<div class="work-space">
			<Panel>
				<Workspace />
			</Panel>
		</div>
		<div class="preview">
			<Panel>
				<Preview />
			</Panel>
		</div>
	</div>
	<div id="PixelPanel" class="flex-row bottom-panels">
		<Panel>Components</Panel>
		<Panel>Patterns</Panel>
		<Panel>Misc</Panel>
	</div>
</div>

<style lang="scss">
	#Designer {
		padding: 2px;
		position: relative;
		height: 100vh;
		width: 100vw;
		background-color: $darker-grey;

		.flex-row {
			position: relative;
			display: flex;
			flex-direction: row;
		}

		.work-space {
			display: flex;
			flex: 1;
		}

		.preview {
			display: flex;
			flex: 1;
		}

		.top-panels {
			height: 90vh;
		}

		.bottom-panels {
			height: 10vh;
		}
	}
</style>

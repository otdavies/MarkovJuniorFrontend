<div class="box-outer">
	<div class="box-inner">
		<slot />
	</div>
</div>

<style lang="scss" scoped>
	.box-outer {
		margin: 1px;
		position: relative;
		width: 100%;
		min-width: 32px;
		background-color: transparent;
		display: flex;

		.box-inner {
			margin-bottom: 4px;
			margin-right: 4px;
			padding: 2px;
			background-color: $light-grey;
			box-shadow: $box-shadow;
			width: 100%;
		}
	}
</style>

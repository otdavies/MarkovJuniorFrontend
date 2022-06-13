<script>
	export /**@type {String}*/
	let color;
	export /**@type {String}*/
	let size;
</script>

<div
	class="color-frame"
	style:background-color={color}
	style:width={size}
	style:padding-top={size}
/>

<style lang="scss" scoped>
	.color-frame {
		height: 0px;
		border: 1px solid $darkest-grey;
	}

	.color-frame:hover {
		border: 1px solid $flare-aqua;
		box-shadow: 0 0 5px $flare-aqua;
	}
</style>

<template>
	<div class="flex1">
		<xspaceNoteComponent />
	</div>
</template>
<script lang="ts">
export default async function () {
	return defineComponent({
		inject: ["inject_project"],
		components: {
			xspaceNoteComponent: () => _.$importVue("@/views/Note/Note.vue")
		},
		data() {
			return {
				title: ""
			};
		}
	});
}
</script>
<style lang="less"></style>

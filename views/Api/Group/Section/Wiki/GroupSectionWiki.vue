<style lang="less"></style>
<template>
	<xBlock
		v-if="isShow"
		class="flex1 mt"
		:bodyClass="{ height100: true, flex: 1 }"
		:style="{ height: '100%' }">
		<xspaceNoteComponent />
	</xBlock>
</template>
<script lang="ts">
export default async function () {
	const { THIS_FILE_URL } = this;
	return defineComponent({
		components: {
			xspaceNoteComponent: () => _.$importVue("@/views/Note/Note.vue")
		},
		data() {
			return { THIS_FILE_URL };
		},
		computed: {
			isShow() {
				return this.$route.query.group_view_tab_name === Vue._xspace_var.TAB_KEY_GROUP_WIKI;
			}
		}
	});
}
</script>

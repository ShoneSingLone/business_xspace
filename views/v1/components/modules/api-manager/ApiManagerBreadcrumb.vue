<script lang="ts">
export default async function () {
	return {
		props: {
			path: {
				type: String,
				default: ""
			}
		},
		computed: {
			pathParts() {
				return this.path ? this.path.split("/").filter(Boolean) : [];
			}
		},
		methods: {
			navigateTo(index) {
				const selectedPath = this.pathParts.slice(0, index + 1).join("/");
				this.$emit("navigateTo", "/" + selectedPath);
			}
		}
	};
}
</script>

<template>
	<div class="api-manager__breadcrumb">
		<div v-for="(part, index) in pathParts" :key="index">
			<span 
				class="api-manager__breadcrumb-part" 
				:class="{ 'api-manager__breadcrumb-part--current': index === pathParts.length - 1 }"
				@click="navigateTo(index)">
				{{ part }}
			</span>
			<span 
				v-if="index < pathParts.length - 1" 
				class="api-manager__breadcrumb-sep">/</span>
		</div>
	</div>
</template>
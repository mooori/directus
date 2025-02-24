<template>
	<div class="module-bar">
		<module-bar-logo />
		<div class="modules">
			<v-button
				v-for="module in internalModules"
				:key="module.id"
				v-tooltip.right="module.name"
				icon
				x-large
				:to="module.to"
				:href="module.href"
				tile
				:style="
					module.color
						? {
								'--v-button-color-active': module.color,
						  }
						: null
				"
			>
				<v-icon :name="module.icon" outline />
			</v-button>
		</div>
		<module-bar-avatar />
	</div>
</template>

<script lang="ts">
import { defineComponent, computed, unref } from 'vue';

import { getModules } from '@/modules/';
import ModuleBarLogo from '../module-bar-logo/';
import ModuleBarAvatar from '../module-bar-avatar/';
import { useUserStore } from '@/stores/';
import { orderBy } from 'lodash';
import { ModuleConfig } from '@directus/shared/types';

export default defineComponent({
	components: {
		ModuleBarLogo,
		ModuleBarAvatar,
	},
	setup() {
		const userStore = useUserStore();
		const { modules } = getModules();

		const internalModules = computed(() => {
			const customModuleListing = userStore.currentUser?.role.module_list;

			const registeredModules = orderBy(
				modules.value
					.map((module: ModuleConfig) => ({
						...module,
						href: module.link,
						to: module.link === undefined ? `/${module.id}` : '',
					}))
					.filter((module: ModuleConfig) => {
						if (module.hidden !== undefined && unref(module.hidden) === true) {
							return false;
						}
						return true;
					}),
				['order'],
				['asc']
			);

			if (customModuleListing && Array.isArray(customModuleListing) && customModuleListing.length > 0) {
				return [
					...customModuleListing.map((custom) => {
						if (custom.link?.startsWith('http') || custom.link?.startsWith('//')) {
							return {
								...custom,
								href: custom.link,
							};
						} else {
							return {
								...custom,
								to: custom.link,
							};
						}
					}),
					...registeredModules.filter((module) => module.persistent === true),
				];
			}
			return registeredModules;
		});
		return { internalModules, modules };
	},
});
</script>

<style>
body {
	--module-background: #18222f;
	--module-background-alt: var(--background-normal);
	--module-icon: #8196b1;
	--module-icon-alt: var(--foreground-normal-alt);
}
</style>

<style lang="scss" scoped>
.module-bar {
	display: flex;
	flex-direction: column;
	width: 64px;
	height: 100%;
	background-color: var(--module-background);

	.modules {
		flex-grow: 1;
		overflow-x: hidden;
		overflow-y: auto;
	}

	.v-button {
		--v-button-color: var(--module-icon);
		--v-button-color-hover: var(--white);
		--v-button-color-active: var(--module-icon-alt);
		--v-button-background-color: var(--module-background);
		--v-button-background-color-hover: var(--module-background);
		--v-button-background-color-active: var(--module-background-alt);
	}
}
</style>

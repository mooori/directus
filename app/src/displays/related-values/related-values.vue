<template>
	<value-null v-if="!relatedCollection" />
	<v-menu
		v-else-if="['o2m', 'm2m', 'm2a', 'translations', 'files'].includes(type.toLowerCase())"
		show-arrow
		:disabled="value.length === 0"
	>
		<template #activator="{ toggle }">
			<span class="toggle" :class="{ subdued: value.length === 0 }" @click.stop="toggle">
				<span class="label">
					{{ value.length }}
					<template v-if="value.length >= 100">+</template>
					{{ unit }}
				</span>
			</span>
		</template>

		<v-list class="links">
			<v-list-item v-for="item in value" :key="item[primaryKeyField]">
				<v-list-item-content>
					<render-template :template="internalTemplate" :item="item" :collection="relatedCollection" />
				</v-list-item-content>
				<v-list-item-icon>
					<router-link :to="getLinkForItem(item)"><v-icon name="launch" small /></router-link>
				</v-list-item-icon>
			</v-list-item>
		</v-list>
	</v-menu>
	<render-template v-else :template="internalTemplate" :item="value" :collection="relatedCollection" />
</template>

<script lang="ts">
import { useI18n } from 'vue-i18n';
import { defineComponent, computed, PropType, Ref } from 'vue';
import getRelatedCollection from '@/utils/get-related-collection';
import useCollection from '@/composables/use-collection';
import ValueNull from '@/views/private/components/value-null';

export default defineComponent({
	components: { ValueNull },
	props: {
		collection: {
			type: String,
			required: true,
		},
		field: {
			type: String,
			required: true,
		},
		value: {
			type: [Array, Object] as PropType<Record<string, any> | Record<string, any>[]>,
			default: null,
		},
		template: {
			type: String,
			default: null,
		},
		type: {
			type: String,
			required: true,
		},
	},
	setup(props) {
		const { t, te } = useI18n();

		const relatedCollection = computed(() => {
			return getRelatedCollection(props.collection, props.field);
		});

		const primaryKeyField = computed(() => {
			if (relatedCollection.value !== null) {
				return useCollection(relatedCollection as unknown as Ref<string>).primaryKeyField.value;
			}
			return null;
		});

		const internalTemplate = computed(() => {
			return props.template || `{{ ${primaryKeyField.value!.field} }}`;
		});

		const unit = computed(() => {
			if (Array.isArray(props.value)) {
				if (props.value.length === 1) {
					if (te(`collection_names_singular.${relatedCollection.value}`)) {
						return t(`collection_names_singular.${relatedCollection.value}`);
					} else {
						return t('item');
					}
				} else {
					if (te(`collection_names_plural.${relatedCollection.value}`)) {
						return t(`collection_names_plural.${relatedCollection.value}`);
					} else {
						return t('items');
					}
				}
			}

			return null;
		});

		return { relatedCollection, primaryKeyField, getLinkForItem, internalTemplate, unit };

		function getLinkForItem(item: any) {
			if (!relatedCollection.value || !primaryKeyField.value) return null;
			const primaryKey = item[primaryKeyField.value.field];

			return `/collections/${relatedCollection.value}/${encodeURIComponent(primaryKey)}`;
		}
	},
});
</script>

<style lang="scss" scoped>
.toggle {
	position: relative;

	&::before {
		position: absolute;
		top: -6px;
		left: -6px;
		z-index: 1;
		width: calc(100% + 12px);
		height: calc(100% + 12px);
		background-color: var(--background-normal);
		border-radius: var(--border-radius);
		opacity: 0;
		transition: opacity var(--fast) var(--transition);
		content: '';
	}

	.label {
		position: relative;
		z-index: 2;
	}

	&:not(.subdued):hover::before {
		opacity: 1;
	}

	&:not(.subdued):active::before {
		background-color: var(--background-normal-alt);
	}
}

.subdued {
	color: var(--foreground-subdued);
}

.links {
	.v-list-item-content {
		height: var(--v-list-item-min-height);
	}
}
</style>

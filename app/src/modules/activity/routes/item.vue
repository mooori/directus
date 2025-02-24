<template>
	<v-drawer :model-value="isOpen" title="Activity Item" @update:model-value="close" @cancel="close">
		<v-progress-circular v-if="loading" indeterminate />

		<div v-else-if="error" class="content">
			<v-notice type="danger">
				{{ error }}
			</v-notice>
		</div>

		<div v-else class="content">
			<!-- @TODO add final design -->
			<p class="type-label">User:</p>
			<user-popover v-if="item.user" :user="item.user.id">
				{{ userName(item.user) }}
			</user-popover>

			<p class="type-label">Action:</p>
			<p>{{ item.action }}</p>

			<p class="type-label">Date:</p>
			<p>{{ item.timestamp }}</p>

			<p class="type-label">IP Address:</p>
			<p>{{ item.ip }}</p>

			<p class="type-label">User Agent:</p>
			<p>{{ item.user_agent }}</p>

			<p class="type-label">Collection:</p>
			<p>{{ item.collection }}</p>

			<p class="type-label">Item:</p>
			<p>{{ item.item }}</p>
		</div>

		<template #actions>
			<v-button v-if="openItemLink" v-tooltip.bottom="t('open')" :to="openItemLink" icon rounded>
				<v-icon name="launch" />
			</v-button>

			<v-button v-tooltip.bottom="t('done')" to="/activity" icon rounded>
				<v-icon name="check" />
			</v-button>
		</template>
	</v-drawer>
</template>

<script lang="ts">
import { useI18n } from 'vue-i18n';
import { defineComponent, computed, ref, watch } from 'vue';
import { useRouter } from 'vue-router';
import api from '@/api';
import { userName } from '@/utils/user-name';
import { useDialogRoute } from '@/composables/use-dialog-route';

type ActivityRecord = {
	user: {
		email: string;
		first_name: string;
		last_name: string;
	} | null;
	action: string;
	timestamp: string;
	ip: string;
	user_agent: string;
	collection: string;
	item: string;
};

export default defineComponent({
	name: 'ActivityDetail',
	props: {
		primaryKey: {
			type: String,
			required: true,
		},
	},
	setup(props) {
		const { t } = useI18n();

		const router = useRouter();

		const isOpen = useDialogRoute();

		const item = ref<ActivityRecord>();
		const loading = ref(false);
		const error = ref<any>(null);

		const openItemLink = computed(() => {
			if (!item.value) return;
			return `/collections/${item.value.collection}/${encodeURIComponent(item.value.item)}`;
		});

		watch(() => props.primaryKey, loadActivity, { immediate: true });

		return { t, isOpen, item, loading, error, close, openItemLink, userName };

		async function loadActivity() {
			loading.value = true;

			try {
				const response = await api.get(`/activity/${props.primaryKey}`, {
					params: {
						fields: [
							'user.id',
							'user.email',
							'user.first_name',
							'user.last_name',
							'action',
							'timestamp',
							'ip',
							'user_agent',
							'collection',
							'item',
						],
					},
				});

				item.value = response.data.data;
			} catch (err: any) {
				error.value = err;
			} finally {
				loading.value = false;
			}
		}

		function close() {
			router.push('/activity');
		}
	},
});
</script>

<style lang="scss" scoped>
.type-label:not(:first-child) {
	margin-top: 24px;
}

.content {
	padding: var(--content-padding);
	padding-top: 0;
	padding-bottom: var(--content-padding);
}
</style>

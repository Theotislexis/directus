<template>
	<span class="datetime">{{ displayValue }}</span>
</template>

<script lang="ts" setup>
import { useI18n } from 'vue-i18n';
import { ref, watch, computed, onMounted, onUnmounted } from 'vue';
import localizedFormat from '@/utils/localized-format';
import localizedFormatDistance from '@/utils/localized-format-distance';
import localizedFormatDistanceStrict from '@/utils/localized-format-distance-strict';
import { parseISO, parse } from 'date-fns';

interface Props {
	value: string;
	type: 'dateTime' | 'date' | 'time' | 'timestamp';
	format?: string;
	relative?: boolean;
	strict?: boolean;
	suffix?: boolean;
}

const props = withDefaults(defineProps<Props>(), {
	format: 'long',
	relative: false,
	strict: false,
	suffix: true,
});

const { t } = useI18n();

const displayValue = ref<string | null>(null);

const localValue = computed(() => {
	if (!props.value) return null;

	if (props.type === 'timestamp') {
		return parseISO(props.value);
	} else if (props.type === 'dateTime') {
		return parse(props.value, "yyyy-MM-dd'T'HH:mm:ss", new Date());
	} else if (props.type === 'date') {
		return parse(props.value, 'yyyy-MM-dd', new Date());
	} else if (props.type === 'time') {
		return parse(props.value, 'HH:mm:ss', new Date());
	}

	return null;
});

const relativeFormat = (value: Date) => {
	const fn = props.strict ? localizedFormatDistanceStrict : localizedFormatDistance;
	return fn(value, new Date(), {
		addSuffix: props.suffix,
	});
};

watch(
	localValue,
	async (newValue) => {
		if (newValue === null) {
			displayValue.value = null;
			return;
		}

		if (props.relative) {
			displayValue.value = await relativeFormat(newValue);
		} else {
			let format;
			if (props.format === 'long') {
				format = `${t('date-fns_date')} ${t('date-fns_time')}`;
				if (props.type === 'date') format = String(t('date-fns_date'));
				if (props.type === 'time') format = String(t('date-fns_time'));
			} else if (props.format === 'short') {
				format = `${t('date-fns_date_short')} ${t('date-fns_time_short')}`;
				if (props.type === 'date') format = String(t('date-fns_date_short'));
				if (props.type === 'time') format = String(t('date-fns_time_short'));
			} else {
				format = props.format;
			}

			displayValue.value = localizedFormat(newValue, format);
		}
	},
	{ immediate: true }
);

let refreshInterval: number | null = null;

onMounted(async () => {
	if (!props.relative) return;

	refreshInterval = window.setInterval(async () => {
		if (!localValue.value) return;
		displayValue.value = await relativeFormat(localValue.value);
	}, 60000);
});

onUnmounted(() => {
	if (refreshInterval) clearInterval(refreshInterval);
});
</script>

<style lang="scss" scoped>
.datetime {
	overflow: hidden;
	line-height: 1.15;
	white-space: nowrap;
	text-overflow: ellipsis;
}
</style>

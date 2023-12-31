<template>
	<div>
		<Combobox
			:modelValue="value"
			@update:modelValue="(val: string|{'value': string}|null) => {
				emit('update:modelValue', val)
			}"
			v-slot="{ open }"
			:nullable="nullable"
			:multiple="multiple">
			<div
				class="form-input flex h-7 w-full items-center justify-between gap-2 rounded border-gray-400 px-2 py-1 pr-6 text-sm transition-colors dark:border-zinc-700 dark:bg-zinc-800 dark:text-zinc-200 dark:focus:bg-zinc-700">
				<!-- {{ displayValue }} -->
				<ComboboxInput
					autocomplete="off"
					@change="query = $event.target.value"
					@focus="() => open"
					:displayValue="
						(option) => {
							if (Array.isArray(option)) {
								return option.map((o) => o.label).join(', ');
							} else if (option) {
								return option.label || option.value || '';
							} else {
								return '';
							}
						}
					"
					:placeholder="!modelValue ? placeholder : null"
					class="h-full w-full border-none bg-transparent p-0 text-xs focus:border-none focus:ring-0" />
			</div>
			<ComboboxOptions
				class="absolute right-0 z-50 max-h-[15rem] w-full max-w-[150px] overflow-y-auto rounded-lg bg-white px-1.5 py-1.5 shadow-2xl"
				v-show="filteredOptions.length">
				<ComboboxOption v-if="query" :value="query" class="flex items-center"></ComboboxOption>
				<ComboboxOption
					v-slot="{ active, selected }"
					v-for="option in filteredOptions"
					:key="option.value"
					:value="option"
					class="flex items-center">
					<li
						class="w-full select-none rounded px-2.5 py-1.5 text-xs"
						:class="{
							'bg-gray-100': active,
							'bg-gray-300': selected,
						}">
						{{ option.label }}
					</li>
				</ComboboxOption>
			</ComboboxOptions>
		</Combobox>
		<div
			class="absolute right-1 top-[3px] cursor-pointer p-1 text-gray-700 dark:text-zinc-300"
			@click="clearValue"
			v-show="modelValue">
			<svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24">
				<path
					fill="currentColor"
					d="M18.3 5.71a.996.996 0 0 0-1.41 0L12 10.59L7.11 5.7A.996.996 0 1 0 5.7 7.11L10.59 12L5.7 16.89a.996.996 0 1 0 1.41 1.41L12 13.41l4.89 4.89a.996.996 0 1 0 1.41-1.41L13.41 12l4.89-4.89c.38-.38.38-1.02 0-1.4z" />
			</svg>
		</div>
	</div>
</template>

<script setup lang="ts">
import { Combobox, ComboboxInput, ComboboxOption, ComboboxOptions } from "@headlessui/vue";
import { ComputedRef, PropType, computed, ref } from "vue";

type Option = {
	label: string;
	value: string;
};

const emit = defineEmits(["update:modelValue"]);

const props = defineProps({
	options: {
		type: Array as PropType<Option[]>,
		default: () => [],
	},
	modelValue: {},
	placeholder: {
		type: String,
		default: "Search",
	},
});

const query = ref("");

const multiple = computed(() => Array.isArray(props.modelValue));
const nullable = computed(() => !multiple.value);

const displayValue = computed(() => {
	if (Array.isArray(props.modelValue) && props.modelValue.length > 0) {
		return props.modelValue.join(", ");
	} else {
		return (props.modelValue as Option)?.label;
	}
});

const value = computed(() => {
	if (
		props.modelValue instanceof String ||
		typeof props.modelValue === "string" ||
		props.modelValue instanceof Number ||
		typeof props.modelValue === "number"
	) {
		return { label: props.modelValue, value: props.modelValue };
	} else {
		return props.modelValue;
	}
}) as ComputedRef<Option>;

const filteredOptions = computed(() => {
	return query.value === ""
		? props.options
		: props.options.filter((option) => {
				return (
					option.label.toLowerCase().includes(query.value.toLowerCase()) ||
					option.value.toLowerCase().includes(query.value.toLowerCase())
				);
		  });
});

const clearValue = () => emit("update:modelValue", null);
</script>

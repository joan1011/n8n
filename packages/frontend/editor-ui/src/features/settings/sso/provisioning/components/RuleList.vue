<script lang="ts" setup>
import Draggable from 'vuedraggable';
import type { RoleMappingRuleResponse, RoleMappingRuleType } from '../types';
import RuleRow from './RuleRow.vue';

const props = withDefaults(
	defineProps<{
		rules: RoleMappingRuleResponse[];
		type?: RoleMappingRuleType;
		projects?: Array<{ id: string; name: string }>;
		disabled?: boolean;
	}>(),
	{
		type: 'instance',
		projects: () => [],
		disabled: false,
	},
);

const emit = defineEmits<{
	reorder: [fromIndex: number, toIndex: number];
	update: [id: string, patch: Partial<RoleMappingRuleResponse>];
	delete: [id: string];
}>();

function onDragEnd(event: { oldIndex?: number; newIndex?: number }) {
	if (
		event.oldIndex !== undefined &&
		event.newIndex !== undefined &&
		event.oldIndex !== event.newIndex
	) {
		emit('reorder', event.oldIndex, event.newIndex);
	}
}
</script>
<template>
	<div :class="$style.list" data-test-id="rule-list">
		<Draggable
			:model-value="props.rules"
			item-key="id"
			handle=".drag-handle"
			:animation="150"
			:disabled="props.disabled"
			:drag-class="$style.dragging"
			:ghost-class="$style.ghost"
			:chosen-class="$style.chosen"
			@end="onDragEnd"
		>
			<template #item="{ element }">
				<RuleRow
					:rule="element"
					:type="props.type"
					:projects="props.projects"
					:disabled="props.disabled"
					@update="(id, patch) => emit('update', id, patch)"
					@delete="(id) => emit('delete', id)"
				/>
			</template>
		</Draggable>
		<p v-if="props.rules.length === 0" :class="$style.empty">
			No rules configured. Click "Add rule" to create one.
		</p>
	</div>
</template>
<style lang="scss" module>
.list {
	display: flex;
	flex-direction: column;
	gap: var(--spacing--2xs);
}

.empty {
	font-size: var(--font-size--2xs);
	color: var(--color--text--tint-2);
	text-align: center;
	padding: var(--spacing--lg) 0;
	margin: 0;
}

.ghost {
	opacity: 0.4;
}

.chosen {
	box-shadow: 0 2px 8px rgba(0, 0, 0, 0.12);
}

.dragging {
	opacity: 0.8;
	box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}
</style>

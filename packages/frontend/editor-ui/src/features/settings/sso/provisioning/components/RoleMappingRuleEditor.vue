<script lang="ts" setup>
import { computed, onMounted } from 'vue';
import { useI18n } from '@n8n/i18n';
import { useRBACStore } from '@/app/stores/rbac.store';
import { useProjectsStore } from '@/features/collaboration/projects/projects.store';
import { useRoleMappingRules } from '../composables/useRoleMappingRules';
import RuleSectionHeader from './RuleSectionHeader.vue';
import RuleList from './RuleList.vue';
import FallbackRoleSelect from './FallbackRoleSelect.vue';
import RemoveMappingButton from './RemoveMappingButton.vue';

const emit = defineEmits<{
	removeMapping: [];
}>();

const i18n = useI18n();
const canManageRules = useRBACStore().hasScope('roleMappingRule:create');
const projectsStore = useProjectsStore();

const availableProjects = computed(() =>
	projectsStore.teamProjects.map((p) => ({ id: p.id, name: p.name })),
);

const {
	instanceRules,
	projectRules,
	fallbackInstanceRole,
	isDirty,
	addRule,
	updateRule,
	deleteRule,
	reorder,
	loadRules,
	save,
} = useRoleMappingRules();

onMounted(() => {
	void loadRules();
});

defineExpose({ isDirty, save });
</script>
<template>
	<div :class="$style.editor" data-test-id="role-mapping-rule-editor">
		<RuleSectionHeader
			:title="i18n.baseText('settings.sso.settings.roleMappingRules.instanceRules.title')"
			:description="
				i18n.baseText('settings.sso.settings.roleMappingRules.instanceRules.description')
			"
			:disabled="!canManageRules"
			@add="addRule('instance')"
		/>
		<RuleList
			:rules="instanceRules"
			type="instance"
			:disabled="!canManageRules"
			@reorder="(from, to) => reorder('instance', from, to)"
			@update="updateRule"
			@delete="deleteRule"
		/>

		<FallbackRoleSelect v-model="fallbackInstanceRole" :disabled="!canManageRules" />

		<RuleSectionHeader
			:title="i18n.baseText('settings.sso.settings.roleMappingRules.projectRules.title')"
			:description="
				i18n.baseText('settings.sso.settings.roleMappingRules.projectRules.description')
			"
			:disabled="!canManageRules"
			@add="addRule('project')"
		/>
		<RuleList
			:rules="projectRules"
			type="project"
			:projects="availableProjects"
			:disabled="!canManageRules"
			@reorder="(from, to) => reorder('project', from, to)"
			@update="updateRule"
			@delete="deleteRule"
		/>

		<RemoveMappingButton :disabled="!canManageRules" @remove="emit('removeMapping')" />
	</div>
</template>
<style lang="scss" module>
.editor {
	padding: var(--spacing--lg) 0;
}
</style>

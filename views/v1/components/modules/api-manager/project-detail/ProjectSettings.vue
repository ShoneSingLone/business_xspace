<script lang="ts">
export default async function () {
	const api = await _.$importVue("@/utils/api.vue");

	return {
		props: {
			project: {
				type: Object,
				default: () => ({})
			},
			projectData: {
				type: Object,
				default: () => ({})
			}
		},
		data() {
			return {
				form: {
					name: '',
					desc: '',
					basepath: '',
					project_type: 'private'
				},
				isSaving: false
			};
		},
		mounted() {
			this.initForm();
		},
		methods: {
			initForm() {
				if (this.projectData) {
					this.form.name = this.projectData.name || '';
					this.form.desc = this.projectData.desc || '';
					this.form.basepath = this.projectData.basepath || '';
					this.form.project_type = this.projectData.project_type || 'private';
				}
			},
			async handleSave() {
				if (!this.form.name.trim()) {
					_.$msgError('项目名称不能为空');
					return;
				}

				this.isSaving = true;
				try {
					const response = await api.projectUp({
						id: this.project.id,
						name: this.form.name,
						desc: this.form.desc,
						basepath: this.form.basepath,
						project_type: this.form.project_type
					});

					if (response.errcode === 0) {
						_.$msgSuccess('保存成功');
						this.$emit('refresh');
					} else {
						_.$msgError(response.errmsg || '保存失败');
					}
				} catch (error) {
					console.error('Failed to save project settings:', error);
					_.$msgError('保存失败，请重试');
				} finally {
					this.isSaving = false;
				}
			}
		}
	};
}
</script>

<template>
	<div class="project-settings">
		<div class="project-settings__form">
			<div class="project-settings__form-section">
				<h3 class="project-settings__section-title">基本信息</h3>
				
				<div class="project-settings__field">
					<label class="project-settings__label">项目名称</label>
					<input 
						v-model="form.name"
						type="text" 
						class="project-settings__input"
						placeholder="请输入项目名称" />
				</div>

				<div class="project-settings__field">
					<label class="project-settings__label">项目描述</label>
					<textarea 
						v-model="form.desc"
						class="project-settings__textarea"
						placeholder="请输入项目描述"
						rows="3"></textarea>
				</div>

				<div class="project-settings__field">
					<label class="project-settings__label">基础路径</label>
					<input 
						v-model="form.basepath"
						type="text" 
						class="project-settings__input"
						placeholder="例如：/api/v1" />
				</div>

				<div class="project-settings__field">
					<label class="project-settings__label">项目类型</label>
					<div class="project-settings__radio-group">
						<label class="project-settings__radio">
							<input 
								type="radio" 
								value="private" 
								v-model="form.project_type" />
							<span class="project-settings__radio-label">私有项目</span>
						</label>
						<label class="project-settings__radio">
							<input 
								type="radio" 
								value="public" 
								v-model="form.project_type" />
							<span class="project-settings__radio-label">公开项目</span>
						</label>
					</div>
				</div>
			</div>

			<div class="project-settings__actions">
				<button 
					@click="handleSave"
					:disabled="isSaving"
					class="project-settings__btn project-settings__btn--primary">
					<xIcon v-if="isSaving" icon="_loader-2" :size="16" class="project-settings__btn-icon" />
					<span>{{ isSaving ? '保存中...' : '保存设置' }}</span>
				</button>
			</div>
		</div>
	</div>
</template>

<style lang="less">
.project-settings {
	height: 100%;
	display: flex;
	flex-direction: column;

	&__form {
		max-width: 600px;
	}

	&__form-section {
		margin-bottom: 32px;
	}

	&__section-title {
		margin: 0 0 16px;
		font-size: 16px;
		font-weight: 700;
		color: var(--color-on-surface);
		padding-bottom: 8px;
		border-bottom: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
	}

	&__field {
		margin-bottom: 20px;
	}

	&__label {
		display: block;
		margin-bottom: 8px;
		font-size: 14px;
		font-weight: 600;
		color: var(--color-on-surface-variant);
	}

	&__input {
		width: 100%;
		padding: 10px 12px;
		border: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
		border-radius: 8px;
		font-size: 14px;
		background: var(--color-surface-container-low);
		color: var(--color-on-surface);
		transition: border-color 0.15s ease;

		&:focus {
			outline: none;
			border-color: var(--color-primary);
		}

		&::placeholder {
			color: var(--color-on-surface-variant);
			opacity: 0.6;
		}
	}

	&__textarea {
		width: 100%;
		padding: 10px 12px;
		border: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
		border-radius: 8px;
		font-size: 14px;
		background: var(--color-surface-container-low);
		color: var(--color-on-surface);
		resize: vertical;
		font-family: inherit;
		transition: border-color 0.15s ease;

		&:focus {
			outline: none;
			border-color: var(--color-primary);
		}

		&::placeholder {
			color: var(--color-on-surface-variant);
			opacity: 0.6;
		}
	}

	&__radio-group {
		display: flex;
		gap: 16px;
	}

	&__radio {
		display: flex;
		align-items: center;
		gap: 8px;
		cursor: pointer;

		input[type="radio"] {
			margin: 0;
			cursor: pointer;
		}
	}

	&__radio-label {
		font-size: 14px;
		color: var(--color-on-surface);
	}

	&__actions {
		display: flex;
		gap: 12px;
		padding-top: 16px;
		border-top: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
	}

	&__btn {
		display: inline-flex;
		align-items: center;
		gap: 6px;
		padding: 10px 20px;
		border: 0;
		border-radius: 8px;
		font-size: 14px;
		font-weight: 600;
		cursor: pointer;
		transition: all 0.15s ease;

		&--primary {
			background: var(--color-primary);
			color: var(--color-on-primary);

			&:hover:not(:disabled) {
				background: var(--color-primary-container);
				color: var(--color-on-primary-container);
			}

			&:disabled {
				opacity: 0.6;
				cursor: not-allowed;
			}
		}
	}

	&__btn-icon {
		animation: spin 1s linear infinite;
	}
}

@keyframes spin {
	from {
		transform: rotate(0deg);
	}
	to {
		transform: rotate(360deg);
	}
}
</style>

<script lang="ts">
export default async function () {
	return {
		components: {
			MemberListPanel: () => _.$importVue("@/views/v1/components/organisms/MemberListPanel.vue")
		},
		props: {
			node: {
				type: Object,
				default: null
			},
			members: {
				type: Array,
				default: null
			},
			membersLoading: {
				type: Boolean,
				default: false
			},
			canManageMembers: {
				type: Boolean,
				default: false
			},
			editingContent: {
				type: [Object, String],
				default: null
			},
			isRequesting: {
				type: Boolean,
				default: false
			},
			responseData: {
				type: String,
				default: null
			},
			responseStatus: {
				type: Number,
				default: null
			},
			responseTime: {
				type: Number,
				default: null
			},
			activeEnvironment: {
				type: String,
				default: "Development"
			}
		},
		methods: {
			startEdit() {
				this.$emit("start-edit");
			},
			saveEdit() {
				this.$emit("save-edit");
			},
			cancelEdit() {
				this.$emit("cancel-edit");
			},
			sendRequest() {
				this.$emit("send-request");
			},
			addMember() {
				this.$emit("add-member");
			},
			removeMember(member) {
				this.$emit("remove-member", member);
			},
			changeMemberRole(member, role) {
				this.$emit("change-member-role", { member, role });
			},
			handlePlainTextInput(event) {
				this.$emit("update:editingContent", event.target.value);
			},
			getIcon(type) {
				switch (type) {
					case "api": return "code";
					case "doc": return "file-text";
					case "code": return "code";
					case "member_list": return "users";
					case "setting": return "settings";
					default: return "file";
				}
			},
			getIconColor(type) {
				switch (type) {
					case "api": return "api-manager__icon api-manager__icon--api";
					case "doc": return "api-manager__icon api-manager__icon--doc";
					case "code": return "api-manager__icon api-manager__icon--code";
					case "member_list": return "api-manager__icon api-manager__icon--member-list";
					case "setting": return "api-manager__icon api-manager__icon--setting";
					default: return "api-manager__icon api-manager__icon--default";
				}
			},
			getMethodColor(method) {
				switch (method?.toUpperCase()) {
					case "GET": return "api-manager__method-badge api-manager__method-badge--get";
					case "POST": return "api-manager__method-badge api-manager__method-badge--post";
					case "PUT": return "api-manager__method-badge api-manager__method-badge--put";
					case "DELETE": return "api-manager__method-badge api-manager__method-badge--delete";
					default: return "api-manager__method-badge api-manager__method-badge--default";
				}
			}
		}
	};
}
</script>

<template>
	<div class="api-manager__editor-wrap">
		<div :class="['api-manager__editor-card', node?.type !== 'member_list' ? 'api-manager__editor-card--wide' : '']">
			<div class="api-manager__editor-header">
				<div class="api-manager__editor-header-left">
					<xIcon :icon="getIcon(node?.type)" :size="24" :class="getIconColor(node?.type)" />
					<div class="api-manager__editor-header-copy">
						<h2 class="api-manager__editor-title">{{ node?.name }}</h2>
						<p class="api-manager__editor-subtitle">{{ node?.type?.replace("_", " ") }}</p>
					</div>
				</div>
				<div class="api-manager__editor-header-actions" v-if="node?.type !== 'member_list'">
					<template v-if="editingContent">
						<button @click="cancelEdit" class="api-manager__action-btn api-manager__action-btn--ghost">Cancel</button>
						<button @click="saveEdit" class="api-manager__action-btn api-manager__action-btn--primary api-manager__action-btn--with-icon">
							<xIcon icon="save" :size="16" />
							Save
						</button>
					</template>
					<template v-else>
						<button @click="startEdit" class="api-manager__action-btn api-manager__action-btn--secondary api-manager__action-btn--with-icon">
							<xIcon icon="edit" :size="16" />
							Edit
						</button>
					</template>
				</div>
			</div>

			<div class="api-manager__editor-body">
				<template v-if="node?.type === 'member_list'">
					<MemberListPanel
						:groupName="node?.groupName"
						:members="members || node?.content || []"
						:loading="membersLoading"
						:canManage="canManageMembers"
						@add-member="addMember"
						@remove-member="removeMember"
						@change-member-role="changeMemberRole" />
				</template>

				<template v-else-if="node?.type === 'setting'">
					<div class="api-manager__settings">
						<div v-for="(val, key) in (editingContent || node?.content)" :key="key" class="api-manager__setting-row">
							<label class="api-manager__setting-label">{{ String(key).replace(/([A-Z])/g, " $1").trim() }}</label>
							<div class="api-manager__setting-control">
								<input v-if="editingContent" v-model="editingContent[key]" class="api-manager__field-input api-manager__field-input--block" />
								<div v-else class="api-manager__field-value">{{ val }}</div>
							</div>
						</div>
					</div>
				</template>

				<template v-else-if="node?.type === 'api'">
					<div class="api-manager__api-editor">
						<div class="api-manager__api-row">
							<select v-if="editingContent" v-model="editingContent.method" class="api-manager__field-input api-manager__method-select">
								<option>GET</option>
								<option>POST</option>
								<option>PUT</option>
								<option>DELETE</option>
							</select>
							<span v-else :class="getMethodColor(node.content?.method)">
								{{ node.content?.method }}
							</span>

							<input v-if="editingContent" v-model="editingContent.endpoint" class="api-manager__field-input api-manager__field-input--code api-manager__endpoint-input" />
							<span v-else class="api-manager__endpoint-text">{{ node.content?.endpoint }}</span>

							<button v-if="!editingContent" @click="sendRequest" :disabled="isRequesting" class="api-manager__action-btn api-manager__action-btn--primary api-manager__action-btn--with-icon">
								<xIcon icon="_loader-2" v-if="isRequesting" :size="16" class="api-manager__spinner" />
								<xIcon icon="video-play" v-else :size="16" />
								Send
							</button>
						</div>

						<div class="api-manager__field">
							<label class="api-manager__field-label">Description</label>
							<textarea v-if="editingContent" v-model="editingContent.description" rows="2" class="api-manager__field-input api-manager__field-input--textarea"></textarea>
							<p v-else class="api-manager__field-text">{{ node.content?.description }}</p>
						</div>

						<div v-if="editingContent?.params || node.content?.params">
							<label class="api-manager__field-label api-manager__field-label--spaced">Parameters</label>
							<div class="api-manager__code-block">
								<pre>{{ JSON.stringify(editingContent?.params || node.content?.params, null, 2) }}</pre>
							</div>
						</div>

						<div v-if="editingContent?.body || node.content?.body">
							<label class="api-manager__field-label api-manager__field-label--spaced">Request Body</label>
							<div class="api-manager__code-block">
								<pre>{{ editingContent?.body || node.content?.body }}</pre>
							</div>
						</div>

						<div v-if="!editingContent" class="api-manager__response">
							<div class="api-manager__response-header">
								<h3 class="api-manager__response-title">
									Response
									<span v-if="responseStatus" class="api-manager__status-badge">{{ responseStatus }} OK</span>
								</h3>
								<span v-if="responseTime" class="api-manager__response-meta">{{ responseTime }} ms</span>
							</div>

							<div v-if="isRequesting" class="api-manager__state-panel api-manager__state-panel--loading">
								<xIcon icon="_loader-2" class="api-manager__spinner api-manager__spinner--lg" />
								<p class="api-manager__state-text">Sending request to {{ activeEnvironment }}...</p>
							</div>
							<div v-else-if="responseData" class="api-manager__state-panel api-manager__state-panel--code">
								<pre>{{ responseData }}</pre>
							</div>
							<div v-else class="api-manager__state-panel api-manager__state-panel--dashed">
								Click "Send" to execute the request
							</div>
						</div>
					</div>
				</template>

				<template v-else-if="node?.type === 'doc' || node?.type === 'code'">
					<div class="api-manager__doc-editor">
						<textarea v-if="editingContent" v-model="editingContent" class="api-manager__field-input api-manager__field-input--code api-manager__doc-textarea"></textarea>
						<div v-else class="api-manager__doc-preview">
							<pre class="api-manager__code-block api-manager__code-block--wrap">{{ node.content }}</pre>
						</div>
					</div>
				</template>

				<template v-else>
					<textarea 
						v-if="editingContent" 
						:value="typeof editingContent === 'string' ? editingContent : JSON.stringify(editingContent, null, 2)"
						@input="handlePlainTextInput" 
						class="api-manager__field-input api-manager__field-input--code api-manager__json-textarea"></textarea>
					<pre v-else class="api-manager__code-block api-manager__code-block--scroll">
						{{ typeof node?.content === "string" ? node.content : JSON.stringify(node?.content, null, 2) }}
					</pre>
				</template>
			</div>
		</div>
	</div>
</template>

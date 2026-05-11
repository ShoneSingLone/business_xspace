<script lang="ts">
export default async function () {
	return {
		formatDate(dateString) {
			try {
				const date = new Date(dateString);
				return date.toLocaleString();
			} catch (e) {
				return dateString;
			}
		},
		isFolderType(type) {
			return [
				"personal",
				"group",
				"project",
				"api_folder",
				"doc_folder",
				"folder"
			].includes(type);
		},
		findParentNode(root, targetId) {
			if (Array.isArray(root)) {
				for (const item of root) {
					if (item.id === targetId) return null;
					const found = this.findParentNode(item, targetId);
					if (found) return found;
				}
				return null;
			}

			if (!root.children) return null;
			for (const child of root.children) {
				if (child.id === targetId) return root;
				const found = this.findParentNode(child, targetId);
				if (found) return found;
			}
			return null;
		},
		getIcon(type) {
			switch (type) {
				case "personal":
					return "user";
				case "group":
					return "folder";
				case "project":
					return "folder";
				case "api_folder":
					return "folder";
				case "doc_folder":
					return "folder";
				case "folder":
					return "folder";
				case "api":
					return "code";
				case "doc":
					return "file-text";
				case "code":
					return "code";
				case "member_list":
					return "users";
				case "setting":
					return "settings";
				case "cicd":
					return "activity";
				case "log":
					return "history";
				default:
					return "file";
			}
		},
		getIconColor(type, node) {
			switch (type) {
				case "personal":
					return "api-manager__icon api-manager__icon--personal";
				case "group":
					return "api-manager__icon api-manager__icon--group";
				case "project":
					if (node && node.path) {
						if (node.path.includes("/个人空间/我的项目/")) {
							return "api-manager__icon api-manager__icon--project-personal";
						} else if (node.path.includes("/个人空间/星标项目/")) {
							return "api-manager__icon api-manager__icon--project-starred";
						} else if (node.path.includes("/群组/")) {
							return "api-manager__icon api-manager__icon--project-group";
						}
					}
					return "api-manager__icon api-manager__icon--project";
				case "api_folder":
					return "api-manager__icon api-manager__icon--api-folder";
				case "doc_folder":
					return "api-manager__icon api-manager__icon--doc-folder";
				case "folder":
					return "api-manager__icon api-manager__icon--folder";
				case "api":
					return "api-manager__icon api-manager__icon--api";
				case "doc":
					return "api-manager__icon api-manager__icon--doc";
				case "code":
					return "api-manager__icon api-manager__icon--code";
				case "member_list":
					return "api-manager__icon api-manager__icon--member-list";
				case "setting":
					return "api-manager__icon api-manager__icon--setting";
				case "cicd":
					return "api-manager__icon api-manager__icon--cicd";
				case "log":
					return "api-manager__icon api-manager__icon--log";
				default:
					return "api-manager__icon api-manager__icon--default";
			}
		},
		getMethodColor(method) {
			switch (method?.toUpperCase()) {
				case "GET":
					return "api-manager__method-badge api-manager__method-badge--get";
				case "POST":
					return "api-manager__method-badge api-manager__method-badge--post";
				case "PUT":
					return "api-manager__method-badge api-manager__method-badge--put";
				case "DELETE":
					return "api-manager__method-badge api-manager__method-badge--delete";
				default:
					return "api-manager__method-badge api-manager__method-badge--default";
			}
		},
		loadExpandedFolders() {
			try {
				const saved = localStorage.getItem("apiManager_expandedFolders");
				let folders = saved ? JSON.parse(saved) : ["personal_space", "groups"];
				return folders.filter(folderId => 
					folderId !== "ps_my_projects" && 
					folderId !== "ps_followed" && 
					!folderId.endsWith("_projects")
				);
			} catch {
				return ["personal_space", "groups"];
			}
		},
		saveExpandedFolders(expandedFolders) {
			try {
				localStorage.setItem("apiManager_expandedFolders", JSON.stringify(expandedFolders));
			} catch (e) {
				console.error("Failed to save expanded folders:", e);
			}
		},
		loadSelectedNodeId() {
			try {
				return localStorage.getItem("apiManager_selectedNodeId") || "personal_space";
			} catch {
				return "personal_space";
			}
		},
		saveSelectedNodeId(nodeId) {
			try {
				localStorage.setItem("apiManager_selectedNodeId", nodeId);
			} catch (e) {
				console.error("Failed to save selected node:", e);
			}
		}
	};
}
</script>
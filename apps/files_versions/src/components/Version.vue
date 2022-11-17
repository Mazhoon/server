<!--
 - @copyright 2022 Carl Schwan <carl@carlschwan.eu>
 - @license AGPL-3.0-or-later
 -
 - This program is free software: you can redistribute it and/or modify
 - it under the terms of the GNU Affero General Public License as
 - published by the Free Software Foundation, either version 3 of the
 - License, or (at your option) any later version.
 -
 - This program is distributed in the hope that it will be useful,
 - but WITHOUT ANY WARRANTY; without even the implied warranty of
 - MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
 - GNU Affero General Public License for more details.
 -
 - You should have received a copy of the GNU Affero General Public License
 - along with this program. If not, see <http://www.gnu.org/licenses/>.
 -->
<template>
	<div>
		<NcListItem class="version"
			:title="versionLabel"
			:href="version.url">
			<template #icon>
				<img lazy="true"
					:src="version.preview"
					alt=""
					height="256"
					width="256"
					class="version__image">
			</template>
			<template #subtitle>
				<div class="version__info">
					<span v-tooltip="formattedDate">{{ version.mtime | humanDateFromNow }}</span>
					<!-- Separate dot to improve alignement -->
					<span class="version__info__size">•</span>
					<span class="version__info__size">{{ version.size | humanReadableSize }}</span>
				</div>
			</template>
			<template #actions>
				<NcActionButton :close-after-click="true"
					@click="openVersionLabelModal">
					<template #icon>
						<Pencil :size="22" />
					</template>
					{{ version.label === '' ? t('files_versions', 'Name this version') : t('files_versions', 'Edit version name') }}
				</NcActionButton>
				<NcActionButton v-if="!isCurrent"
					:close-after-click="true"
					@click="restoreVersion">
					<template #icon>
						<BackupRestore :size="22" />
					</template>
					{{ t('files_versions', 'Restore version') }}
				</NcActionButton>
				<NcActionLink :href="version.url"
					:close-after-click="true"
					:download="version.url">
					<template #icon>
						<Download :size="22" />
					</template>
					{{ t('files_versions', 'Download version') }}
				</NcActionLink>
				<NcActionButton v-if="!isCurrent"
					:close-after-click="true"
					@click="deleteVersion">
					<template #icon>
						<Delete :size="22" />
					</template>
					{{ t('files_versions', 'Delete version') }}
				</NcActionButton>
			</template>
		</NcListItem>
		<NcModal v-if="showVersionLabelForm"
			:title="t('files_versions', 'Name this version')"
			@close="showVersionLabelForm = false">
			<form class="version-name-modal"
				@submit="setVersionLabel(formVersionNameValue)">
				<label>
					<div class="version-name-modal__title">{{ t('photos', 'Version name') }}</div>
					<NcTextField ref="nameInput"
						:value.sync="formVersionNameValue"
						:placeholder="t('photos', 'Version name')"
						:label-outside="true" />
				</label>

				<div class="version-name-modal__info">
					{{ t('photos', 'Named versions are persisted, and excluded from automatic cleanups when your storage quota is full.') }}
				</div>

				<div class="version-name-modal__actions">
					<NcButton :disabled="formVersionNameValue.trim().length === 0" @click="setVersionLabel('')">
						{{ t('files_versions', 'Remove version name') }}
					</NcButton>
					<NcButton type="primary" native-type="submit">
						<template #icon>
							<Check />
						</template>
						{{ t('files_versions', 'Save version name') }}
					</NcButton>
				</div>
			</form>
		</NcModal>
	</div>
</template>

<script>
import BackupRestore from 'vue-material-design-icons/BackupRestore.vue'
import Download from 'vue-material-design-icons/Download.vue'
import Pencil from 'vue-material-design-icons/Pencil.vue'
import Check from 'vue-material-design-icons/Check.vue'
import Delete from 'vue-material-design-icons/Delete'
import { NcActionButton, NcActionLink, NcListItem, NcModal, NcButton, NcTextField, Tooltip } from '@nextcloud/vue'
import moment from '@nextcloud/moment'
import { translate } from '@nextcloud/l10n'

export default {
	name: 'Version',
	components: {
		NcActionLink,
		NcActionButton,
		NcListItem,
		NcModal,
		NcButton,
		NcTextField,
		BackupRestore,
		Download,
		Pencil,
		Check,
		Delete,
	},
	directives: {
		// TODO: not working
		// https://nextcloud-vue-components.netlify.app/#/Directives
		tooltip: Tooltip,
	},
	filters: {
		humanReadableSize(bytes) {
			return OC.Util.humanFileSize(bytes)
		},
		humanDateFromNow(timestamp) {
			return moment(timestamp).fromNow()
		},
	},
	props: {
		/** @type {Vue.PropOptions<import('../utils/versions.js').Version>} */
		version: {
			type: Object,
			required: true,
		},
		isCurrent: {
			type: Boolean,
			default: false,
		},
		isFirstVersion: {
			type: Boolean,
			default: false,
		},
	},
	data() {
		return {
			showVersionLabelForm: false,
			formVersionNameValue: this.version.label,
		}
	},
	computed: {
		versionLabel() {
			if (this.isCurrent && this.version.label === '') {
				return translate('files_versions', 'Current version')
			}

			if (this.isFirstVersion && this.version.label === '') {
				return translate('files_versions', 'Initial version')
			}

			return this.version.label
		},
	},
	methods: {
		openVersionLabelModal() {
			this.showVersionLabelForm = true
			this.$nextTick(() => {
				this.$refs.nameInput.$el.getElementsByTagName('input')[0].focus()
			})
		},

		restoreVersion() {
			this.$emit('restore', this.version)
		},

		setVersionLabel(name) {
			this.formVersionNameValue = name
			this.showVersionNameForm = false
			this.$emit('label-update', this.version, name)
		},

		deleteVersion() {
			this.$emit('delete', this.version)
		},

		formattedDate() {
			return moment(this.version.mtime)
		},
	},
}
</script>

<style scoped lang="scss">
.version {
	display: flex;
	flex-direction: row;

	&__info {
		display: flex;
		flex-direction: row;
		align-items: center;
		gap: 0.5rem;

		&__size {
			color: var(--color-text-lighter);
		}
	}

	&__image {
		width: 3rem;
		height: 3rem;
		border: 1px solid var(--color-border);
		margin-right: 1rem;
		border-radius: var(--border-radius-large);
	}
}

.version-name-modal {
	display: flex;
	justify-content: space-between;
	flex-direction: column;
	height: 250px;
	padding: 16px;

	&__title {
		margin-bottom: 12px;
		font-weight: 600;
	}

	&__info {
		margin-top: 12px;
		color: var(--color-text-maxcontrast);
	}

	&__actions {
		display: flex;
		justify-content: space-between;
		margin-top: 64px;
	}
}
</style>

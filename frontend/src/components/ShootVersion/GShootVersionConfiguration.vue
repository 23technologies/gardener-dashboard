<!--
SPDX-FileCopyrightText: 2023 SAP SE or an SAP affiliate company and Gardener contributors

SPDX-License-Identifier: Apache-2.0
-->

<template>
  <g-action-button-dialog
    ref="actionDialog"
    :shoot-item="shootItem"
    :icon="shootSupportedPatchAvailable ? 'mdi-arrow-up-bold-circle' : 'mdi-arrow-up-bold-circle-outline'"
    width="450"
    caption="Update Cluster"
    confirm-button-text="Update"
    :confirm-required="confirmRequired"
    :text="buttonText"
    :disabled="!canUpdate"
    @dialog-opened="onConfigurationDialogOpened"
  >
    <template #content>
      <v-card-text>
        <g-shoot-version-update
          ref="shootVersionUpdate"
          :available-k8s-updates="shootAvailableK8sUpdates"
          :current-k8s-version="shootKubernetesVersionObject"
          @selected-version="onSelectedVersion"
          @selected-version-type="onSelectedVersionType"
          @confirm-required="onConfirmRequired"
        />
        <template v-if="!v$.$invalid && selectedVersionType === 'minor'">
          <div class="my-2">
            You should always test your scenario and back up all your data before attempting an upgrade. Don’t forget to include the workload inside your cluster!
          </div>
          <div class="my-2">
            You should consider the
            <a
              href="https://github.com/kubernetes/kubernetes/releases"
              target="_blank"
              rel="noopener"
              class="text-anchor"
            >
              Kubernetes release notes
              <v-icon style="font-size:80%">mdi-open-in-new</v-icon>
            </a>
            before upgrading your cluster.
          </div>
          <div class="my-2">
            Type <strong>{{ shootName }}</strong> below and confirm to upgrade the Kubernetes version of your cluster.
          </div>
          <div class="my-2 font-weight-bold">
            This action cannot be undone.
          </div>
        </template>
        <template v-if="!v$.$invalid && selectedVersionType === 'patch'">
          <div class="my-2">
            Applying a patch to your cluster will increase the Kubernetes version which can lead to unexpected side effects.
          </div>
          <div class="my-2 font-weight-bold">
            This action cannot be undone.
          </div>
        </template>
      </v-card-text>
    </template>
  </g-action-button-dialog>
</template>

<script>
import { useVuelidate } from '@vuelidate/core'

import GShootVersionUpdate from '@/components/ShootVersion/GShootVersionUpdate.vue'
import GActionButtonDialog from '@/components/dialogs/GActionButtonDialog'

import { errorDetailsFromError } from '@/utils/error'
import shootItem from '@/mixins/shootItem'

import { find } from '@/lodash'

export default {
  components: {
    GActionButtonDialog,
    GShootVersionUpdate,
  },
  mixins: [
    shootItem,
  ],
  inject: ['api', 'logger'],
  props: {
    text: {
      type: Boolean,
      default: false,
    },
  },
  setup () {
    return {
      v$: useVuelidate(),
    }
  },
  data () {
    return {
      selectedVersion: undefined,
      selectedVersionType: undefined,
      confirmRequired: false,
      updateErrorMessage: null,
      updateDetailedErrorMessage: null,
    }
  },
  computed: {
    shootSupportedPatchAvailable () {
      return !!find(this.shootAvailableK8sUpdates?.patch, 'isSupported')
    },
    canUpdate () {
      return !!this.shootAvailableK8sUpdates
    },
    confirm () {
      return this.confirmRequired ? this.shootName : undefined
    },
    buttonText () {
      if (!this.text) {
        return
      }
      return 'Update Cluster'
    },
  },
  methods: {
    onSelectedVersion (value) {
      this.selectedVersion = value
    },
    onSelectedVersionType (value) {
      this.selectedVersionType = value
    },
    onConfirmRequired (value) {
      this.confirmRequired = value
    },
    async onConfigurationDialogOpened () {
      await this.reset()
      const confirmed = await this.$refs.actionDialog.waitForDialogClosed()
      if (confirmed) {
        await this.updateConfiguration()
      }
    },
    async updateConfiguration () {
      try {
        await this.api.updateShootVersion({
          namespace: this.shootNamespace,
          name: this.shootName,
          data: {
            version: this.selectedVersion,
          },
        })
      } catch (err) {
        const errorMessage = 'Update Kubernetes version failed'
        const errorDetails = errorDetailsFromError(err)
        const detailedErrorMessage = errorDetails.detailedMessage
        this.$refs.actionDialog.setError({ errorMessage, detailedErrorMessage })
        this.logger.error(errorMessage, errorDetails.errorCode, errorDetails.detailedMessage, err)
      }
    },
    reset () {
      const defaultData = this.$options.data.apply(this)
      Object.assign(this.$data, defaultData)

      this.updateErrorMessage = undefined
      this.updateDetailedErrorMessage = undefined

      this.$refs.shootVersionUpdate.reset()
    },
  },
}
</script>

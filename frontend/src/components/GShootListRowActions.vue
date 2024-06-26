<!--
SPDX-FileCopyrightText: 2023 SAP SE or an SAP affiliate company and Gardener contributors

SPDX-License-Identifier: Apache-2.0
-->

<template>
  <v-menu
    v-model="actionMenu"
    location="left"
    :close-on-content-click="false"
    eager
  >
    <template #activator="{ props }">
      <g-action-button
        v-bind="props"
        icon="mdi-dots-vertical"
        tooltip="Cluster Actions"
      />
    </template>
    <v-list
      dense
      @click.capture="actionMenu = false"
    >
      <v-list-item>
        <g-shoot-action-change-hibernation
          :shoot-item="shootItem"
          text
        />
      </v-list-item>
      <v-list-item>
        <g-shoot-action-maintenance-start
          :shoot-item="shootItem"
          text
        />
      </v-list-item>
      <v-list-item>
        <g-shoot-action-reconcile-start
          :shoot-item="shootItem"
          text
        />
      </v-list-item>
      <v-list-item>
        <g-shoot-action-rotate-credentials
          :shoot-item="shootItem"
          :type="rotationType"
          text
        />
      </v-list-item>
      <v-list-item>
        <g-shoot-version-configuration
          :shoot-item="shootItem"
          text
        />
      </v-list-item>
      <v-divider />
      <v-list-item>
        <g-shoot-action-delete-cluster
          v-if="!canForceDeleteShoot"
          :shoot-item="shootItem"
          text
        />
        <g-shoot-action-force-delete
          v-else
          :shoot-item="shootItem"
          text
        />
      </v-list-item>
    </v-list>
  </v-menu>
</template>

<script>
import GActionButton from '@/components/GActionButton.vue'
import GShootActionChangeHibernation from '@/components/ShootHibernation/GShootActionChangeHibernation.vue'
import GShootActionMaintenanceStart from '@/components/ShootMaintenance/GShootActionMaintenanceStart.vue'
import GShootActionReconcileStart from '@/components/GShootActionReconcileStart.vue'
import GShootActionRotateCredentials from '@/components/GShootActionRotateCredentials.vue'
import GShootActionDeleteCluster from '@/components/GShootActionDeleteCluster.vue'
import GShootActionForceDelete from '@/components/GShootActionForceDelete.vue'
import GShootVersionConfiguration from '@/components/ShootVersion/GShootVersionConfiguration.vue'

import { shootItem } from '@/mixins/shootItem'

export default {
  components: {
    GActionButton,
    GShootActionChangeHibernation,
    GShootActionMaintenanceStart,
    GShootActionReconcileStart,
    GShootActionRotateCredentials,
    GShootActionDeleteCluster,
    GShootActionForceDelete,
    GShootVersionConfiguration,
  },
  mixins: [shootItem],
  props: {
    shootItem: {
      type: Object,
    },
  },
  data () {
    return {
      menu: false,
      rotationType: 'ALL_CREDENTIALS',
    }
  },
  computed: {
    actionMenu: {
      get () {
        return this.menu
      },
      set (value) {
        this.menu = value
      },
    },
  },
}
</script>

<!--
  Copyright (C) 2025 Nethesis S.r.l.
  SPDX-License-Identifier: GPL-3.0-or-later
-->
<template>
  <cv-grid fullWidth>
    <cv-row>
      <cv-column class="page-title">
        <h2>{{ $t("settings.title") }}</h2>
      </cv-column>
    </cv-row>

    <cv-row v-if="error.getConfiguration">
      <cv-column>
        <NsInlineNotification
          kind="error"
          :title="$t('action.get-configuration')"
          :description="error.getConfiguration"
          :showCloseButton="false"
        />
      </cv-column>
    </cv-row>

    <cv-row>
      <cv-column>
        <cv-tile light>
          <cv-form @submit.prevent="configureModule">
            <!-- host input -->
            <NsTextInput
              :label="$t('settings.lemonldapng_fqdn')"
              placeholder="lemonldapng.example.org"
              v-model.trim="host"
              class="mg-bottom"
              :invalid-message="$t(error.host)"
              :disabled="loading.getConfiguration || loading.configureModule || configured"
              ref="host"
            >
              <template #tooltip>{{ $t("settings.lemonldapng_fqdn_tooltip") }}</template>
            </NsTextInput>

            <!-- toggles -->
            <cv-toggle
              value="letsEncrypt"
              :label="$t('settings.lets_encrypt')"
              v-model="isLetsEncryptEnabled"
              :disabled="loading.getConfiguration || loading.configureModule"
              class="mg-bottom"
            >
              <template slot="text-left">{{ $t("settings.disabled") }}</template>
              <template slot="text-right">{{ $t("settings.enabled") }}</template>
            </cv-toggle>

            <cv-toggle
              value="httpToHttps"
              :label="$t('settings.http_to_https')"
              v-model="isHttpToHttpsEnabled"
              :disabled="loading.getConfiguration || loading.configureModule"
              class="mg-bottom"
            >
              <template slot="text-left">{{ $t("settings.disabled") }}</template>
              <template slot="text-right">{{ $t("settings.enabled") }}</template>
            </cv-toggle>

            <!-- LDAP combo -->
            <NsComboBox
              v-model.trim="ldap_domain"
              :autoFilter="true"
              :autoHighlight="true"
              :title="$t('settings.ldap_domain')"
              :label="$t('settings.choose_ldap_domain')"
              :options="ldap_domain_list"
              :userInputLabel="core.$t('common.user_input_l')"
              :acceptUserInput="false"
              :showItemType="true"
              :invalid-message="$t(error.ldap_domain)"
              :disabled="loading.getConfiguration || loading.configureModule"
              tooltipAlignment="start"
              tooltipDirection="top"
              class="mg-bottom"
              ref="ldap_domain"
            >
              <template slot="tooltip">
                {{ $t("settings.choose_the_ldap_domain_to_authenticate_users") }}
              </template>
            </NsComboBox>

            <!-- advanced options -->
            <cv-accordion ref="accordion" class="maxwidth mg-bottom">
              <cv-accordion-item :open="toggleAccordion[0]">
                <template slot="title">{{ $t("settings.advanced") }}</template>
                <template slot="content">
                  <NsToggle
                    v-model="saml_status"
                    ref="saml_status"
                    :label="$t('settings.saml_status')"
                    :form-item="true"
                    value="toggleValue"
                    :disabled="loading.getConfiguration || loading.configureModule"
                  >
                    <template slot="tooltip">
                      <span>{{ $t("settings.saml_status_tooltip") }}</span>
                    </template>
                    <template slot="text-left">{{ $t("settings.disabled") }}</template>
                    <template slot="text-right">{{ $t("settings.enabled") }}</template>
                  </NsToggle>

                  <NsToggle
                    v-model="cda_status"
                    ref="cda_status"
                    :label="$t('settings.cda_status')"
                    :form-item="true"
                    value="toggleValue"
                    :disabled="loading.getConfiguration || loading.configureModule"
                  >
                    <template slot="tooltip">
                      <span>{{ $t("settings.cda_status_tooltip") }}</span>
                    </template>
                    <template slot="text-left">{{ $t("settings.disabled") }}</template>
                    <template slot="text-right">{{ $t("settings.enabled") }}</template>
                  </NsToggle>

                  <NsToggle
                    v-model="sample_apps_enabled"
                    ref="sample_apps_enabled"
                    :label="$t('settings.sample_apps_enabled')"
                    :form-item="true"
                    value="toggleValue"
                    :disabled="loading.getConfiguration || loading.configureModule"
                  >
                    <template slot="tooltip">
                      <span>{{ $t("settings.sample_apps_enabled_tooltip") }}</span>
                    </template>
                    <template slot="text-left">{{ $t("settings.disabled") }}</template>
                    <template slot="text-right">{{ $t("settings.enabled") }}</template>
                  </NsToggle>
                </template>
              </cv-accordion-item>
            </cv-accordion>

            <!-- errors -->
            <cv-row v-if="error.configureModule">
              <cv-column>
                <NsInlineNotification
                  kind="error"
                  :title="$t('action.configure-module')"
                  :description="error.configureModule"
                  :showCloseButton="false"
                />
              </cv-column>
            </cv-row>

            <!-- submit -->
            <NsButton
              kind="primary"
              :icon="Save20"
              :loading="loading.configureModule"
              :disabled="loading.getConfiguration || loading.configureModule"
            >
              {{ $t("settings.save") }}
            </NsButton>
          </cv-form>
        </cv-tile>
      </cv-column>
    </cv-row>
  </cv-grid>
</template>

<script>
import to from "await-to-js";
import { mapState } from "vuex";
import {
  QueryParamService,
  UtilService,
  TaskService,
  IconService,
  PageTitleService,
} from "@nethserver/ns8-ui-lib";

export default {
  name: "Settings",
  mixins: [TaskService, IconService, UtilService, QueryParamService, PageTitleService],
  pageTitle() {
    return this.$t("settings.title") + " - " + this.appName;
  },
  data() {
    return {

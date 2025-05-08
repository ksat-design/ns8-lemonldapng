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
            <NsTextInput
              :label="$t('settings.lemonldapng_fqdn')"
              placeholder="lemonldapng.example.org"
              v-model.trim="host"
              class="mg-bottom"
              :invalid-message="$t(error.host)"
              :disabled="
                loading.getConfiguration ||
                loading.configureModule ||
                configured
              "
              ref="host"
            >
              <template #tooltip>{{
                $t("settings.lemonldapng_fqdn_tooltip")
              }}</template>
            </NsTextInput>
            <cv-toggle
              value="letsEncrypt"
              :label="$t('settings.lets_encrypt')"
              v-model="isLetsEncryptEnabled"
              :disabled="loading.getConfiguration || loading.configureModule"
              class="mg-bottom"
            >
              <template slot="text-left">{{
                $t("settings.disabled")
              }}</template>
              <template slot="text-right">{{
                $t("settings.enabled")
              }}</template>
            </cv-toggle>
            <cv-toggle
              value="httpToHttps"
              :label="$t('settings.http_to_https')"
              v-model="isHttpToHttpsEnabled"
              :disabled="loading.getConfiguration || loading.configureModule"
              class="mg-bottom"
            >
              <template slot="text-left">{{
                $t("settings.disabled")
              }}</template>
              <template slot="text-right">{{
                $t("settings.enabled")
              }}</template>
            </cv-toggle>
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
                {{
                  $t("settings.choose_the_ldap_domain_to_authenticate_users")
                }}
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
                    :disabled="
                      loading.getConfiguration || loading.configureModule
                    "
                  >
                    <template slot="tooltip">
                      <span>{{ $t("settings.saml_status_tooltip") }}</span>
                    </template>
                    <template slot="text-left"
                      >{{ $t("settings.disabled") }}
                    </template>
                    <template slot="text-right"
                      >{{ $t("settings.enabled") }}
                    </template>
                  </NsToggle>
                  <NsToggle
                    v-model="cda_status"
                    ref="cda_status"
                    :label="$t('settings.cda_status')"
                    :form-item="true"
                    value="toggleValue"
                    :disabled="
                      loading.getConfiguration || loading.configureModule
                    "
                  >
                    <template slot="tooltip">
                      <span>{{ $t("settings.cda_status_tooltip") }}</span>
                    </template>
                    <template slot="text-left"
                      >{{ $t("settings.disabled") }}
                    </template>
                    <template slot="text-right"
                      >{{ $t("settings.enabled") }}
                    </template>
                  </NsToggle>
                  <NsToggle
                    v-model="sample_apps_status"
                    ref="sample_apps_status"
                    :label="$t('settings.sample_apps_status')"
                    :form-item="true"
                    value="toggleValue"
                    :disabled="
                      loading.getConfiguration || loading.configureModule
                    "
                  >
                    <template slot="tooltip">
                      <span>{{ $t("settings.sample_apps_status_tooltip") }}</span>
                    </template>
                    <template slot="text-left"
                      >{{ $t("settings.disabled") }}
                    </template>
                    <template slot="text-right"
                      >{{ $t("settings.enabled") }}
                    </template>
                  </NsToggle>
                </template>
              </cv-accordion-item>
            </cv-accordion>
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
            <NsButton
              kind="primary"
              :icon="Save20"
              :loading="loading.configureModule"
              :disabled="loading.getConfiguration || loading.configureModule"
              >{{ $t("settings.save") }}</NsButton
            >
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
  mixins: [
    TaskService,
    IconService,
    UtilService,
    QueryParamService,
    PageTitleService,
  ],
  pageTitle() {
    return this.$t("settings.title") + " - " + this.appName;
  },
  data() {
    return {
      q: {
        page: "settings",
      },
      saml_status: false,
      cda_status: false,
      sample_apps_status: true,
      urlCheckInterval: null,
      host: "",
      configured: false,
      isLetsEncryptEnabled: false,
      isHttpToHttpsEnabled: true,
      ldap_domain: "",
      ldap_domain_list: [],
      loading: {
        getConfiguration: false,
        configureModule: false,
      },
      error: {
        getConfiguration: "",
        configureModule: "",
        host: "",
        lets_encrypt: "",
        http2https: "",
      },
    };
  },
  computed: {
    ...mapState(["instanceName", "core", "appName"]),
  },
  created() {
    this.getConfiguration();
  },
  beforeRouteEnter(to, from, next) {
    next((vm) => {
      vm.watchQueryData(vm);
      vm.urlCheckInterval = vm.initUrlBindingForApp(vm, vm.q.page);
    });
  },
  beforeRouteLeave(to, from, next) {
    clearInterval(this.urlCheckInterval);
    next();
  },
  methods: {
    async getConfiguration() {
      this.loading.getConfiguration = true;
      this.error.getConfiguration = "";
      const taskAction = "get-configuration";
      const eventId = this.getUuid();

      this.core.$root.$once(
        `${taskAction}-aborted-${eventId}`,
        this.getConfigurationAborted
      );
      this.core.$root.$once(
        `${taskAction}-completed-${eventId}`,
        this.getConfigurationCompleted
      );

      const res = await to(
        this.createModuleTaskForApp(this.instanceName, {
          action: taskAction,
          extra: {
            title: this.$t("action." + taskAction),
            isNotificationHidden: true,
            eventId,
          },
        })
      );
      const err = res[0];

      if (err) {
        console.error(`error creating task ${taskAction}`, err);
        this.error.getConfiguration = this.getErrorMessage(err);
        this.loading.getConfiguration = false;
        return;
      }
    },
    getConfigurationAborted(taskResult, taskContext) {
      console.error(`${taskContext.action} aborted`, taskResult);
      this.error.getConfiguration = this.$t("error.generic_error");
      this.loading.getConfiguration = false;
    },
    getConfigurationCompleted(taskContext, taskResult) {
      const config = taskResult.output;
      this.host = config.host;
      this.isLetsEncryptEnabled = config.lets_encrypt;
      this.isHttpToHttpsEnabled = config.http2https;
      this.configured = config.configured;
      this.saml_status = config.saml_status;
      this.cda_status = config.cda_status;
      this.sample_apps_status =
        config.sample_apps_status != null
          ? config.sample_apps_status
          : this.sample_apps_status;
      this.$nextTick(() => {
        this.ldap_domain = config.ldap_domain;
        if (this.ldap_domain == "") {
          this.ldap_domain = "-";
        }
      });
      this.ldap_domain_list = config.ldap_domain_list;
      this.loading.getConfiguration = false;
      this.focusElement("host");
    },
    validateConfigureModule() {
      this.clearErrors(this);
      let isValidationOk = true;

      if (!this.host) {
        this.error.host = "common.required";
        if (isValidationOk) this.focusElement("host");
        isValidationOk = false;
      }
      if (!this.ldap_domain) {
        this.error.ldap_domain = "common.required";
        if (isValidationOk) this.focusElement("ldap_domain");
        isValidationOk = false;
      }
      return isValidationOk;
    },
    configureModuleValidationFailed(validationErrors) {
      this.loading.configureModule = false;
      let focusAlreadySet = false;

      for (const validationError of validationErrors) {
        const param = validationError.parameter;
        this.error[param] = this.$t("settings." + validationError.error);
        if (!focusAlreadySet) {
          this.focusElement(param);
          focusAlreadySet = true;
        }
      }
    },
    async configureModule() {
      this.error.test_imap = false;
      this.error.test_smtp = false;
      const isValidationOk = this.validateConfigureModule();
      if (!isValidationOk) return;

      this.loading.configureModule = true;
      const taskAction = "configure-module";
      const eventId = this.getUuid();

      this.core.$root.$once(
        `${taskAction}-aborted-${eventId}`,
        this.configureModuleAborted
      );
      this.core.$root.$once(
        `${taskAction}-validation-failed-${eventId}`,
        this.configureModuleValidationFailed
      );
      this.core.$root.$once(
        `${taskAction}-completed-${eventId}`,
        this.configureModuleCompleted
      );

      const res = await to(
        this.createModuleTaskForApp(this.instanceName, {
          action: taskAction,
          data: {
            host: this.host,
            lets_encrypt: this.isLetsEncryptEnabled,
            http2https: this.isHttpToHttpsEnabled,
            ldap_domain: this.ldap_domain == "-" ? "" : this.ldap_domain,
            saml_status: this.saml_status,
            cda_status: this.cda_status,
            sample_apps_status: this.sample_apps_status, // included in submission
          },
          extra: {
            title: this.$t("settings.instance_configuration", {
              instance: this.instanceName,
            }),
            description: this.$t("settings.configuring"),
            eventId,
          },
        })
      );
      const err = res[0];

      if (err) {
        console.error(`error creating task ${taskAction}`, err);
        this.error.configureModule = this.getErrorMessage(err);
        this.loading.configureModule = false;
        return;
      }
    },
    configureModuleAborted(taskResult, taskContext) {
      console.error(`${taskContext.action} aborted`, taskResult);
      this.error.configureModule = this.$t("error.generic_error");
      this.loading.configureModule = false;
    },
    configureModuleCompleted() {
      this.loading.configureModule = false;
      this.getConfiguration();
    },
  },
};
</script>

<style scoped lang="scss">
@import "../styles/carbon-utils";
.mg-bottom {
  margin-bottom: $spacing-06;
}
.maxwidth {
  max-width: 38rem;
}
</style>

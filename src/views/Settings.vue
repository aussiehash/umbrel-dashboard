<template>
  <div class="p-sm-2">
    <div class="my-3 pb-2">
      <h1>settings</h1>
      <!-- <p class="text-muted">Your umbrel, your rules.</p> -->
    </div>
    <b-row>
      <b-col col cols="12" md="6" xl="4">
        <card-widget
          header="Tor"
          :status="{ text: 'Running', variant: 'success', blink: false }"
          title="100%"
          sub-title="Traffic relayed through Tor"
          icon="icon-app-tor.svg"
        >
          <div class="pt-2">
            <div class="d-flex w-100 justify-content-between px-3 px-lg-4 mb-4">
              <div>
                <span class="d-block">Bitcoin</span>
                <small class="d-block" style="opacity: 0.4">Run Bitcoin Core on Tor</small>
              </div>
              <toggle-switch class="align-self-center"></toggle-switch>
            </div>
            <div class="d-flex w-100 justify-content-between px-3 px-lg-4 mb-4">
              <div>
                <span class="d-block">Lightning Network</span>
                <small class="d-block" style="opacity: 0.4">Run Lightning on Tor</small>
              </div>
              <toggle-switch class="align-self-center"></toggle-switch>
            </div>
            <div class="px-3 px-lg-4 mb-4">
              <div class="d-flex justify-content-between w-100 mb-3">
                <div>
                  <span class="d-block">Remote Access</span>
                  <small class="d-block" style="opacity: 0.4">Remotely access your Umbrel on Tor</small>
                </div>
                <toggle-switch class="align-self-center"></toggle-switch>
              </div>
              <input-copy class="w-100" size="sm" :value="onionAddress"></input-copy>
            </div>
            <div class="px-3 px-lg-4 py-2"></div>
          </div>
        </card-widget>
      </b-col>
      <b-col col cols="12" md="6" xl="4">
        <card-widget header="Change password" :loading="isChangingPassword">
          <div class="px-4 pb-2">
            <label class="sr-onlsy" for="input-withdrawal-amount">Current password</label>
            <input-password
              v-model="currentPassword"
              ref="password"
              inputGroupClass="neu-input-group"
              :inputClass="[
                isIncorrectPassword ? 'incorrect-password' : '',
                'form-control form-control-lg neu-input w-100'
              ]"
              :disabled="isChangingPassword"
            />
            <div class="py-2"></div>
            <label class="sr-onlsy" for="input-withdrawal-amount">New password</label>
            <input-password
              v-model="newPassword"
              ref="password"
              inputGroupClass="neu-input-group"
              inputClass="form-control form-control-lg neu-input w-100"
              :disabled="isChangingPassword"
            />
            <div class="py-2"></div>
            <label class="sr-onlsy" for="input-withdrawal-amount">Confirm new password</label>
            <input-password
              v-model="confirmNewPassword"
              ref="password"
              inputGroupClass="neu-input-group"
              inputClass="form-control form-control-lg neu-input w-100"
              :disabled="isChangingPassword"
            />
            <div class="py-2"></div>
            <b-alert variant="warning" show>
              <small>
                ⚠ Remember, there is no "Forgot Password" button. If you lose
                your password, you will have to recover your node using your 24
                secret words.
              </small>
            </b-alert>
          </div>
          <b-button
            class="w-100"
            variant="success"
            style="border-radius: 0; border-bottom-left-radius: 1rem; border-bottom-right-radius: 1rem; padding-top: 1rem; padding-bottom: 1rem;"
            :disabled="isChangingPassword || !isAllowedToChangePassword"
            @click="changePassword"
          >{{ isChangingPassword ? 'Changing password...' : 'Change password'}}</b-button>
        </card-widget>
      </b-col>
    </b-row>
  </div>
</template>

<script>
import { mapState } from "vuex";
import API from "@/helpers/api";

import CardWidget from "@/components/CardWidget";
import ToggleSwitch from "@/components/ToggleSwitch";
import InputPassword from "@/components/Utility/InputPassword";
import InputCopy from "@/components/Utility/InputCopy";

export default {
  data() {
    return {
      currentPassword: "",
      isIncorrectPassword: false,
      newPassword: "",
      confirmNewPassword: "",
      isChangingPassword: false
    };
  },
  computed: {
    ...mapState({
      onionAddress: state => state.system.onionAddress
    }),
    isAllowedToChangePassword() {
      if (!this.currentPassword) {
        return false;
      }
      if (this.newPassword.length < 12) {
        return false;
      }
      if (this.newPassword !== this.confirmNewPassword) {
        return false;
      }
      if (this.currentPassword === this.newPassword) {
        return false;
      }
      return true;
    }
  },
  created() {
    this.$store.dispatch("system/getOnionAddress");
  },
  methods: {
    async changePassword() {
      // disable on testnet
      if (window.location.hostname === "testnet.getumbrel.com") {
        return this.$bvToast.toast('y u try to do dis on testnet :"(', {
          title: "Error",
          autoHideDelay: 3000,
          variant: "danger",
          solid: true,
          toaster: "b-toaster-bottom-right"
        });
      }

      const payload = {
        password: this.currentPassword,
        newPassword: this.newPassword
      };

      this.isChangingPassword = true;

      try {
        await API.post(
          `${process.env.VUE_APP_MANAGER_API_URL}/v1/account/change-password`,
          payload,
          false
        );
      } catch (error) {
        if (error.response && error.response.data) {
          this.$bvToast.toast(error.response.data, {
            title: "Error",
            autoHideDelay: 3000,
            variant: "danger",
            solid: true,
            toaster: "b-toaster-bottom-right"
          });
          if (error.response.data === "Incorrect password") {
            this.isIncorrectPassword = true;
          }
        }
        this.isChangingPassword = false;
        return;
      }

      this.$bvToast.toast(
        `You've successfully changed your Umbrel's password`,
        {
          title: "Password Changed",
          autoHideDelay: 3000,
          variant: "success",
          solid: true,
          toaster: "b-toaster-bottom-right"
        }
      );

      this.isChangingPassword = false;

      // Remove passwords from view
      this.currentPassword = "";
      this.newPassword = "";
      this.confirmNewPassword = "";
    }
  },
  watch: {
    currentPassword: function() {
      this.isIncorrectPassword = false;
    }
  },
  components: {
    CardWidget,
    ToggleSwitch,
    InputPassword,
    InputCopy
  }
};
</script>

<style lang="scss" scoped></style>

<template>
  <div id="config">
    <div class="title">
      {{ $t("app.config.gameSettings" /* Game Settings */) }}
    </div>
    <div class="block">
      <file-select :path.sync="config.wowpath.value"></file-select>
      <img
        v-if="config.wowpath.valided"
        class="green"
        :src="require(`@/assets/ok.png`)"
      />
      <img v-else class="red" :src="require(`@/assets/error.png`)" />
      <span v-if="config.wowpath.valided">
        <p class="label">
          {{ $t("app.config.selectAccount" /* Select Account */) }}
        </p>
        <select v-model="config.account.value" class="form-control">
          <option v-for="item in config.account.choices" :key="item.name">{{
            item.name
          }}</option>
        </select>
        <img
          v-if="config.account.valided"
          class="green"
          :src="require(`@/assets/ok.png`)"
        />
        <img v-else class="red" :src="require(`@/assets/error.png`)" />
      </span>
    </div>
    <div class="title">
      {{ $t("app.config.wagoSettings" /* Wago Settings */) }}
    </div>
    <div class="block">
      <p class="label">
        {{ $t("app.config.wagoAccount" /* Wago Account (optional) */) }}
      </p>
      <input type="text" v-model="wagoUsername" size="11" />
      <v-button @click="config.wagoUsername = wagoUsername">{{
        $t("app.config.ok" /* Ok */)
      }}</v-button>
      <img
        v-if="config.wagoUsername"
        class="green"
        :src="require(`@/assets/ok.png`)"
      />
      <br /><br />
      <input
        id="ignoreOwnAuras"
        type="checkbox"
        v-model="config.ignoreOwnAuras"
      />
      {{ $t("app.config.ignoreOwnAuras" /* Ignore auras from your account */) }}
    </div>
    <div class="title">
      {{ $t("app.config.clientSettings" /* Client Settings */) }}
    </div>
    <div class="block">
      <p class="label">{{ $t("app.config.lang" /* Language */) }}</p>
      <select v-model="config.lang" class="form-control language">
        <option
          v-for="lang in langs"
          :value="lang.value"
          v-html="lang.text"
          :key="lang.value"
        ></option> </select
      ><br /><br />
      <input type="checkbox" id="update-notification" v-model="config.notify" />
      <label for="update-notification">
        {{
          $t(
            "app.config.notification" /* Receive a notification when auras get updated */
          )
        }}
      </label>
      <br /><br />
      <p class="label subtitle">{{ $t("app.config.startup" /* Startup */) }}</p>
      <div class="option">
        <input id="autostart" type="checkbox" v-model="config.autostart" />
        <label for="autostart">
          {{
            $t("app.config.autoStart" /* Launch client with your computer */)
          }}
        </label>
      </div>
      <div class="option">
        <input
          id="startminimize"
          type="checkbox"
          v-model="config.startminimize"
        />
        <label for="startminimize">
          {{ $t("app.config.minimized" /* Start client minimized */) }}
        </label>
      </div>
    </div>
    <br /><br />
    <v-button @click="reset" type="reset">{{
      $t("app.config.reset" /* Reset Settings and Data */)
    }}</v-button>
    <br /><br />
  </div>
</template>

<script>
import path from "path";
import FileSelect from "./FileSelect.vue";
import Button from "./Button.vue";

const fs = require("fs");
const AutoLaunch = require("auto-launch");

const AutoLauncher = new AutoLaunch({
  name: "WeakAuras Companion"
});

export default {
  props: ["config"],
  data() {
    return {
      langs: [
        { value: "de", text: "Deutsch (de)" },
        { value: "en", text: "English (en)" },
        { value: "fr", text: "Français (fr)" },
        { value: "ru", text: "Русский (ru)" }
      ],
      wagoUsername: this.config.wagoUsername
    };
  },
  components: { FileSelect, "v-button": Button },
  methods: {
    reset() {
      this.$parent.reset();
      this.wagoUsername = null;
    }
  },
  watch: {
    // eslint-disable-next-line func-names
    "config.autostart": function() {
      if (this.config.autostart) {
        AutoLauncher.enable();
      } else {
        AutoLauncher.disable();
      }
    },
    // eslint-disable-next-line func-names
    "config.wowpath.value": function() {
      if (this.config.wowpath.value) {
        // clean Accounts options
        while (this.config.account.choices.length > 0) {
          this.config.account.choices.pop();
        }
        // test if ${wowpath}\WTF\Account exists
        const accountFolder = path.join(
          this.config.wowpath.value,
          "WTF",
          "Account"
        );
        fs.access(accountFolder, fs.constants.F_OK, err => {
          if (!err) {
            // add option for each account found
            fs.readdirSync(accountFolder).forEach(file => {
              if (file !== "SavedVariables") {
                this.config.account.choices.push({ name: file });
                this.config.wowpath.valided = true;
              }
            });
          } else {
            this.config.wowpath.valided = false;
          }
        });
      }
    },
    // eslint-disable-next-line func-names
    "config.account.value": function() {
      if (this.config.wowpath.valided && !!this.config.account.value) {
        const WeakAurasSavedVariable = path.join(
          this.config.wowpath.value,
          "WTF",
          "Account",
          this.config.account.value,
          "SavedVariables",
          "WeakAuras.lua"
        );
        fs.access(WeakAurasSavedVariable, fs.constants.F_OK, err => {
          if (!err) {
            this.config.account.valided = true;
          } else {
            this.config.account.valided = false;
          }
        });
      }
    },
    // eslint-disable-next-line func-names
    "config.lang": function() {
      this.$i18n.locale = this.config.lang;
    }
  }
};
</script>

<style scoped>
#config {
  padding: 5px 0 5px 30px;
  text-align: left;
  overflow: auto;
  height: 100%;
  width: 100%;
}
label,
.label {
  color: #eee;
  margin: 2px 0 3px;
  font-size: 14px;
}
.red,
.green {
  border-radius: 2px;
  width: 27px;
  height: 27px;
  vertical-align: top;
}
input,
select,
.fakeinput {
  padding: 5px;
  font-size: small;
  border-radius: 2px;
  border: none;
  background-color: #eee;
  color: #1e1e1e;
}
#config .title:first-child {
  margin-top: 5px;
}
.title {
  font-size: 25px;
  margin: 25px 0 10px;
  font-weight: 600;
  padding: 4px 5px 4px;
  border-left: 2px solid rgb(255, 209, 0);
  background-color: rgba(0, 0, 0, 0.1);
}
.block {
  margin-left: 10px;
  font-size: 15px;
}

.option {
  margin-bottom: 5px;
}

/* Checkboxes */
input[type="checkbox"] {
  position: relative;
  cursor: pointer;
  padding: 0;
  margin-right: 8px;
}

input[type="checkbox"]:before {
  content: "";
  margin-right: 10px;
  display: inline-block;
  vertical-align: text-top;
  position: relative;
  bottom: 1px;
  right: 1px;
  border-radius: 2px;
  width: 20px;
  height: 20px;
  background: #e7e7e7;
}

input[type="checkbox"]:hover:before {
  background: #e7e7e7;
}

input[type="checkbox"]:focus:before {
  box-shadow: 0 0 0 3px rgba(0, 0, 0, 0.12);
}

input[type="checkbox"]:checked:before {
  background: #2b2b2b;
}

input[type="checkbox"]:disabled {
  color: #1d1d1d;
  cursor: auto;
}

input[type="checkbox"]:disabled:before {
  box-shadow: none;
  background: #ddd;
}

input[type="checkbox"]:checked:after {
  content: "";
  position: absolute;
  background-image: url("~@/assets/checkmark.png");
  background-size: contain;
  width: 15px;
  height: 15px;
  left: 2px;
  top: 2px;
}
input[type="checkbox"] + label {
  margin-bottom: 5px;
  padding-bottom: 5px;
}

.subtitle {
  font-size: 18px;
  margin-bottom: 5px;
  font-weight: 600;
  color: white;
}

.form-control.language {
  width: 110px;
}
</style>

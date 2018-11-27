<template>
  <div id="config">
    <div class="title">Game Settings</div>
    <div class="block">
      <file-select :path.sync="config.wowpath.value"></file-select>
      <span v-if="config.wowpath.value">
        <span v-if="config.wowpath.valided" class="green">✔</span>
        <span v-else class="red">✘</span>
      </span>
      <span v-if="config.wowpath.valided">
        <p class="label">Select account</p>
        <select v-model="config.account.value" class="form-control">
          <option v-for="item in config.account.choices" :key="item.name">{{ item.name }}</option>
        </select>
        <span v-if="config.account.value">
          <span v-if="config.account.valided" class="green">✔</span>
          <span v-else class="red">✘</span>
        </span>
      </span>
    </div>
    <div class="title">Wago Settings</div>
    <div class="block">
      <p class="label">Wago account (optional)</p>
      <input type="text" v-model="config.wagoUsername" size="11">
    </div>
    <div class="title">Addon Settings</div>
    <div class="block">
      <input type="checkbox" v-model="config.notify"> Receive a notification when an aura is updated
      <br>
      <br>
      <p class="label">Startup</p>
      <input type="checkbox" v-model="config.autostart"> Launch client with your computer
      <br>
      <input type="checkbox" v-model="config.startminimize"> Start client minimized
    </div>
    <br>
    <br>
    <v-button @click="reset" type="info">Reset Settings and Data</v-button>
    <br>
    <br>
  </div>
</template>

<script>
import FileSelect from "./FileSelect";
import Button from './Button'

const fs = require("fs");
import path from "path";
const AutoLaunch = require("auto-launch");
var AutoLauncher = new AutoLaunch({
  name: "WeakAuras Wago Updater"
});

export default {
  props: ['config'],
  components: { FileSelect , 'v-button': Button },
  methods: {
    reset(){
      this.$parent.reset()
    }
  },
  watch: {
    "config.autostart"() {
      if (this.config.autostart) {
        AutoLauncher.enable();
      } else {
        AutoLauncher.disable();
      }
    },
    "config.wowpath.value"() {
      if (!!this.config.wowpath.value) {
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
              if (file != "SavedVariables") {
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
    "config.account.value"() {
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
    }
  }
}
</script>

<style scoped>
#config {
  margin: 5px 50px;
  text-align: left;
}
.label {
  color: gray;
}
input,
select,
.fakeinput {
  padding: 2px;
  font-size: small;
}
.title {
  font-size: 20px;
  margin: 10px 0px;
}
.block {
  margin-left: 30px;
}
</style>
<template>
  <div class="aura">
    <a
      class="wago_icon"
      target="_blank"
      :href="aura.slug | wago"
      :title="aura.slug | wago"
    />
    <div class="aura_name_container">
      <span class="aura_name" :title="childs" v-html="aura.name" />
    </div>
    <a
      class="author"
      target="_blank"
      :href="aura.author | wagoAuthor"
      :title="aura.author | wagoAuthor"
      v-html="aura.author"
    />
    <div class="upgrade-text">
      <div class="current-version">
        <!-- {{ $t("app.aura.currentversion" /* Current: */) }}-->
        v{{ aura.version }}
      </div>
      <div class="wago-version" :title="aura.modified | fromNow($i18n.locale)">
        v{{ aura.wagoVersion }}
      </div>
    </div>
  </div>
</template>

<script>
import moment from "moment";

export default {
  props: ["aura"],
  filters: {
    wago: value => {
      if (!value) return "";
      return `https://wago.io/${value}`;
    },
    wagoAuthor: value => {
      if (!value) return "";
      return `https://wago.io/p/${value}`;
    },
    fromNow: (value, locale) => {
      if (!value) return "n/a";
      return moment(value)
        .locale(locale)
        .fromNow();
    }
  },
  computed: {
    childs() {
      return this.aura.ids.join("\n");
    }
  }
};
</script>

<style scoped>
.aura {
  background-color: #1d1d1d;
  color: rgb(255, 209, 0);
  margin: 1px;
  height: 32px;
  padding: 2px 10px;
  vertical-align: middle;
  white-space: nowrap;
  display: flex;
  overflow: hidden;
  font-size: 11px;
  flex-direction: row;
}
.aura_name_container {
  text-align: left;
  flex: 1;
  overflow: hidden;
}
.aura_name {
  width: 95%;
  overflow: hidden;
  text-overflow: ellipsis;
  padding-left: 10px;
  display: inline-block;
  font-weight: 600;
  font-size: 12px;
  text-shadow: #000000 0 1px 0;
  text-align: left;
  line-height: 32px;
}
.wago_icon {
  height: 100%;
  vertical-align: middle;
  display: inline-block;
}
.wago_icon {
  content: url("~@/assets/wago_plain.png");
}
.author {
  width: 100px;
  overflow: hidden;
  text-overflow: ellipsis;
  padding-left: 10px;
  font-size: 12px;
  text-align: left;
  line-height: 32px;
}
.author:before {
  content: "@";
}
.upgrade-text {
  font-weight: 600;
  text-shadow: #000000 0 1px 0;
  width: 50px;
  display: flex;
}
.current-version {
  font-size: 9px;
  color: #777;
  width: 70px;
  line-height: 32px;
  text-align: left;
  padding-top: 1px;
}
.wago-version {
  text-shadow: rgba(219, 185, 50, 0.267) 0 0 5px;
  text-align: right;
  width: 30px;
  line-height: 32px;
}
</style>

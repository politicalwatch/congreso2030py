<template>
  <div v-if="deputy" id="deputy" class="u-margin-bottom-10">
    <tipi-deputy v-if="deputy" :deputy="deputy" :parliamentaryGroup="parliamentarygroup">
      <a v-if="deputy.hasOwnProperty('twitter')" :href="deputy.twitter" target="_blank"><tipi-icon icon="twitter" /> @{{ deputy.twitter.split('/').reverse()[0] }}</a>
      <a v-if="deputy.hasOwnProperty('email')" :href="`mailto:${deputy.email}`" target="_blank"><tipi-icon icon="mail" /> {{deputy.email}}</a>
      <a v-if="deputy.hasOwnProperty('url')" :href="deputy.url" target="_blank"><tipi-icon icon="building" /> Ver en SILpy</a>
    </tipi-deputy>
    <div class="o-container" v-if="!deputy.active">
      <tipi-message type="info" icon>
        Causó baja en el Congreso
      </tipi-message>
    </div>
    <div v-if="latestInitiatives && latestInitiatives.length" class="o-container o-section">
      <h4 class="u-margin-bottom-4">Últimos expedientes</h4>
      <tipi-results layout="tiny" :initiatives="latestInitiatives" :topicsStyles="styles.topics"/>
    </div>
    <div class="o-container" v-else>
      <tipi-message type="info" icon>
        Sin actividad parlamentaria relacionada con la Agenda 2030
      </tipi-message>
    </div>
  </div>
  <div v-else class="o-container o-section u-margin-bottom-10">
    <tipi-loader title="Cargando datos" subtitle="Puede llevar unos segundos"/>
  </div>
</template>

<script>

import { TipiHeader, TipiDeputy, TipiMessage, TipiResults, TipiIcon, TipiLoader } from 'tipi-uikit'
import api from '@/api';
import config from '@/config';
import { mapState } from 'vuex';

export default {
  name: 'deputy',
  components: {
    TipiHeader,
    TipiDeputy,
    TipiMessage,
    TipiResults,
    TipiIcon,
    TipiLoader,
  },
  data: function() {
    return {
      deputy: null,
      parliamentarygroup: null,
      latestInitiatives: null,
      styles: config.STYLES,
    }
  },
  computed: {
    ...mapState(['allParliamentaryGroups'])
  },
  methods: {
    getDeputy: function() {
      api.getDeputy(this.$route.params.id)
        .then(response => {
          this.deputy = response;
          this.parliamentarygroup = this.allParliamentaryGroups.find(allPG => allPG.shortname === this.deputy.parliamentarygroup);
          this.getLatestInitiatives();
        })
        .catch(error => {
          this.errors = error
          this.$router.push({name: 'Page404', params: { '0': '404'}});
        });
    },
    getLatestInitiatives: function() {
      api.getInitiatives({ 'deputy': this.deputy.name + ' [' + this.deputy.id + ']', 'per_page': 12 })
        .then(response => {
          if (response.initiatives) this.latestInitiatives = response.initiatives;
        })
        .catch(error => this.errors = error);
    }
  },
  created: function() {
    this.getDeputy()
  }
}
</script>

<style scoped lang="scss">
</style>

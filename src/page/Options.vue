<template>
<div>
  <v-layout>
    <v-flex xs12 sm6 offset-sm3>
      <div v-for="(optionsList, cate) in optionsLists" :key="cate">

        <v-subheader>{{ __('ui_options_' + cate) }}</v-subheader>
        <v-card>
          <v-list>
            <template v-for="(option, optionIndex) in optionsList">
              <v-list-tile>
                <v-list-tile-content>
                  <v-layout wrap style="width:100%">
                    <v-flex xs8>
                      <v-subheader>
                        {{ option.desc }}
                      </v-subheader>
                    </v-flex>
                    <v-flex xs4>
                      <v-select
                        class="select-amend"
                        v-if="option.type === String"
                        :items="option.items"
                        :value="opts[option.name]"
                        label=""
                        item-text="label"
                        item-value="value"
                        @change="optionsChanged(option.name, $event)"
                      ></v-select>
                      <v-switch
                        class="switch-amend"
                        v-if="option.type === Boolean"
                        v-model="opts[option.name]"
                        @change="optionsChanged(option.name, $event)"
                      ></v-switch>
                    </v-flex>
                  </v-layout>
                </v-list-tile-content>
              </v-list-tile>
              <v-divider v-if="optionIndex !== optionsList.length - 1"></v-divider>
            </template>
          </v-list>
        </v-card>
        <!-- loop render end -->
      </div>

      <v-subheader>{{ __('ui_options_sync') }}</v-subheader>
      <v-card>
        <v-list>
          <v-list-tile>
            <v-list-tile-content>

              <v-subheader>
                Sync
                <v-tooltip top>
                  <v-chip slot="activator" outline color="red" small>BETA</v-chip>
                  <span>{{ __('ui_beta_warn') }}</span>
                </v-tooltip>
              </v-subheader>
            </v-list-tile-content>
            <v-list-tile-action>
              <v-btn icon ripple :to="'/app/options/sync'">
                <v-icon color="grey lighten-1">arrow_forward</v-icon>
              </v-btn>
            </v-list-tile-action>
          </v-list-tile>
        </v-list>
      </v-card>
    </v-flex>
  </v-layout>
  <v-snackbar
    :timeout="2000"
    bottom
    v-model="snackbar"
  >
    {{ __('ui_opt_changes_saved') }}
  </v-snackbar>
</div>
</template>
<script>
import storage from '@/common/storage'
import options from '@/common/options'
import __ from '@/common/i18n'
import _ from 'lodash'
import browser from 'webextension-polyfill'
import {formatTime} from '@/common/utils'
import {mapState, mapMutations} from 'vuex'

export default {
  data() {
    return {
      optionsLists: _.groupBy(options.optionsList, 'cate'),
      snackbar: false,
    }
  },
  computed: {
    ...mapState(['opts']),
  },
  created() {
    this.init()
  },
  methods: {
    __,
    formatTime,
    ...mapMutations(['setOption']),
    emitChanges: _.debounce(async function (key, value) {
      console.log(1)
      console.log(key, value)
      // when type of option is string options can not be set correctly after first storage.setOptions() called
      await storage.setOptions(this.opts)
      await storage.setOptions(this.opts)
      console.log(2)
      chrome.runtime.sendMessage({optionsChanged: {[key]: value}})
    }, 100),
    optionsChanged(key, value) {
      this.setOption({[key]: value})
      this.emitChanges(key, value)
    },
    async init() {
      chrome.runtime.onMessage.addListener(msg => {
        if (msg.optionsChangeHandledStatus === 'success') {
          this.snackbar = true
        }
      })
    }
  }
}
</script>
<style lang="scss">
.select-amend {
  padding: 4px 0 0;
}
.switch-amend {
  height: 100%;
  div {
    height: 100%;
  }
}
</style>

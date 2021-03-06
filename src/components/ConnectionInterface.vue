<template>
  <div class="interface connection-interface">
    <div class="interface-wrap row">
      <sidebar class="connection-sidebar" ref="sidebar">
        <connection-sidebar :defaultConfig="defaultConfig" :selectedConfig="config" @edit="edit" @connect="handleConnect"></connection-sidebar>
      </sidebar>
      <div ref="content" class="connection-main page-content" id="page-content">
        <div class="small-wrap">
          <div class="card-flat padding">
            <h3 class="card-title">{{pageTitle}}</h3>
            <div class="alert alert-danger" v-show="errors">
              <i class="material-icons">warning</i>
              <div>
                <span>Please fix the following errors:</span>
                <ul>
                  <li v-for="(e, i) in errors" :key="i">{{e}}</li>
                </ul>
              </div>
            </div>
            <form @action="submit" v-if="config">
              <div class="form-group">
                <label for="connectionType">Connection Type</label>
                <select name="connectionType" class="form-control custom-select" v-model="config.connectionType" @change="changeType" id="connection-select">
                  <option disabled value="null">Select a connection type...</option>
                  <option :key="t.value" v-for="t in connectionTypes" :value="t.value">{{t.name}}</option>
                </select>
              </div>
              <div v-if="config.connectionType">
                <!-- INDIVIDUAL DB CONFIGS -->
                <mysql-form v-if="['mysql', 'mariadb'].includes(config.connectionType)" :config="config" :testing="testing" @save="save" @test="testConnection" @connect="submit"></mysql-form>
                <postgres-form v-if="config.connectionType === 'postgresql'" :config="config" :testing="testing"></postgres-form>
                <postgres-form v-if="config.connectionType === 'redshift'" :config="config" :testing="testing"></postgres-form>
                <sqlite-form v-if="config.connectionType === 'sqlite'" :config="config" :testing="testing"></sqlite-form>
                <sql-server-form v-if="config.connectionType === 'sqlserver'" :config="config" :testing="testing"></sql-server-form>

                <!-- TEST AND CONNECT -->
                <div class="row flex-middle">
                  <span class="expand"></span>
                  <div class="btn-group">
                    <button :disabled="testing" class="btn btn-flat" @click.prevent="testConnection">Test</button>
                    <button :disabled="testing" class="btn btn-primary" @click.prevent="submit">Connect</button>
                  </div>
                </div>

                <!-- Save Connection -->
                <div class="save-connection expand">
                  <h3>Save Connection</h3>
                  <div class="row">
                    <div class="expand"><input class="form-control" @keydown.enter.prevent.stop="save" type="text" v-model="config.name" placeholder="Connection Name"></div>
                    <div><button class="btn btn-flat" @click.prevent="save">Save</button></div>
                  </div>

                  <!-- TODO MATTHEW: Not sure if this breaks this -->
                  <label for="rememberPassword" class="checkbox-group row">
                    <input id="rememberPassword" type="checkbox" name="rememberPassword" class="form-control" v-model="config.rememberPassword">
                    <span>Save Passwords</span>
                    <i class="material-icons" v-tooltip="'Passwords are encrypted when saved'">help_outlined</i>
                  </label>
                </div>
              </div>

            </form>

          </div>
          <div v-if="connectionError" class="alert alert-danger">
            {{connectionError}}
          </div>

        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import os from 'os'
  import config from '../config'
  import {SavedConnection} from '../entity/saved_connection'
  import ConnectionSidebar from './ConnectionSidebar'
  import MysqlForm from './connection/MysqlForm'
  import PostgresForm from './connection/PostgresForm'
  import Sidebar from './Sidebar'
  import SqliteForm from './connection/SqliteForm'
  import SqlServerForm from './connection/SqlServerForm'
  import Split from 'split.js'
  import _ from 'lodash'

  export default {
    components: { ConnectionSidebar, MysqlForm, PostgresForm, Sidebar, SqliteForm, SqlServerForm },
    data() {
      return {
        defaultConfig: new SavedConnection(),
        config: null,
        errors: null,
        connectionError: null,
        testing: false,
        split: null
      }
    },
    computed: {
      connectionTypes() {
        return config.defaults.connectionTypes
      },
      pageTitle() {
        if(_.isNull(this.config) || _.isUndefined(this.config.id)) {
          return "Quick Connect"
        } else {
          return this.config.name
        }
      }
    },
    mounted() {
      this.config = this.defaultConfig
      this.config.sshUsername = os.userInfo().username
      this.$store.dispatch('loadSavedConfigs')
      this.$store.dispatch('fetchUsername')
      this.$nextTick(() => {
        const components = [
          this.$refs.sidebar.$refs.sidebar,
          this.$refs.content
        ]
        this.split = Split(components, {
          elementStyle: (dimension, size) => ({
              'flex-basis': `calc(${size}%)`,
          }),
          sizes: [25,75],
          gutterize: 8,
        })
      })
    },
    beforeDestroy() {
      if(this.split) {
        console.log("destroying split")
        this.split.destroy()
      }
    },
    methods: {
      edit(config) {
        this.config = config
      },
      changeType() {
        if(['mysql', 'mariadb'].includes(this.config.connectionType)) {
          this.config.port = 3306
        } else if(this.config.connectionType === 'postgresql') {
          this.config.port = 5432
        } else if(this.config.connectionType === 'sqlserver') {
          this.config.port = 1433
        }
      },
      async submit() {
        this.connectionError = null
        try {
          await this.$store.dispatch('connect', this.config)
        } catch(ex) {
          this.connectionError = ex.message
          this.$noty.error("Error establishing a connection")
        }
      },
      async handleConnect(config) {
        this.config = config
        await this.submit()
      },
      async testConnection(){

        try {
          this.testing = true
          this.connectionError = null
          await this.$store.dispatch('test', this.config)
          this.$noty.success("Connection looks good!")
          return true
        } catch(ex) {
          this.connectionError = ex.message
          this.$noty.error("Error establishing a connection")
        } finally {
          this.testing = false
        }
      },
      clearForm(){

      },
      async save() {
        try {
          this.errors = null
          this.connectionError = null
          this.$store.dispatch('saveConnectionConfig', this.config)
          if(this.config === this.defaultConfig) {
            this.defaultConfig = new SavedConnection()
          }
          this.$noty.success("Connection Information Saved")
        } catch (ex) {
          this.errors = [ex.message]
          this.$noty.error("Could not save connection information")
        }
      }
    },


  }
</script>

<template>
  <div class="connection-interface">
    <connection-sidebar @edit="edit"></connection-sidebar>
    <div class="connection-main">
      <div class="container">
        <div class="row justify-content-sm-center">
          <div class="col-lg-9 col-md-10 col-xl-6">
            <div class="card mt-5">
              <div class="card-body">
                <h5 class="card-title">Enter Connection Information</h5>
                <form @action="submit">
                  <div class="form-group">
                    <label for="connectionType">Connection Type</label>
                    <select name="connectionType" @change="typeChanged" class="form-control custom-select" v-model="config.connectionType" id="connection-select">
                      <option disabled value="null">Select a connection type...</option>
                      <option :key="t.value" v-for="t in connectionTypes" :value="t.value">{{t.name}}</option>
                    </select>
                  </div>
                  <div v-if="config.connectionType" class="with-connection-type">
                    <div class="row">
                      <div class="form-group col-lg-9">
                        <label for="Host">Host</label>
                        <input type="text" class="form-control" name="host" v-model="config.host">
                      </div>
                      <div class="form-group col-lg-3">
                        <label for="port">Port</label>
                        <input type="number" class="form-control" name="port" v-model="config.port">
                      </div>
                    </div>
                    <div class="row">
                      <div class="form-group col-lg-6">
                        <label for="user">User</label><input type="text" name="user" v-model="config.user" class="form-control">
                      </div>
                      <div class="form-group col-lg-6">
                        <label for="password">Password</label><input type="password" v-model="config.password" class="form-control">
                      </div>

                    </div>
                    <div class="form-group">
                      <label for="defaultDatabase">Default Database</label>
                      <input type="text" class="form-control" v-model="config.defaultDatabase">
                    </div>


                    <div class="text-right">
                      <button class="btn btn-success mr-2" @click="testConnection">Test</button>
                      <button class="btn btn-info" @click.prevent="submit">Connect</button>
                    </div>

                    <h5 class="card-title">Save the Connection</h5>
                    <div class="row">
                      <div class="col-lg-8"><input type="text" v-model="config.name" placeholder="Connection Name" class="form-control"></div>
                      <div class="col-lg-4 text-lg-right"><button class="btn btn-primary" @click.prevent="save">Save</button></div>
                    </div>
                  </div>
                </form>

              </div>


            </div>

          </div>
        </div>
        <div class="row justify-content-sm-center mt-5">
          <div class="col-lg-9 col-md-10 col-xl-6">
            <div class="card">
              <div class="card-body">
                <h5 class="card-title">Recent Connections</h5>
              </div>

            </div>
          </div>
        </div>
      </div>
    </div>

  </div>

</template>

<script>

  import config from '../config'
  import { mapActions } from 'vuex'

  import ConnectionSidebar from './ConnectionSidebar'

  export default {
    components: { ConnectionSidebar },
    data() {
      return {
        config: config.defaults.connectionConfig,
        errors: null
      }
    },
    methods: {
      ...mapActions(['saveConnectionConfig']),
      edit(config) {
        this.config = config
      },
      submit() {
        this.$store.commit('')
      },
      typeChanged() {
        if(this.config.connectionType === 'mysql') {
          this.config.port = 3306
          this.config.host = this.config.host || 'localhost'
        } else if(this.config.connectionType === 'psql') {
          this.config.port = 5432
          this.config.host = this.config.host || 'localhost'
        }

      },
      checkConfig() {

      },
      testConnection(){

      },
      clearForm(){

      },
      save() {
        this.checkConfig(this.config)
        this.testConnection(this.config)
        if(!this.errors) {
          this.saveConnectionConfig(this.config)
        }
      }
    },
    computed: {
      connectionTypes() {
        return config.defaults.connectionTypes
      }
    },
    mounted() {
      window.config = this.config
    },

  }
</script>
<template>
  <section>

    <h3 class="text-center">Add/Remove Permissions</h3>

    <div>
      <input type="text" class="form-control input-lg" @input="getAvailablePermissions()" placeholder="Search for permissions to add" v-model="permissionSearchText">
    </div>

    <div v-if="loading" class="content content-centered">
      <pulse-loader></pulse-loader>
    </div>

    <div v-show="!loading">
      <div class="row content-centered" style="margin-top: 15px;">

        <div class="col-sm-5">
          <div class="box box-info box-solid">
            <div class="box-header">
              <h3 class="box-title" style="text-align: center;">Available Permissions</h3>
            </div>
            <div class="box-body">
              <select multiple ref="availablelist" size="10" style="width: 100%;"
                      v-model="selectedAvailablePermissions" @change="selectedRolePermissions = []">
                <option v-for="obj in availablePermissions" v-bind:value="obj">
                  {{ (obj.permission || {}).name }}
                </option>
              </select>
            </div>
          </div>
        </div>

        <div class="col-sm-2 content-centered">
          <div>
            <a @click="addPermissions()">
              <i class="fa fa-arrow-circle-right fa-3x icon-btn icon-btn-primary"></i>
            </a>
            <br/>
            <br/>
            <a @click="removePermissions()">
              <i class="fa fa-arrow-circle-left fa-3x icon-btn icon-btn-info"></i>
            </a>
          </div>
        </div>

        <div class="col-sm-5">
          <div class="box box-primary box-solid">
            <div class="box-header">
              <h3 class="box-title">Current Permissions</h3>
            </div>
            <div class="box-body">
              <select multiple ref="rolelist" size="10" style="width: 100%;"
                      v-model="selectedRolePermissions" @change="selectedAvailablePermissions = []">
                <option v-for="obj in newRole.permissions" v-bind:value="obj">
                  {{ obj.permission.name }}
                </option>
              </select>
            </div>
          </div>
        </div>
      </div>

      <div>
        <input class="form-control" style="margin-top: 15px" :disabled="true" :placeholder="permissionDescription"/>
      </div>


      <div class="row content-centered" style="margin-top: 15px">

        <div class="col-sm-4">

          <div class="box box-success box-solid">
            <div class="box-header">
              <h3 class="box-title">Included</h3>
            </div>
            <div class="box-body">
              <select multiple ref="includedlist" size="10" style="width: 100%;"
                      v-model="selectedIncludedPermissions">
                <option v-for="obj in includedPermissions" v-bind:value="obj">
                  {{ (obj.permission || {}).name }}
                </option>
              </select>
            </div>
          </div>
        </div>

        <div class="col-sm-4 content-centered">

          <div class="box box-warning box-solid">
            <div class="box-header">
              <h3 class="box-title">Excluded</h3>
            </div>
            <div class="box-body">
              <select multiple ref="excludedlist" size="10" style="width: 100%;"
                      v-model="selectedExcludedPermissions">
                <option v-for="obj in excludedPermissions" v-bind:value="obj">
                  {{ obj.permission.name }}
                </option>
              </select>
            </div>
          </div>
        </div>

        <div class="col-sm-4">
          <div class="box box-danger box-solid">
            <div class="box-header">
              <h3 class="box-title">Forbidden</h3>
            </div>
            <div class="box-body">
              <select multiple ref="forbiddenlist" size="10" style="width: 100%;"
                      v-model="selectedForbiddenPermissions">
                <option v-for="obj in forbiddenPermissions" v-bind:value="obj">
                  {{ obj.permission.name }}
                </option>
              </select>
            </div>
          </div>
        </div>

      </div>

      <div class="row content-centered">

        <div class="col-sm-4" style="margin-top: 15px">
          <div class="row content-centered">
            <div class="col-sm-6 text-right pull-right">
              <button class="btn btn-primary" @click="applyStateToIncluded"
                      :disabled="lodash.isEmpty(selectedIncludedPermissions)">Apply</button>
            </div>
            <div class="col-sm-6">
              <div class="radio radio-success">
                <input type="radio" id="includedToState_includedPermission" :value="PERMISSION_STATES.INCLUDED" v-model="includedToState">
                <label for="includedToState_includedPermission">{{ PERMISSION_STATES.INCLUDED }}</label>
              </div>
              <div class="radio radio-warning">
                <input type="radio" id="includedToState_excludedPermission" :value="PERMISSION_STATES.EXCLUDED" v-model="includedToState">
                <label for="includedToState_excludedPermission">{{ PERMISSION_STATES.EXCLUDED }}</label>
              </div>
              <div class="radio radio-danger">
                <input type="radio" id="includedToState_forbiddenPermission" :value="PERMISSION_STATES.FORBIDDEN" v-model="includedToState">
                <label for="includedToState_forbiddenPermission">{{ PERMISSION_STATES.FORBIDDEN }}</label>
              </div>
            </div>
          </div>
        </div>

        <div class="col-sm-4" style="margin-top: 15px">
          <div class="row content-centered">
            <div class="col-sm-6 text-right pull-right">
              <button class="btn btn-primary" @click="applyStateToExcluded"
                      :disabled="lodash.isEmpty(selectedExcludedPermissions)">Apply</button>
            </div>
            <div class="col-sm-6">
              <div class="radio radio-success">
                <input type="radio" id="excludedToState_includedPermission" :value="PERMISSION_STATES.INCLUDED" v-model="excludedToState">
                <label for="excludedToState_includedPermission">{{ PERMISSION_STATES.INCLUDED }}</label>
              </div>
              <div class="radio radio-warning">
                <input type="radio" id="excludedToState_excludedPermission" :value="PERMISSION_STATES.EXCLUDED" v-model="excludedToState">
                <label for="excludedToState_excludedPermission">{{ PERMISSION_STATES.EXCLUDED }}</label>
              </div>
              <div class="radio radio-danger">
                <input type="radio" id="excludedToState_forbiddenPermission" :value="PERMISSION_STATES.FORBIDDEN" v-model="excludedToState">
                <label for="excludedToState_forbiddenPermission">{{ PERMISSION_STATES.FORBIDDEN }}</label>
              </div>
            </div>
          </div>
        </div>

        <div class="col-sm-4" style="margin-top: 15px">
          <div class="row content-centered">
            <div class="col-sm-6 text-right pull-right">
              <button class="btn btn-primary" @click="applyStateToForbidden"
                      :disabled="lodash.isEmpty(selectedForbiddenPermissions)">Apply</button>
            </div>
            <div class="col-sm-6">
              <div class="radio radio-success">
                <input type="radio" id="forbiddenToState_includedPermission" :value="PERMISSION_STATES.INCLUDED" v-model="forbiddenToState">
                <label for="forbiddenToState_includedPermission">{{ PERMISSION_STATES.INCLUDED }}</label>
              </div>
              <div class="radio radio-warning">
                <input type="radio" id="forbiddenToState_excludedPermission" :value="PERMISSION_STATES.EXCLUDED" v-model="forbiddenToState">
                <label for="forbiddenToState_excludedPermission">{{ PERMISSION_STATES.EXCLUDED }}</label>
              </div>
              <div class="radio radio-danger">
                <input type="radio" id="forbiddenToState_forbiddenPermission" :value="PERMISSION_STATES.FORBIDDEN" v-model="forbiddenToState">
                <label for="forbiddenToState_forbiddenPermission">{{ PERMISSION_STATES.FORBIDDEN }}</label>
              </div>
            </div>
          </div>
        </div>

      </div>

      <div v-show="newRole._id" class="py-2 text-center row" style="margin-top: 10px">
        <button class="btn btn-primary" type="submit" @click="updateRolePermissions" :disabled="!dirty"
                v-permission.enable="['role', 'addRolePermissions', 'removeRolePermissions',
                'addRoleAssociations', 'removeRoleAssociations']">Update Role Permissions</button>
      </div>
    </div>

  </section>
</template>

<script>
  import _ from 'lodash'
  import { eventBus, roleService, userService } from '../../../services'
  import { PERMISSION_STATES, EVENTS } from '../../../config'

  export default {
    name: 'RolePermissions',
    props: ['role'],
    data () {
      return {
        lodash: _,
        loading: null,
        dirty: false,
        newRole: _.cloneDeep(this.role),
        oldRole: null,
        PERMISSION_STATES: PERMISSION_STATES,
        permissionSearchText: null,
        availablePermissions: [],
        selectedAvailablePermissions: [],
        selectedRolePermissions: [],
        selectedIncludedPermissions: [],
        selectedExcludedPermissions: [],
        selectedForbiddenPermissions: [],
        includedToState: PERMISSION_STATES.INCLUDED,
        excludedToState: PERMISSION_STATES.EXCLUDED,
        forbiddenToState: PERMISSION_STATES.FORBIDDEN
      }
    },
    watch: {
      selectedIncludedPermissions (val) {
        this.selectedRolePermissions = [].concat(val, this.selectedExcludedPermissions, this.selectedForbiddenPermissions)
      },
      selectedExcludedPermissions (val) {
        this.selectedRolePermissions = [].concat(val, this.selectedIncludedPermissions, this.selectedForbiddenPermissions)
      },
      selectedForbiddenPermissions (val) {
        this.selectedRolePermissions = [].concat(val, this.selectedExcludedPermissions, this.selectedIncludedPermissions)
      },
      role (val) {
        this.newRole = _.cloneDeep(val)
        this.newRole.permissions = this.newRole.permissions || []
        this.getAvailablePermissions()
        this.dirty = false
      }
    },
    computed: {
      permissionDescription () {
        const permission = (this.selectedAvailablePermissions[0] || {}).permission || (this.selectedRolePermissions[0] || {}).permission
        if (permission) {
          return permission.name + ': ' + permission.description
        }

        return 'Select a permission to see its description.'
      },
      includedPermissions () {
        return this.newRole.permissions.filter((permission) => { return permission.state === PERMISSION_STATES.INCLUDED })
      },
      excludedPermissions () {
        return this.newRole.permissions.filter((permission) => { return permission.state === PERMISSION_STATES.EXCLUDED })
      },
      forbiddenPermissions () {
        return this.newRole.permissions.filter((permission) => { return permission.state === PERMISSION_STATES.FORBIDDEN })
      }
    },
    methods: {
      getAvailablePermissions () {
        this.loading = true
        const rolePermissionIds = this.newRole.permissions.map((object) => { return object.permission._id })
        const params = {}
        if (this.permissionSearchText) {
          params.$term = this.permissionSearchText
          params.$searchFields = ['name']
        }
        if (!_.isEmpty(rolePermissionIds)) {
          // EXPL: Exclude the current role permissions from the list of available permissions
          params.$exclude = rolePermissionIds
        }
        return userService.getAvailablePermissions(params)
          .then((response) => {
            this.loading = false
            this.availablePermissions = response.data.docs.map((permission) => {
              return { permission, state: this.PERMISSION_STATES.INCLUDED }
            })
            this.sortLists()
          })
          .catch((error) => {
            this.loading = false
            console.error('RolePermissions.getAvailablePermissions-error:\n', error)
            this.$snotify.error('There was an error loading the permissions', 'Error!')
          })
      },
      addPermissions () {
        this.dirty = true
        this.newRole.permissions = this.newRole.permissions.concat(this.selectedAvailablePermissions)

        this.availablePermissions = this.availablePermissions.filter((object) => {
          return !this.selectedAvailablePermissions.find((selectedObject) => {
            return selectedObject.permission._id === object.permission._id
          })
        })

        this.selectedAvailablePermissions = []
        this.sortLists()
        eventBus.$emit(EVENTS.USER_PERMISSIONS_UPDATED, this.newRole.permissions)
      },
      removePermissions () {
        this.dirty = true
        this.availablePermissions = this.availablePermissions.concat(this.selectedRolePermissions)

        this.newRole.permissions = this.newRole.permissions.filter((object) => {
          return !this.selectedRolePermissions.find((selectedObject) => {
            return selectedObject.permission._id === object.permission._id
          })
        })

        this.selectedRolePermissions = []
        this.sortLists()
        eventBus.$emit(EVENTS.USER_PERMISSIONS_UPDATED, this.newRole.permissions)
      },
      sortLists () {
        this.newRole.permissions.sort((a, b) => { return a.permission.name.localeCompare(b.permission.name) })

        this.availablePermissions.sort((a, b) => { return a.permission.name.localeCompare(b.permission.name) })
      },
      applyStateToIncluded () {
        this.dirty = true
        for (let permission of this.selectedIncludedPermissions) {
          permission.state = this.includedToState
        }
        this.selectedIncludedPermissions = []
        eventBus.$emit(EVENTS.USER_PERMISSIONS_UPDATED, this.newRole.permissions)
      },
      applyStateToExcluded () {
        this.dirty = true
        for (let permission of this.selectedExcludedPermissions) {
          permission.state = this.excludedToState
        }
        this.selectedExcludedPermissions = []
        eventBus.$emit(EVENTS.USER_PERMISSIONS_UPDATED, this.newRole.permissions)
      },
      applyStateToForbidden () {
        this.dirty = true
        for (let permission of this.selectedForbiddenPermissions) {
          permission.state = this.forbiddenToState
        }
        this.selectedForbiddenPermissions = []
        eventBus.$emit(EVENTS.USER_PERMISSIONS_UPDATED, this.newRole.permissions)
      },
      updateRolePermissions () {
        roleService.updateRolePermissions(this.newRole._id, this.newRole.permissions, this.oldRole.permissions)
          .then((response) => {
            this.loading = false
            this.dirty = false
            this.oldRole = _.cloneDeep(this.newRole)
            eventBus.$emit(EVENTS.USER_PERMISSIONS_SAVED)
            this.$snotify.success('Role permissions updated', 'Success!')
          })
          .catch((error) => {
            this.loading = false
            console.error('RolePermissions.updateRolePermissions-error:', error)
            this.$snotify.error('Update role permissions failed', 'Error!')
          })
      }
    },
    created () {
      this.newRole.permissions = this.newRole.permissions || []
      this.oldRole = _.cloneDeep(this.newRole)
      this.getAvailablePermissions()
    }
  }
</script>

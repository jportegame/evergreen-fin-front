<template>
  <div>
    <b-card>
      <b-row class="mb-1">
        <b-col
          cols="11"
        >
          <div class="d-flex flex-row">
            <b-button
              variant="primary"
              class="btn-icon mr-25"
              @click="viewData"
            >
              <feather-icon icon="RefreshCcwIcon" />
            </b-button>
            <b-button
              v-b-modal.modal-customer
              variant="primary"
              class="btn-icon"
              @click="showForm"
            >
              <feather-icon icon="PlusIcon" />
              Nuevo Cliente
            </b-button>
          </div>
        </b-col>
        <b-col cols="1">
          <div class="d-flex justify-content-end">
            <b-spinner
              v-if="loading"
              label="Loading..."
              class="loading"
              variant="primary"
            />
          </div>
        </b-col>
        <b-modal
          id="modal-customer"
          title="Formulario Cliente"
          ok-only
          size="lg"
          :ok-title="update?'Actualizar':'Guardar'"
          @ok="update?updateCustomer():addCustomer()"
        >
          <p>Completa los datos para registrar un nuevo cliente.</p>
          <b-row>
            <b-col>
              <b-form-group
                label-cols="4"
                label-cols-lg="2"
                label="Identificacion"
                label-for="input-identify"
              >
                <b-form-input
                  id="input-identify"
                  v-model="form.identify"
                  placeholder="Identificacion"
                />
              </b-form-group>
              <b-form-group
                label-cols="4"
                label-cols-lg="2"
                label="Nombre"
                label-for="input-name"
              >
                <b-form-input
                  id="input-name"
                  v-model="form.name"
                  placeholder="Nombre"
                />
              </b-form-group>
              <b-form-group
                label-cols="4"
                label-cols-lg="2"
                label="Email"
                label-for="input-email"
              >
                <b-form-input
                  id="input-email"
                  v-model="form.email"
                  type="email"
                  placeholder="Email"
                />
              </b-form-group>
              <b-form-group
                label-cols="4"
                label-cols-lg="2"
                label="Telefono"
                label-for="input-phone"
              >
                <b-form-input
                  id="input-phone"
                  v-model="form.phone"
                  placeholder="Telefono"
                />
              </b-form-group>
              <b-form-group
                label-cols="4"
                label-cols-lg="2"
                label="Direccion"
                label-for="input-address"
              >
                <b-form-input
                  id="input-address"
                  v-model="form.address"
                  placeholder="Direccion"
                />
              </b-form-group>
              <b-form-group
                label-cols="4"
                label-cols-lg="2"
                label="Ciudad"
                label-for="input-city"
              >
                <b-form-input
                  id="input-city"
                  v-model="form.city"
                  placeholder="Ciudad"
                />
              </b-form-group>
            </b-col>
          </b-row>

        </b-modal>
      </b-row>
      <b-row>
        <b-col cols="12">
          <b-table
            :items="items"
            :fields="fields"
            bordered
            responsive
            class="mb-2 mw-100"
            show-empty
            empty-text="No records found"
          >
            <!-- Column: Actions -->
            <template #cell(actions)="data">
              <div class="d-flex flex-row justify-content-center">
                <b-button
                  v-ripple.400="'rgba(113, 102, 240, 0.15)'"
                  v-b-modal.modal-customer
                  variant="outline-primary"
                  size="sm"
                  class="mr-25"
                  @click="showUpdate(data.item)"
                >
                  <feather-icon icon="Edit2Icon" />
                </b-button>
                <b-button
                  v-b-modal.modal-2
                  v-ripple.400="'rgba(113, 102, 240, 0.15)'"
                  variant="outline-danger"
                  size="sm"
                  @click="deleteCustomer(data.item)"
                >
                  <feather-icon icon="TrashIcon" />

                </b-button>

              </div>
            </template>

            <!-- Optional default data cell scoped slot -->
            <template #cell()="data">
              <div>
                <p>{{ data.value }}</p>
              </div>
            </template>
          </b-table>
        </b-col>
      </b-row>
    </b-card>
  </div>
</template>

<script>
import {
  BCard, BTable, BRow, BCol, BButton, BModal, VBModal, VBToggle, BSpinner, BFormGroup, BFormInput,
} from 'bootstrap-vue'
import ToastificationContent from '@core/components/toastification/ToastificationContent.vue'
import Ripple from 'vue-ripple-directive'

// axios
import axios from '@axios'

export default {
  components: {
    BCard,
    BTable,
    BRow,
    BCol,
    BButton,
    BModal,
    BSpinner,
    BFormGroup,
    BFormInput,
  },
  directives: {
    'b-modal': VBModal,
    'b-toggle': VBToggle,
    Ripple,
  },
  data() {
    return {
      fields: [
        {
          key: 'id', label: 'Id',
        },
        {
          key: 'identify', label: 'Identificacion',
        },
        {
          key: 'name', label: 'Nombre',
        },
        {
          key: 'email', label: 'Email',
        },
        {
          key: 'phone', label: 'Telefono',
        },
        {
          key: 'address', label: 'Direccion',
        },
        {
          key: 'city', label: 'Ciudad',
        },
        {
          key: 'actions', label: '',
        },
      ],
      items: [],
      form: {
        identify: '',
        name: '',
        email: '',
        phone: '',
        address: '',
        city: '',
      },
      update: false,
      loading: false,
    }
  },
  async mounted() {
    await this.viewData()
  },
  methods: {
    async viewData() {
      this.loading = true
      await axios.get('https://gorest.co.in/public/v2/users').then(res => {
        if (res.status === 200) {
          this.items = res.data
        }
        this.loading = false
      }).catch(error => {
        this.$toast({
          component: ToastificationContent,
          props: {
            title: 'Error',
            icon: 'BellIcon',
            text: error,
            variant: 'danger',
          },
        })
        this.loading = false
      })
    },
    showUpdate(item) {
      this.update = true
      this.form.identify = item.identify
      this.form.name = item.name
      this.form.email = item.email
      this.form.phone = item.phone
      this.form.address = item.address
      this.form.city = item.city
    },
    showForm() {
      this.update = false
      this.form.identify = ''
      this.form.name = ''
      this.form.email = ''
      this.form.phone = ''
      this.form.address = ''
      this.form.city = ''
    },
    async addCustomer() {
      this.loading = true
      await axios.post('https://gorest.co.in/public/v2/users', this.form).then(res => {
        if (res.status === 200) {
          this.viewData()
        }
        this.loading = false
      }).catch(error => {
        this.$toast({
          component: ToastificationContent,
          props: {
            title: 'Error',
            icon: 'BellIcon',
            text: error,
            variant: 'danger',
          },
        })
        this.loading = false
      })
    },
    async updateCustomer() {
      this.loading = true
      await axios.put('https://gorest.co.in/public/v2/users', this.form).then(res => {
        if (res.status === 200) {
          this.viewData()
        }
        this.loading = false
      }).catch(error => {
        this.$toast({
          component: ToastificationContent,
          props: {
            title: 'Error',
            icon: 'BellIcon',
            text: error,
            variant: 'danger',
          },
        })
        this.loading = false
      })
    },
    async deleteCustomer(item) {
      this.loading = true
      await axios.delete(`https://gorest.co.in/public/v2/users/${item.id}`, this.form).then(res => {
        if (res.status === 200) {
          this.viewData()
        }
        this.loading = false
      }).catch(error => {
        this.$toast({
          component: ToastificationContent,
          props: {
            title: 'Error',
            icon: 'BellIcon',
            text: error,
            variant: 'danger',
          },
        })
        this.loading = false
      })
    },
  },
}
</script>

<style>

</style>

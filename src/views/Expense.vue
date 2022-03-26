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
              v-b-modal.modal-expense
              variant="primary"
              class="btn-icon"
              @click="showForm"
            >
              <feather-icon icon="PlusIcon" />
              Nuevo Gasto
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
          id="modal-expense"
          title="Formulario Gasto"
          ok-only
          size="lg"
          :ok-title="update?'Actualizar':'Guardar'"
          @ok="update?updateExpense():addExpense()"
        >
          <p>Completa los datos para registrar un nuevo gasto.</p>
          <b-row>
            <b-col>
              <b-form-group
                label-cols="4"
                label-cols-lg="2"
                label="Cliente"
                label-for="input-customer"
              >
                <b-form-select
                  v-model="form.client_id"
                  value-field="id"
                  text-field="name"
                  :options="customers"
                />
              </b-form-group>
              <b-form-group
                label-cols="4"
                label-cols-lg="2"
                label="Valor"
                label-for="input-value"
              >
                <b-form-input
                  id="input-value"
                  v-model="form.value"
                  placeholder="Valor"
                />
              </b-form-group>
              <b-form-group
                label-cols="4"
                label-cols-lg="2"
                label="Concepto"
                label-for="input-concept"
              >
                <b-form-input
                  id="input-concept"
                  v-model="form.concept"
                  type="email"
                  placeholder="Concepto"
                />
              </b-form-group>
              <b-form-group
                label-cols="4"
                label-cols-lg="2"
                label="Fecha"
                label-for="input-date"
              >
                <b-form-input
                  id="input-date"
                  v-model="form.date"
                  type="date"
                  placeholder="Fecha"
                />
              </b-form-group>
              <b-form-group
                label-cols="4"
                label-cols-lg="2"
                label="Fecha vencimiento"
                label-for="input-due_date"
              >
                <b-form-input
                  id="input-due_date"
                  v-model="form.due_date"
                  type="date"
                  placeholder="Fecha vencimiento"
                />
              </b-form-group>
              <b-form-group
                label-cols="4"
                label-cols-lg="2"
                label="Observaciones"
                label-for="input-observations"
              >
                <b-form-input
                  id="input-observations"
                  v-model="form.observations"
                  placeholder="Observaciones"
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
                  v-b-modal.modal-expense
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
                  @click="deleteExpense(data.item)"
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
  BCard, BTable, BRow, BCol, BButton, BModal, VBModal, VBToggle, BSpinner, BFormGroup, BFormInput, BFormSelect,
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
    BFormSelect,
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
          key: 'client_id', label: 'Id Cliente',
        },
        {
          key: 'value', label: 'Valor',
        },
        {
          key: 'concept', label: 'Concepto',
        },
        {
          key: 'date', label: 'Fecha',
        },
        {
          key: 'due_date', label: 'Fecha Vencimiento',
        },
        {
          key: 'observations', label: 'Observaciones',
        },
        {
          key: 'actions', label: '',
        },
      ],
      items: [],
      customers: [],
      form: {
        id: '',
        client_id: '',
        value: '',
        concept: '',
        date: '',
        due_date: '',
        observations: '',
      },
      update: false,
      loading: false,
    }
  },
  async mounted() {
    await this.getCustomers()
    await this.viewData()
  },
  methods: {
    async getCustomers() {
      this.loading = true
      await axios.get('https://evergreen-fin-back.herokuapp.com/client/').then(res => {
        if (res.status === 200) {
          this.customers = res.data
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
    async viewData() {
      this.loading = true
      await axios.get('https://evergreen-fin-back.herokuapp.com/expense/').then(res => {
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
      this.form.id = item.id
      this.form.client_id = item.client_id
      this.form.value = item.value
      this.form.concept = item.concept
      this.form.date = item.date
      this.form.due_date = item.due_date
      this.form.observations = item.observations
    },
    showForm() {
      this.update = false
      this.form.client_id = ''
      this.form.value = ''
      this.form.concept = ''
      this.form.date = ''
      this.form.due_date = ''
      this.form.observations = ''
    },
    async addExpense() {
      this.loading = true
      await axios.post('https://evergreen-fin-back.herokuapp.com/expense/', this.form).then(res => {
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
    async updateExpense() {
      this.loading = true
      await axios.put(`https://evergreen-fin-back.herokuapp.com/expense/${this.form.id}`, this.form).then(res => {
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
    async deleteExpense(item) {
      this.loading = true
      await axios.delete(`https://evergreen-fin-back.herokuapp.com/expense/${item.id}`, this.form).then(res => {
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

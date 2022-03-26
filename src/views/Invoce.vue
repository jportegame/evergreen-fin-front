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
              v-b-modal.modal-invoce
              variant="primary"
              class="btn-icon"
              @click="showForm"
            >
              <feather-icon icon="PlusIcon" />
              Nueva Factura
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
          id="modal-invoce"
          title="Formulario Factura"
          ok-only
          size="lg"
          :ok-title="update?'Actualizar':'Guardar'"
          @ok="update?updateInvoce():addInvoce()"
        >
          <p>Completa los datos para registrar una nueva factura.</p>
          <b-row>
            <b-col>

              <h4 class="my-2">
                Informacion de plazos
              </h4>

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
                label-for="input-due-date"
              >
                <b-form-input
                  id="input-due-date"
                  v-model="form.dueDate"
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

              <h4 class="my-2">
                Informacion de pago
              </h4>

              <b-form-group
                label-cols="4"
                label-cols-lg="2"
                label="Metodo de pago"
                label-for="input-payment-method"
              >
                <b-form-select
                  id="input-payment-method"
                  v-model="form.paymentMethod"
                  placeholder="Metodo de pago"
                  :options="paymentMethodOptions"
                />

              </b-form-group>
              <b-form-group
                label-cols="4"
                label-cols-lg="2"
                label="Periodicidad"
                label-for="input-periodicity"
              >
                <b-form-select
                  id="input-periodicity"
                  v-model="form.periodicity"
                  placeholder="Periodicidad"
                  :options="periodicityOptions"
                />

              </b-form-group>
              <b-form-group
                label-cols="4"
                label-cols-lg="2"
                label="Cuotas"
                label-for="input-quotes"
              >
                <b-form-input
                  id="input-quotes"
                  v-model="form.numberQuotas"
                  type="number"
                  placeholder="Cuotas"
                />
              </b-form-group>
            </b-col>
          </b-row>
          <b-row>
            <b-col>
              <h4 class="my-2">
                Productos
              </h4>
              <b-table
                :items="form.items"
                :fields="fieldsItems"
                bordered
                responsive
                class="mb-2 mw-100"
                show-empty
                empty-text="No records found"
              >

                <!-- Optional default data cell scoped slot -->
                <template #cell()="data">
                  <div>
                    <p>{{ data.value }}</p>
                  </div>
                </template>
              </b-table>
            </b-col>
          </b-row>
          <b-row>
            <b-col>
              <h4 class="my-2">
                Vendedor
              </h4>
              <b-form-group
                label-cols="4"
                label-cols-lg="2"
                label="Nombre"
                label-for="input-seller-name"
              >
                <b-form-input
                  id="input-seller-name"
                  v-model="form.seller.name"
                  disabled
                  placeholder="Nombre"
                />
              </b-form-group>
              <b-form-group
                label-cols="4"
                label-cols-lg="2"
                label="Identificacion"
                label-for="input-seller-identification"
              >
                <b-form-input
                  id="input-seller-identification"
                  v-model="form.seller.identification"
                  disabled
                  placeholder="Identificacion"
                />
              </b-form-group>
              <b-form-group
                label-cols="4"
                label-cols-lg="2"
                label="Observaciones"
                label-for="input-seller-observations"
              >
                <b-form-input
                  id="input-seller-observations"
                  v-model="form.seller.observations"
                  disabled
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
                  v-b-modal.modal-invoce
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
                  @click="deleteInvoce(data.item)"
                >
                  <feather-icon icon="TrashIcon" />

                </b-button>

              </div>
            </template>

            <!-- Column: items -->
            <template #cell(items)="data">
              <div>
                <b-button
                  v-ripple.400="'rgba(113, 102, 240, 0.15)'"
                  variant="outline-primary"
                  size="sm"
                  class="mr-25"
                  @click="data.toggleDetails"
                >
                  Productos
                </b-button>
              </div>
            </template>

            <template #row-details="row">
              <b-table
                :fields="fieldsItems"
                :items="row.item.items"
                bordered
                class="mb-2 mw-100"
                show-empty
                empty-text="No hay productos"
              />
              <b-button
                size="sm"
                variant="outline-secondary"
                @click="row.toggleDetails"
              >
                Ocultar
              </b-button>
            </template>

            <!-- Column: seller -->
            <template #cell(seller)="data">
              <div>
                <ul>
                  <li>
                    {{ data.item.seller.name }}
                  </li>
                  <li>
                    {{ data.item.seller.identification }}
                  </li>
                  <li>
                    {{ data.item.seller.observations }}
                  </li>
                </ul>
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
          key: 'date', label: 'Fecha',
        },
        {
          key: 'dueDate', label: 'Fecha Vencimiento',
        },
        {
          key: 'observations', label: 'Observaciones',
        },
        {
          key: 'paymentMethod', label: 'Metodo de pago',
        },
        {
          key: 'periodicity', label: 'Periodicidad',
        },
        {
          key: 'numberQuotas', label: 'Cuotas',
        },
        {
          key: 'items', label: 'Items',
        },
        {
          key: 'seller', label: 'Vendedor',
        },
        {
          key: 'actions', label: '',
        },
      ],
      items: [
        {
          id: '0',
          date: '03-03-2022',
          dueDate: '03-03-2025',
          observations: 'ninguna',
          paymentMethod: 'Debito',
          periodicity: 'Mensual',
          numberQuotas: '20',
          items: [
            {
              id: 0,
              name: 'Producto 1',
              price: 0,
              quantity: 0,
            },
            {
              id: 0,
              name: 'Producto 2',
              price: 0,
              quantity: 0,
            },
          ],
          seller: {
            id: 0,
            name: 'Yamile Perdomo',
            identification: '4322342',
            observations: 'Ninguna',
          },
        },
      ],
      fieldsItems: [
        {
          key: 'id', label: 'Id',
        },
        {
          key: 'name', label: 'Nombre',
        },
        {
          key: 'price', label: 'Precio',
        },
        {
          key: 'quantity', label: 'Cantidad',
        },
      ],
      paymentMethodOptions: [
        { value: 'Credito', text: 'Credito' },
        { value: 'Debito', text: 'Debito' },
      ],
      periodicityOptions: [
        { value: 'Quincenal', text: 'Quincenal' },
        { value: 'Mensual', text: 'Mensual' },
        { value: 'Bimestral', text: 'Bimestral' },
        { value: 'Trimestral', text: 'Trimestral' },
        { value: 'Semestral', text: 'Semestral' },
        { value: 'Anual', text: 'Anual' },
      ],
      form: {
        date: '',
        dueDate: '',
        observations: '',
        paymentMethod: '',
        periodicity: '',
        numberQuotas: '',
        client: 0,
        items: [
          {
            id: 0,
            name: 'Producto 1',
            price: 0,
            quantity: 0,
          },
        ],
        seller: {
          id: 0,
          name: 'Yamile Perdomo',
          identification: '4322342',
          observations: 'Ninguna',
        },
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
          // this.items = res.data
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
      this.form.date = item.date
      this.form.dueDate = item.dueDate
      this.form.observations = item.observations
      this.form.paymentMethod = item.paymentMethod
      this.form.periodicity = item.periodicity
      this.form.numberQuotas = item.numberQuotas
      this.form.items = item.items
      this.form.seller = item.seller
    },
    showForm() {
      this.update = false
      this.form.date = ''
      this.form.dueDate = ''
      this.form.observations = ''
      this.form.paymentMethod = ''
      this.form.periodicity = ''
      this.form.numberQuotas = ''
      this.form.items = [
        {
          id: 0,
          name: 'Producto 1',
          price: 0,
          quantity: 0,
        },
      ]
      this.form.seller = {
        id: 0,
        name: 'Yamile Perdomo',
        identification: '4322342',
        observations: 'Ninguna',
      }
    },
    async addInvoce() {
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
    async updateInvoce() {
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
    async deleteInvoce(item) {
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

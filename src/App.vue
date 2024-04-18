<template>
  <div class="card p-fluid">
    <DataTable
      :value="customers"
      lazy
      paginator
      :first="first"
      :rows="10"
      v-model:filters="filters"
      ref="dt"
      dataKey="id"
      :totalRecords="totalRecords"
      :loading="loading"
      @page="onPage($event)"
      @sort="onSort($event)"
      @filter="onFilter($event)"
      filterDisplay="row"
      :globalFilterFields="[
        'name',
        'country.name',
        'company',
        'representative.name',
      ]"
      v-model:selection="selectedCustomers"
      :selectAll="selectAll"
      @select-all-change="onSelectAllChange"
      @row-select="onRowSelect"
      @row-unselect="onRowUnselect"
      tableStyle="min-width: 75rem"
    >
      <Column selectionMode="multiple" headerStyle="width: 3rem"></Column>
      <Column field="name" header="Name" filterMatchMode="startsWith" sortable>
        <template #filter="{ filterModel, filterCallback }">
          <InputText
            type="text"
            v-model="filterModel.value"
            @keydown.enter="filterCallback()"
            class="p-column-filter"
            placeholder="Search"
          />
        </template>
      </Column>
      <Column
        field="country.name"
        header="Country"
        filterField="country.name"
        filterMatchMode="contains"
        sortable
      >
        <template #body="{ data }">
          <div class="flex align-items-center gap-2">
            <img
              alt="flag"
              src="https://primefaces.org/cdn/primevue/images/flag/flag_placeholder.png"
              :class="`flag flag-${data.country.code}`"
              style="width: 24px"
            />
            <span>{{ data.country.name }}</span>
          </div>
        </template>
        <template #filter="{ filterModel, filterCallback }">
          <InputText
            type="text"
            v-model="filterModel.value"
            @keydown.enter="filterCallback()"
            class="p-column-filter"
            placeholder="Search"
          />
        </template>
      </Column>
      <Column
        field="company"
        header="Company"
        filterMatchMode="contains"
        sortable
      >
        <template #filter="{ filterModel, filterCallback }">
          <InputText
            type="text"
            v-model="filterModel.value"
            @keydown.enter="filterCallback()"
            class="p-column-filter"
            placeholder="Search"
          />
        </template>
      </Column>
      <Column
        field="representative.name"
        header="Representative"
        filterField="representative.name"
        sortable
      >
        <template #body="{ data }">
          <div class="flex align-items-center gap-2">
            <img
              :alt="data.representative.name"
              :src="`https://primefaces.org/cdn/primevue/images/avatar/${data.representative.image}`"
              style="width: 32px"
            />
            <span>{{ data.representative.name }}</span>
          </div>
        </template>
        <template #filter="{ filterModel, filterCallback }">
          <InputText
            type="text"
            v-model="filterModel.value"
            @keydown.enter="filterCallback()"
            class="p-column-filter"
            placeholder="Search"
          />
        </template>
      </Column>
    </DataTable>
  </div>
</template>

<script>
import { CustomerService } from '@/service/CustomerService';

export default {
  data() {
    return {
      loading: false,
      totalRecords: 0,
      customers: null,
      selectedCustomers: null,
      selectAll: false,
      first: 2,
      filters: {
        name: { value: '', matchMode: 'contains' },
        'country.name': { value: '', matchMode: 'contains' },
        company: { value: '', matchMode: 'contains' },
        'representative.name': { value: '', matchMode: 'contains' },
      },
      lazyParams: {},
      columns: [
        { field: 'name', header: 'Name' },
        { field: 'country.name', header: 'Country' },
        { field: 'company', header: 'Company' },
        { field: 'representative.name', header: 'Representative' },
      ],
    };
  },
  mounted() {
    this.loading = true;

    this.lazyParams = {
      first: 2,
      rows: 10,
      sortField: null,
      sortOrder: null,
      filters: this.filters,
    };

    this.loadLazyData();
  },
  methods: {
    loadLazyData() {
      this.loading = true;
      this.lazyParams = {
        ...this.lazyParams,
        first: event?.first || this.first,
      };

      setTimeout(() => {
        CustomerService.getCustomers({
          lazyEvent: JSON.stringify(this.lazyParams),
        }).then((data) => {
          this.customers = data.customers;
          this.totalRecords = data.totalRecords;
          this.loading = false;
        });
      }, Math.random() * 1000 + 250);
    },
    onPage(event) {
      this.lazyParams = event;
      this.loadLazyData(event);
    },
    onSort(event) {
      this.lazyParams = event;
      this.loadLazyData(event);
    },
    onFilter(event) {
      this.lazyParams.filters = this.filters;
      this.loadLazyData(event);
    },
    onSelectAllChange(event) {
      const selectAll = event.checked;

      if (selectAll) {
        CustomerService.getCustomers().then((data) => {
          this.selectAll = true;
          this.selectedCustomers = data.customers;
        });
      } else {
        this.selectAll = false;
        this.selectedCustomers = [];
      }
    },
    onRowSelect() {
      this.selectAll = this.selectedCustomers.length === this.totalRecords;
    },
    onRowUnselect() {
      this.selectAll = false;
    },
  },
};
</script>

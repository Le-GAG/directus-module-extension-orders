<template>
  <vue-good-table v-if="groupedOrders.length"
                  :columns="visibleColumns"
                  :rows="groupedOrders"
                  :fixed-header="true"
                  :line-numbers="true"
                  :groupOptions="{ enabled: true }"
                  :search-options="{
                        enabled: true,
                        trigger: 'enter',
                        placeholder: 'Filtrer ce tableau',
                      }"
  >
    <div slot="emptystate">Il n'y a actuellement pas de commandes pour cette vente.</div>

    <div slot="table-actions">
      <div class="v-table__group-by">
        Grouper par:
        <label
          class="v-table__group-by-label"
          for="groupBy"
          @click.prevent="groupByProducer = false"
        >Adhérent</label>
        <v-toggle id="groupBy" v-model="groupByProducer"/>
        <label
          class="v-table__group-by-label"
          for="groupBy"
          @click.prevent="groupByProducer = true"
        >Producteur</label>
      </div>
    </div>

    <template slot="table-header-row" slot-scope="props">
      <span class="v-table__table-header">
        <span v-html="props.row.label"></span>
        <span><button @click="exportXls(props)">Excel</button></span>
      </span>
    </template>

    <div slot="table-actions-bottom">
      This will show up on the bottom of the table.
    </div>
  </vue-good-table>
</template>


<script>
  import 'vue-good-table/dist/vue-good-table.css';
  import { VueGoodTable } from 'vue-good-table';
  import VToggle          from '../../app/src/components/form-inputs/toggle';
  import XLSX             from 'xlsx';

  export default {
    name: 'VTable',

    components: { VueGoodTable, VToggle },

    props: {
      rows: { required: true },
    },

    data () {
      return {
        groupByProducer: true,

        columns: [
          //@formatter:off
          { field: 'productName', type: 'text',   label: 'Nom du produit' },
          { field: 'packaging',   type: 'text',   label: 'Conditionnement' },
          { field: 'capacity',    type: 'number', label: 'Contenance' },
          { field: 'quantity',    type: 'number', label: 'Quantité',        globalSearchDisabled: true },
          { field: 'unitPrice',   type: 'number', label: 'Prix unitaire',   globalSearchDisabled: true },
          { field: 'totalPrice',  type: 'number', label: 'Prix total',      globalSearchDisabled: true },
          { field: 'user',        type: 'text',   label: 'Adhérent' },
          { field: 'producer',    type: 'text',   label: 'Producteur' },
          //@formatter:on
        ],
      };
    },

    computed: {
      visibleColumns () {
        return this.columns.filter(column => {
          return (this.groupByProducer && column.field !== 'producer')
                 || (!this.groupByProducer && column.field !== 'user');
        });
      },

      groupedOrders () {
        if (this.groupByProducer) {
          return this.formatOrders(this.rows, 'producer');
        }

        return this.formatOrders(this.rows, 'user');
      },
    },

    methods: {
      exportXls(props) {
        const data = props.row.children.map(item => {
          const row = [
            item.productName,
            item.packaging,
            item.capacity,
            item.quantity,
            item.unitPrice,
            item.totalPrice,
          ];
          row.push(this.groupByProducer ? item.producer : item.user);

          return row;
        });

        const headerRow = [
          'Produit',
          'Conditionnement',
          'Contenance',
          'Quantité',
          'Prix unitaire',
          'Prix total',
        ];
        headerRow.push(this.groupByProducer ? 'Producteur' : 'Adhérent');
        data.unshift(headerRow);


        const wb = XLSX.utils.book_new();
        const ws = XLSX.utils.aoa_to_sheet(data);

        /* add worksheet to workbook */
        XLSX.utils.book_append_sheet(wb, ws, "Commande du GAG");

        /* write workbook */
        XLSX.writeFile(wb, 'Commande.xlsx');
      },

      formatOrders (orders, property) {
        const groupedOrders = _.groupBy(orders, item => {
          if(this.groupByProducer) {
            return `<a href="mailto:${item.producerEmail}">${item[ property ]}</a>`;
          }

          return `<a href="mailto:${item.userEmail}">${item[ property ]}</a>`;
        });

        return Object.entries(groupedOrders).map(([ key, item ]) => {
          return {
            mode:     'span',
            label:    key,
            html:     true,
            children: item,
          };
        });
      },
    },
  };
</script>

<style lang="scss" scoped>
  .v-table {
    &__group-by {
      display: flex;
      align-items: center;
    }

    &__group-by-label {
      margin: 0 1em;
    }

    &__table-header {
      display: flex;
      justify-content: space-between;
    }
  }
</style>

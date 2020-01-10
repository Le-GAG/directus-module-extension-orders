<template>
  <div class="orders-page">
    <v-header title="Vue d'ensemble des commandes"
              icon="shopping_basket"
    />

    <div class="orders-page__layout" v-if="saleOptions">
      <label class="orders-page__label" for="selected-sale">Vente</label>
      <v-select class="orders-page__sale-dropdown"
                id="selected-sale"
                :options="saleOptions"
                placeholder="Choisir une vente"
                v-model="currentSaleIndex"
      />

      <v-table :rows="orders" />

      <p>Possibilité de saisir la quantité et le montant (certains produits seulement)</p>
      <p>Il faut donc une table de liaison commandes_producteurs_adherents</p>
    </div>
  </div>
</template>

<script>
  import VTooltip      from 'v-tooltip';
  import Vue           from 'vue';
  import VHeader       from '../../app/src/components/header/header';
  import VSelect       from '../../app/src/components/form-inputs/select';
  import VHeaderButton from '../../app/src/components/header/header-button';
  import VIcon         from '../../app/src/components/icon';
  import VSpinner      from '../../app/node_modules/vue-simple-spinner';
  import VTable        from './VTable';

  Vue.component('v-icon', VIcon);
  Vue.component('v-spinner', VSpinner);

  Vue.use(VTooltip, {
    defaultDelay: { show: 500 },
    defaultOffset: 2,
    autoHide: false,
  });

  export default {
    name: 'LeGagOrdersModule',

    components: { VHeader, VHeaderButton, VSelect, VTable },

    filters: {
      date(value) {
        if (!value) {
          return null;
        }

        if (!(value instanceof Date)) {
          value = new Date(value);
        }

        return new Intl.DateTimeFormat('fr-FR').format(value);
      },
    },

    data () {
      return {
        currentSaleIndex:  null,
        sales:             null,
        orders:            [],
        currencyFormatter: new Intl.NumberFormat('fr-FR', { style: 'currency', currency: 'EUR' }),
      };
    },

    computed: {
      currentSale () {
        if (!this.currentSaleIndex) {
          return null;
        }

        return this.sales.find(sale => { return sale.id === parseInt(this.currentSaleIndex, 10); });
      },

      saleOptions () {
        if (!this.sales) {
          return null;
        }

        const sales = {};

        this.sales.forEach(sale => {
          const openingDate = this.$options.filters.date(sale.date_ouverture);
          const closureDate = this.$options.filters.date(sale.date_cloture);
          sales[sale.id] =`${openingDate} - ${closureDate}`;
        });

        return sales;
      },
    },

    watch: {
      currentSaleIndex(saleId) {
        if (!saleId) {
          return;
        }

        this.fetchOrders();
      },
    },

    methods: {
      async fetchOrders() {
        if (!this.currentSaleIndex) {
          return [];
        }

        const response = await this.$api.getItems('commandes', {
          fields:  [
            'produits.prix',
            'produits.quantite',
            'produits.produits_variantes_id.conditionnement.nom',
            'produits.produits_variantes_id.contenance',
            'produits.produits_variantes_id.produit.*.*',
            'created_by.first_name',
            'created_by.last_name',
            'created_by.email',
          ],
          filters: {
            commande_id: this.currentSaleIndex,
          },
        });

        const formattedOrders = [];
        response.data.forEach(order => {
          order.produits_variantes.forEach(variant => {
            formattedOrders.push({
              productName:   variant.produits_variantes_id.produit.nom,
              packaging:     variant.produits_variantes_id.conditionnement.nom,
              capacity:      variant.produits_variantes_id.contenance,
              quantity:      variant.quantite,
              unitPrice:     this.currencyFormatter.format(parseFloat(variant.prix)),
              totalPrice:    this.currencyFormatter.format(parseInt(variant.quantite, 10) * parseFloat(variant.prix)),
              user:          `${order.created_by.first_name} ${order.created_by.last_name }`,
              userEmail:     order.created_by.email,
              producer:      variant.produits_variantes_id.produit.producteur.raison_sociale,
              producerEmail: variant.produits_variantes_id.produit.producteur.email,
            });
          });
        });

        this.orders = formattedOrders;
      },
    },

    async mounted () {
      const ventes = await this.$api.getItems('ventes');
      this.sales = ventes.data;
    },
  };
</script>

<style scoped lang="scss">
  .material-icons {
    font-family: Material Icons;
    font-weight: 400;
    font-style: normal;
    display: inline-block;
    line-height: 1;
    text-transform: none;
    letter-spacing: normal;
    word-wrap: normal;
    white-space: nowrap;
    -webkit-font-feature-settings: "liga";
    font-feature-settings: "liga";
    vertical-align: middle;
  }

  .orders-page {
    &__header {
      display: flex;
      align-items: center;

      position: fixed;
      top: 0;
      left: var(--nav-sidebar-width);
      right: 0;

      height: var(--header-height);
      padding: 0 var(--page-padding);

      background-color: var(--body-background);
    }

    &__header-icon {
      display: flex;
      justify-content: center;
      align-items: center;

      width: 40px;
      height: 40px;

      margin-right: 16px;

      border-radius: 100%;
      background-color: var(--lightest-gray);
      cursor: initial;

      >>> i {
        font-size: 24px;
        color: var(--gray);
      }
    }

    &__title {
      color: var(--darker-gray);
      font-size: 22px;
    }

    &__label {
      font-family: var(--family-sans-serif);
      font-size: var(--size-2);
      font-weight: var(--weight-normal);
      color: var(--darkest-gray);
      padding: 10px;
    }

    &__layout {
      padding: var(--page-padding);
    }

    &__sale-dropdown {
      margin-bottom: var(--page-padding);
    }
  }
</style>

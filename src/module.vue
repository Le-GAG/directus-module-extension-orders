<template>
  <div class="orders-page">
    <v-header title="Commandes"
              :breadcrumb="breadcrumb"
    ></v-header>

    <label class="type-label orders-page__label" for="selected-sale">Vente</label>

    <v-select class="orders-page__sale-dropdown"
              id="selected-sale"
              :options="saleOptions"
              placeholder="Choisir une vente"
              v-model="currentSaleIndex"
              v-if="saleOptions"
    ></v-select>

    <div class="orders-page__loading-indicator">
      <v-spinner v-if="isLoading"></v-spinner>
    </div>

    <orders-table :rows="orders"></orders-table>

    <br><br><br>
    <p>Possibilité de saisir la quantité et le montant (certains produits seulement)</p>
    <p>Il faut donc une table de liaison commandes_producteurs_adherents</p>
  </div>
</template>

<script>
  import OrdersTable from './OrdersTable';

  export default {
    name: 'LeGagOrdersModule',

    components: { OrdersTable },

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
        isLoading:         false,
        currentSaleIndex:  null,
        sales:             null,
        orders:            [],
        currencyFormatter: new Intl.NumberFormat('fr-FR', { style: 'currency', currency: 'EUR' }),
      };
    },

    computed: {
      breadcrumb () {
        return [
          {
            name: 'Modules',
            path: `/${this.$store.state.currentProjectKey}/ext/modules`
          }
        ];
      },
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

    async mounted () {
      const ventes = await this.$api.getItems('ventes');
      this.sales = ventes.data;
    },

    methods: {
      async fetchOrders() {
        if (!this.currentSaleIndex) {
          return [];
        }

        this.isLoading = true;

        const response = await this.$api.getItems('commandes', {
          fields:  [
            'produits_variantes.prix',
            'produits_variantes.quantite',
            'produits_variantes.produits_variantes_id.conditionnement.nom',
            'produits_variantes.produits_variantes_id.contenance',
            'produits_variantes.produits_variantes_id.produit.*.*',
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
        this.isLoading = false;
      },
    },
  };
</script>

<style scoped lang="scss">
  .orders-page {
    padding: var(--page-padding);

    &__label {
      font-family: var(--family-sans-serif);
      font-size: var(--type-label-size);
      font-weight: var(--weight-normal);
      color: var(--heading-text-color);

      margin-top: var(--form-vertical-gap);
      margin-bottom: var(--input-label-margin);
    }

    &__sale-dropdown {
    }

    &__loading-indicator {
      display: flex;
      justify-content: center;

      padding: var(--input-padding);
    }
  }
</style>

<template>
  <div class="uds-welcome">
    <div class="uds-welcome__content">
      <h1>Pizzaria UDS</h1>
      <p>Faça seu pedido online.</p>
    </div>

    <div class="uds-welcome__media">
      <img
        src="@/assets/home-page-illustration.svg"
        alt="Monte seu pedido de pizza online."
      />
    </div>

    <div v-if="error" class="uds-welcome__error">
      <h3 class="uds-welcome__error__error-title">Atenção</h3>
      <p>{{ error }}</p>
    </div>

    <div class="uds-welcome__actions full-width">
      <UDSButton @click="startOrder" :loading="loading">
        Montar minha pizza
      </UDSButton>
    </div>
  </div>
</template>

<script>
import UDSButton from "@/components/UDSButton";
import { mapActions, mapMutations } from "vuex";
import * as TYPES from "@/store/types";

export default {
  name: "UDSWelcome",
  components: { UDSButton },
  data: () => ({
    selected: [],
    loading: false,
    error: false,
    options: [
      {
        name: "1",
        label: "1 chbx",
        checked: false
      },
      {
        name: "2",
        label: "2 chbx",
        checked: false
      },
      {
        name: "3",
        label: "3 chbx",
        checked: false
      }
    ]
  }),
  computed: {
    shouldRequest() {
      const { sizes, flavors, addOns } = this.$store.state;

      return {
        sizes: sizes.length === 0,
        flavors: flavors.length === 0,
        addOns: addOns.length === 0
      };
    }
  },
  methods: {
    async startOrder() {
      const { shouldRequest } = this;

      this.loading = true;

      if (this.error) {
        this.error = false;
      }

      if (
        shouldRequest.sizes &&
        shouldRequest.flavors &&
        shouldRequest.addOns
      ) {
        try {
          const makeRequest = request => request();

          const requests = [
            this.GET_SIZES,
            this.GET_ADD_ONS,
            this.GET_FLAVORS
          ].map(makeRequest);

          const [sizes, addOns, flavors] = await Promise.all(requests);

          this.SET_SIZES(sizes);
          this.SET_FLAVORS(flavors);
          this.SET_ADD_ONS(addOns);

          this.loading = false;

          this.$router.push("/order/size");
        } catch (error) {
          this.error = "Não foi possível iniciar seu pedido no momento.";
        } finally {
          this.loading = false;
        }
      } else {
        this.$router.push("/order/size");
      }
    },

    ...mapActions([
      TYPES.ACTIONS.GET_SIZES,
      TYPES.ACTIONS.GET_ADD_ONS,
      TYPES.ACTIONS.GET_FLAVORS
    ]),

    ...mapMutations([
      TYPES.MUTATIONS.SET_SIZES,
      TYPES.MUTATIONS.SET_ADD_ONS,
      TYPES.MUTATIONS.SET_FLAVORS
    ])
  }
};
</script>

<style>
.uds-welcome {
  padding-top: 24px;
  text-align: center;
}

.uds-welcome__content {
  text-align: center;
}

.uds-welcome__media {
  margin: 12px 0;
  text-align: center;
}

.uds-welcome__media img {
  width: 100%;
  max-width: 260px;
  margin-left: -24px;
}

.uds-welcome__error {
  width: 100%;
  max-width: 300px;
  margin: 16px 0;
}

.uds-welcome__error-title {
  font-size: 20px;
  line-height: 30px;
  font-weight: 700;
}

.uds-welcome__actions {
  margin-bottom: 24px;
  text-align: center;
}

.uds-welcome__actions button {
  margin: 0;
}
</style>

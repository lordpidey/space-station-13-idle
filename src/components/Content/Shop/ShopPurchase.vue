<template>
  <div
    class="content-block p-1 my-1 d-flex flex-column flex-md-row align-items-center"
    :class="[canPurchase ? 'clickable' :'locked']"
    @click="buy"
  >
    <img :src="icon" class="mr-2 purchase-icon" alt />
    <div class="d-flex flex-column">
      <span class="name">{{name}}</span>
      <span class="description">{{purchase.description}}</span>
      <div>
        <button v-if="canOpen" class="my-1 btn btn-primary btn-sm" @click.stop="viewOdds">View Odds</button>
      </div>
      <div class="requires d-flex flex-row align-items-center flex-wrap">
        <span class="requires mr-1">Requires:</span>
        <div
          v-for="(pair, index) in Object.entries(requiredLevels)"
          :key="'level'+index"
          class="d-flex flex-row align-items-center"
        >
          {{allJobs[pair[0]]}}
          <img :src="allJobs.find(job => job.id == pair[0]).icon" />
          <span>lvl{{pair[1]}}</span>
        </div>
        <div
          v-for="(pair, index) in Object.entries(requiredItems)"
          :key="'item'+index"
          class="d-flex flex-row align-items-center"
        >
          <img :id="id+index" :src="items[pair[0]].icon" />
          <item-popover :target="id+index" :itemId="pair[0]" />
          <span>x{{pair[1] | cleanNum}}</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import ITEMS from "@/data/items";
import { PURCHASES } from "@/data/shop";
import ItemPopover from "@/components/ItemPopover";
import ModalItemChance from "@/components/Modals/ModalItemChance";
import { ALL_JOBS } from "@/data/jobs";
export default {
  components: { ItemPopover },
  props: ["purchaseId"],
  computed: {
    id() {
      return this._uid.toString();
    },
    item() {
      return ITEMS[this.purchase.item];
    },
    items() {
      return ITEMS;
    },
    allJobs() {
      return ALL_JOBS;
    },
    purchase() {
      return PURCHASES[this.purchaseId];
    },
    icon() {
      if (this.purchase.icon) return this.purchase.icon;
      if (this.item) return this.item.icon;
      return null;
    },
    name() {
      if (this.purchase.name) return this.purchase.name;
      if (this.item) return this.item.name;
      return "Missing name";
    },
    canPurchase() {
      return this.$store.getters["inventory/canPurchase"](this.purchase);
    },
    requiredItems() {
      if (!this.purchase.requiredItems) return {};
      return this.purchase.requiredItems;
    },
    requiredLevels() {
      if (!this.purchase.requiredLevels) return {};
      return this.purchase.requiredLevels;
    },
    canOpen() {
      if (!this.item) return false;
      if (!this.item.itemTable && !this.itemTables) return false;
      return true;
    }
  },
  methods: {
    buy() {
      if (!this.canPurchase) return;
      this.$store.dispatch("inventory/purchase", this.purchase);
    },
    viewOdds() {
      this.$modal.show(
        ModalItemChance,
        { itemId: this.purchase.item },
        { height: "auto", width: "420px" }
      );
    }
  }
};
</script>

<style>
.purchase-icon {
  width: 64px;
  min-width: 64px;
}
.name {
  font-size: 18px;
}
.description {
  color: rgb(144, 144, 144);
  font-size: 14px;
}
.locked .description {
  color: rgb(214, 214, 214);
}
.requires {
  color: rgb(97, 97, 97);
  font-size: 15px;
}
.locked .requires {
  color: rgb(241, 241, 241);
}
.requires img {
  width: 32px;
}
</style>
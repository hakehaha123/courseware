<template>
  <div class="sc__container">
    <input
      type="search"
      v-model="searchInput"
      @blur="blur"
      @input="inputChanged"
      @focus="focus"
      @keyup.esc="escape"
      @keyup.enter="enter"
      @keydown.tab="enter"
      @keydown.up="up"
      @keydown.down="down"
    />
    <div class="sc__filtered-items" v-if="canShowFilteredItems">
      <div
        class="sc__filtered-item"
        :class="{ 'sc__filtered-item__hovered': index === cursor }"
        v-for="(item, index) in filteredItems"
        :key="index"
        @click="selectItem(item)"
        @mouseover="cursor = index"
      >
        <div>{{ item }}</div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from "vue";
export default Vue.extend({
  name: "auto-complete",
  data() {
    return {
      searchInput: "",
      showItems: false,
      filteredItems: new Array<string>(),
      cursor: -1,
      tempObjectItems: new Array<unknown>()
    };
  },
  props: {
    items: {
      type: Array,
      required: true
    },
    objectMatchkey: {
      type: String,
      required: false
    },
    template: {
      type: Object,
      required: false
    }
  },
  mounted() {
    this.setFilteredItems();
  },
  computed: {
    canShowFilteredItems(): boolean {
      return this.showItems && !!this.filteredItems.length;
    }
  },
  watch: {
    searchInput(newInput) {
      this.setFilteredItems(newInput);
    }
  },
  methods: {
    blur() {
      setTimeout(() => (this.showItems = false), 200);
    },
    focus() {
      this.showItems = true;
    },
    inputChanged() {
      this.showItems = true;
      this.$emit("inputChanged", this.searchInput);
    },
    setFilteredItems(newInput = "") {
      const areItemsOfStringtype: boolean = this.items.every(
        item => item && typeof item === "string"
      );
      if (areItemsOfStringtype) {
        this.setFilteredItemsForStringType(newInput);
      } else if (this.items.every(item => item && typeof item === "object")) {
        this.setFilteredItemsForObjectType(newInput);
      }

      this.cursor = -1;
    },
    setFilteredItemsForStringType(newInput: string) {
      const matchedItems: unknown[] = this.items.filter(item => {
        const stringItem: string = item as string;
        return this.isMatchFoundInStringItem(stringItem, newInput);
      });
      this.filteredItems = matchedItems.map(item => String(item));
    },
    setFilteredItemsForObjectType(newInput: string) {
      const matchedItems: unknown[] = this.items.filter(item => {
        const objectItem: unknown = item as unknown;
        return this.isMatchFoundInObjectItem(objectItem, newInput);
      });
      this.tempObjectItems.splice(0, this.tempObjectItems.length);
      this.filteredItems =
        matchedItems.length > 0
          ? matchedItems.map(item =>
              this.constructFileteredItemFromObject(item)
            )
          : this.items.map(item => this.constructFileteredItemFromObject(item));
    },
    constructFileteredItemFromObject(item: unknown) {
      let stringItem = "";
      if (item && typeof item === "object") {
        if (this.template) {
          stringItem = this.getStringItemBasedOnTemplate(item);
        } else {
          stringItem = String(
            item[this.objectMatchkey as keyof typeof item] ?? ""
          );
        }
      }
      return stringItem;
    },
    getStringItemBasedOnTemplate(item: unknown) {
      if (item && typeof item === "object") {
        if (this.template && this.template.keys) {
          const templateKeys: string[] = this.template.keys as string[];
          const itemValues = templateKeys.map(
            key => item[key as keyof typeof item] as string
          );
          this.tempObjectItems.push(item);
          return itemValues.join(this.template.separator);
        }

        return String(item[this.objectMatchkey as keyof typeof item]);
      }
      
      return "";
    },
    isMatchFoundInStringItem(item: string, newInput: string) {
      return item.toLocaleLowerCase().includes(newInput.toLocaleLowerCase());
    },
    isMatchFoundInObjectItem(item: unknown, newInput: string) {
      if (item && typeof item === "object"  && newInput && this.objectMatchkey) {
        const itemValue: string =
          item[this.objectMatchkey as keyof typeof item];
        const isMatchFound: boolean = itemValue
          .toLocaleLowerCase()
          .includes(newInput.toLocaleLowerCase());
        return isMatchFound;
      }

      return false;
    },
    selectItem(item: string) {
      if (item) {
        this.searchInput = this.getSelectedItem(item);
        this.$emit("inputChanged", this.searchInput);
        this.showItems = false;
      }
    },
    getSelectedItem(item: string) {
      if (this.objectMatchkey && this.tempObjectItems) {
        const itemObject = this.tempObjectItems.find(tempItem => 
            tempItem && typeof tempItem === "object" && item.includes(tempItem[this.objectMatchkey as keyof typeof tempItem])
        );
        if (itemObject && typeof itemObject === "object") {
          return (
            itemObject[this.objectMatchkey as keyof typeof itemObject] ?? item
          );
        }
      }
      return item;
    },
    enter() {
      if (this.showItems && this.filteredItems[this.cursor]) {
        this.selectItem(this.filteredItems[this.cursor]);
        this.$emit("inputChanged", this.searchInput);
        this.showItems = false;
      }
    },
    up() {
      if (this.cursor > -1) {
        this.cursor--;
        this.$el.getElementsByClassName("sc__filtered-item")[this.cursor];
      }
    },
    down() {
      this.showItems = true;
      if (this.cursor < this.filteredItems.length) {
        this.cursor++;
        this.$el.getElementsByClassName("sc__filtered-item")[this.cursor];
      }
    },
    escape() {
      this.showItems = !this.showItems;
    }
  }
});
</script>

<style scoped>
.sc__container {
  position: relative;
  display: grid;
  grid-template-columns: auto;
  justify-content: start;
}

.sc__filtered-items {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  width: auto;
  grid-row: 2;
  padding: 2px;
  text-align: left;
  border: 2px solid #ececec;
  border-top: none;
  border-radius: 2px;
  max-height: 400px;
  overflow-y: auto;
  z-index: 9999;
}

.sc__container .sc__filtered-items .sc__filtered-item {
  cursor: pointer;
}

.sc__container .sc__filtered-items .sc__filtered-item:hover,
.sc__container .sc__filtered-items .sc__filtered-item__hovered {
  background-color: #eee;
  color: #101010;
}
</style>

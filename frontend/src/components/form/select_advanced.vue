<template>
  <div
    class="select-border d-flex align-items-center flex-row flex-row-reverse"
    style=""
  >
    <span
      class="material-symbols-outlined px-2 d-flex align-items-center"
      style="font-size: 20px; cursor: pointer"
      @click="activeSelect = !activeSelect"
    >
      {{ activeSelect == true ? "expand_less" : "expand_more" }}</span
    >
    <input
      type="search"
      class="px-2"
      v-model="searchText"
      style="font-size: 16px;"
      @input="
        (activeSelect = true),
          $emit('searchSelect', searchText.toLocaleLowerCase())
      "
      @focus="activeSelect = true"
    />
    <!-- options -->
    <div
      v-if="activeSelect && disabled == 'false'"
      class="select-content overflow-auto d-flex flex-column align-items-start justify-content-start"
    >
      <span
        v-for="(option, index) in options"
        :key="index"
        class="select-option py-1 px-2 d-flex align-items-center justify-content-between"
        :class="modelValue == option.name ? 'text-slate-300' : 'text-slate-600'"
      >
        <span
          @click="
            [
              $emit('chose', option._id, option),
              (activeSelect = false),
              (modelValue = option.name),
              (searchText = modelValue),
            ]
          "
          class="mr-2"
          style="cursor: pointer; font-size: 15px;"
          onmouseover="this.style.color='green';"
          onmouseout="this.style.color='';"
          >{{ option.name }}</span
        >
        <span
          v-if="option._id != 'other' && add== true"
          @click="[$emit('delete', option)]"
          class="material-symbols-outlined"
          style="font-size: 15px; cursor: pointer"
          onmouseover="this.style.color='red';"
          onmouseout="this.style.color='';"
        >
          close
        </span>
      </span>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    modelValue: "",
    options: {
      type: Array,
      default: [],
    },
    disabled: {
      type: String,
      default: `false`,
    },
    add: {
      type: Boolean,
      default: true,
    },
  },
  watch: {
    modelValue() {
      this.searchText = this.modelValue;
    },
  },
  data() {
    return {
      activeSelect: false,
      searchText: this.modelValue,
    };
  },
  mounted() {
    // this.searchText = this.modelValue;
  },
};
</script>

<style scoped>
.select-border {
  position: relative;
  width: 100%;
  height: 100%;
  border: 1px solid var(--gray);
  border-radius: 5px;
}

input {
  background-color: inherit;
  width: 100%;
}

.select-content {
  position: absolute;
  top: 0;
  width: 100%;
  border-radius: 5px;
  border: 1px solid var(--gray);
  border-radius: 5px;
  z-index: 10;
  margin-top: 50px;
  background-color: white;
}

.select-option {
  cursor: pointer;
  width: 100%;
}

</style>

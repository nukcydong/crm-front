<template>
  <div class="searchPane">
    <div class="menuItem"
         v-for="menu in menuList"
         :key="menu.id"
         @click="selectItemFn(menu)"
    >
      <ta-icon :type="menu.icon || 'setting'"></ta-icon>
      {{menu.name}}
    </div>

  </div>
</template>

<script>
import {mapGetters} from 'vuex'

export default {
  name: 'search-pane',
  computed: {
    ...mapGetters({
      menuList: 'getSearchMenuList'
    })
  },
  methods: {
    selectItemFn (item) {
      this.$store.dispatch('addTabMenuList', {value: item})
      this.$store.dispatch('setActiveMenu', {level: 'two', value: item.id})
    }
  }

}
</script>

<style scoped type="text/scss" lang="scss">
  .searchPane {
    background-color: #001529;
    height: 100%;
    width: 100%;
    overflow: auto;
    position: absolute;
    top: 0px;
    left: 0px;
    @include beautifyScrollbar();

    > .menuItem {
      color: rgba(255, 255, 255, 0.71);

      height: 40px;
      line-height: 40px;
      cursor: pointer;
      @include text-overflow();
      box-sizing: border-box;
      padding-right: 30px;
      font-size: 14px;
      &:hover {
        background-color: $main-color;
        color: rgba(255, 255, 255, 0.71);
        > i {
          opacity: 1;
        }
      }

      > i {
        margin: 0px 14px 0px 12px;
        opacity: 0.6;
      }
    }
  }
</style>

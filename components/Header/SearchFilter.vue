<template>
  <div class="filter">
    <div class="filter--selector" @click="selectorClicked">
      <span class="name"> {{selectedCategory}} </span>
      <span ref="icon" class="icon "> <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="24" height="24"><path fill="none" d="M0 0h24v24H0z"/><path d="M12 11.828l-2.828 2.829-1.415-1.414L12 9l4.243 4.243-1.415 1.414L12 11.828z"/></svg> </span>
    </div>
    <transition name="slide-down-fade">
     <div class="filter--list-to-select" v-if="listActive">
        <ul>
          <li @click="selectCategory(i)" v-for="(f,i) in selectionList" :key="f"> {{f}} </li>
        </ul>
      </div>
    </transition>
  </div>
</template>

<script>
import hasClass from '~/utils/hasClass'
import {icons} from "~/services/icons.json"

//TODO write test for component
//TODO real source of data

export default {
  name: "SearchFilter",
  data() {
    return {
      listActive:false,
      indexOfSelectedCategory:0,
      selectionList:['all'],
    }
  },

  computed: {
    selectedCategory(){
      return this.selectionList[this.indexOfSelectedCategory];
    }
  },
  methods:{
    //listen on click
    selectorClicked(){
      this.listActive = !this.listActive
      const el = this.$refs['icon']

      if(hasClass(el,'selecting'))
        el.classList.remove('selecting')
      else
        el.classList.add('selecting')
    },

    //listen on category click
    selectCategory(i) {
      this.listActive = false
      this.indexOfSelectedCategory = i
      this.$bus.$emit('filterSelected',this.selectedCategory)
    }
  },
  mounted(){
    icons.map(icon => this.selectionList.push(icon.categoryName))
  }
}
</script>

<style lang="scss" scoped>

@mixin filter-selector {
    @apply px-2 py-4 rounded-md;
    background-color: $bg-tertiary ;
    width:15rem;
    cursor: pointer;
}

  .filter{
    position: relative;
    &--selector{
      @include filter-selector;
      @apply flex justify-between;
      span{
        text-transform: capitalize;

        &.name{
          @apply px-3;
        }
        &.icon{
          transition: transform .4s ease-in;
          transform: rotate(180deg);

        }
        &.selecting{
          transform: rotate(0deg);
        }
      }
    }

    &--list-to-select{
      @include filter-selector;
      position: absolute;
      height: fit-content;
      @apply mt-2;
      z-index: 1;

      ul{
        max-height: 20rem;
        overflow-y: auto;

        @include vertical-scrollbar;

        li{
          @apply py-2 px-3 rounded-sm;
          text-transform: capitalize;

          &:hover{
            background-color:  $bg-tertiary;
            border-right:3px solid $bg-secondary;
          }
        }
      }
    }
  }


// animations
.slide-down-fade-enter-active {
  transition: all .3s ease;
}
.slide-down-fade-leave-active {
  transition: all .4s cubic-bezier(1.0, 0.5, 0.8, 1.0);
}
.slide-down-fade-enter, .slide-down-fade-leave-to{
  transform: translateY(10px);
  opacity: 0;
}

</style>

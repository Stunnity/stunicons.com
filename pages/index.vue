<template>
  <div class="page-container">
    <div class="page">

      <section class="welcome-part">
        <section class="welcome-part--wrapper maximum-width">
          <!-- navbar -->
          <Navbar />

          <!-- welcome world -->
          <WelcomingText />

          <!-- Header bar -->
          <article class="flex justify-center w-full">
            <SearchInput @input="search" v-model="searchKey" />
          </article>

          <!--      icon body selector-->

        </section>

      </section>

      <section class="header-filter flex justify-center ">
        <section class="header-filter--wrapper maximum-width">
          <HeaderFilter @openCollection="openCollection" />
        </section>
      </section>

      <!-- icon pack list-->
      <section class="icon-pack flex justify-center" v-for="iconGroup in icons" :key="iconGroup.categoryName">
        <section class="maximum-width flex flex-col">
          <div class="icon-pack--header">
            <icon-pack-header :heading="iconGroup.categoryName" :number="iconGroup.icons.length"/>
          </div>
          <div class="icon-pack--icons " >
            <icon
              v-for="icon in iconGroup.icons"
              :stored="inStoredIcons(icon.id)"
              :key="icon.id"
              @add="addToCollection(icon,iconGroup.categoryName)"
              @remove="removeFromCollection(icon,iconGroup.categoryName)"
              @click="clickIcon(icon,iconGroup.categoryName)">
              <template #svg >
                <i :class="`${icon.id}`"></i>
              </template>
              <template #name> {{icon.name}} </template>
            </icon>
          </div>

        </section>
      </section>
      <!--          no icons-->
      <section class="no-result grid place-items-center" v-if="icons.length < 1">
        <p class="py-4 pb-32 text-center px-10">No result found for `{{searchKey}}`</p>
      </section>


        <!-- footer -->
        <app-footer/>

    </div>

    <!-- icon editor -->
    <transition name="fade">
      <div class="icon-editor-holder" ref="icon-editor-holder" @click="iconEditorHolderClicked" v-if="editorVisible">
        <div class="icon-editor-holder--wrapper">
            <icon-editor :icon="icon"/>
        </div>
      </div>
    </transition>

<!--    icon collection-->
    <transition name="fade">
      <div class="icon-collection-holder" ref="icon-collection-holder" @click="iconCollectionHolderClicked"  v-if="collectionVisible">
        <div class="icon-collection-holder--wrapper">
        <icon-collection />
        </div>
      </div>
    </transition>

<!--    icon collection-->
    <transition name="fade-icon">
      <div class="copied" v-if="isCopyMessageVisible">
        <span class="copied--wrapper">Copied!</span>
      </div>
    </transition>


  </div>
</template>

<script>
import Navbar from '~/components/Navbar.vue'
import WelcomingText from "../components/WelcomingText";
import HeaderFilter from "../components/Header/Header";
import IconPackHeader from "../components/IconPack/IconPackHeader";
import Icon from "../components/IconPack/Icon";
import IconBodySelector from "../components/IconBodySelector";
import appFooter from "../components/Footer";
import {icons} from "~/services/icons.json"
import IconEditor from "../components/IconEditor/IconEditor";
import IconCollection from "../components/IconCollection/iconCollection";
import iconCollectionMixin from "../mixins/iconCollection";
import SearchInput from "../components/Header/SearchInput";

export default {
  name:"Home",
  mixins:[iconCollectionMixin],
  components:{
    SearchInput,
    IconCollection,
    IconEditor, appFooter, IconBodySelector, Icon, IconPackHeader, HeaderFilter, WelcomingText, Navbar},
  data(){
    return {
      isCopyMessageVisible: false,
      searchKey:"",
      icons: icons,
      editorVisible:false,
      collectionVisible:false,
      icon:{
        name:"",
        id:"",
        src:""
      },
      numberOfStoredIcons: 0
    }
  },
  methods:{
    clickIcon(icon, category){
      this.icon.src = `${category}/${icon.id}.svg`
      this.icon.name = icon.name;
      this.icon.id = icon.id;
      this.editorVisible = true
    },
    // method to check if icon is stored
    inStoredIcons(id){
      return this.svgIcons.filter(icon => icon.id === id).length > 0
    },
    search(value){
      const foundIcons = []
      const {icons} = require('~/services/icons.json')

      //  if there is no search keyword
      //  show all icons
      if(value.trim().length <=0){
        this.icons = icons;
        return ;
      }

      //search

      icons.map(icon => { // loop in all icons categories

        icon.icons.map(singleIcon => { //loop into icons into single icon category

          //find icons that may have tags which contains search key
          const possibleSearches = singleIcon.tags.filter(tag => tag.indexOf(value) !== -1)

          if(possibleSearches.length > 0){ // if there are some icons , it is time to add them to search results
            let iconGroupIndex;

            // know if the icon category exists in searched icon results
            const existenceOfCategoryGroup = foundIcons.filter((iconGroup,index) => {
              iconGroupIndex = index;
              return iconGroup.categoryName === icon.categoryName
            })

            if(existenceOfCategoryGroup.length > 0) // if icon group exists push new icons to the category
              foundIcons[iconGroupIndex].icons.push(singleIcon)
            else    //if category does not exist add it with new found icon
              foundIcons.push({categoryName:icon.categoryName,icons:[singleIcon]})
          }


        })
      })

      this.icons = foundIcons

      //analytics
      this.$gtag.event('search', {
        'event_category': 'interaction',
        'event_label': 'search',
        'value': value
      })

    },
    openCollection(){
      this.collectionVisible = true
    },
    iconEditorHolderClicked(e){
      const targetToClick = this.$refs['icon-editor-holder']

      if(e.target === targetToClick)
        this.editorVisible = false;
    },
    iconCollectionHolderClicked(e){
      const targetToClick = this.$refs['icon-collection-holder']

      if(e.target === targetToClick)
        this.collectionVisible = false;
    },
    addToCollection(icon,category){
      this.$store.dispatch('storeIcon',{icon,category})
    },
    track(){
      this.$gtag.pageview(this.$route)
    }
  },
  mounted(){
    let timeout;
    const self = this;
    this.numberOfStoredIcons = this.svgIcons.length
    this.$store.commit('readStoredIcons')

    this.$bus.$on('filterSelected',(category) => {
      if(category.toLowerCase() === 'all')
        this.icons = icons;
      else
        this.icons = icons.filter(iconGroup => iconGroup.categoryName === category)
    })

    this.$bus.$on('iconCopy',()=>{
      self.isCopyMessageVisible = true;
      timeout = setTimeout(()=>{
        self.isCopyMessageVisible = false;
      },2000)
    })

    this.track();
  }

}
</script>

<style lang="scss" scoped>
.page-container{
  position: relative;
  .icon-editor-holder, .icon-collection-holder{
    z-index: 1;
    top:0;
    left:0;
    width: 100vw;
    height: 100vh;
    background-color: transparentize(#000,.8);
    position: fixed;
    display: grid;
    place-items: center;
    overflow: auto;
    padding:1rem;

    &--wrapper{

    }

  }

  .page{
    .welcome-part{
      @apply w-full flex justify-center;
      transition: all .1s ease;
      background-image: linear-gradient(-45deg, $red , $clr-primary 95%);
    }

    .icon-pack{

      &--header{
        @apply mb-8 ;
      }

      &--icons{
        display: grid;
        grid-template-columns: repeat(auto-fill, 7rem);
        grid-gap: 1rem;
        justify-content: space-between;
      }
    }
  }

  .copied{
    @apply rounded;
    position: fixed;
    top:90vh;
    z-index: 100;
    left:47%;
    background-color: $clr-primary;


    span{
      @apply py-2 px-10 block;
      color:$bg;
      font-size: .9rem;

    }
  }
}
.fade-enter-active, .fade-leave-active {
  transition: opacity .5s;
  .icon-collection-holder--wrapper,
  .icon-editor-holder--wrapper{
    transition: .5s;
    transition-property: opacity,transform;
  }
}
.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
  opacity: 0;

  .icon-collection-holder--wrapper,
  .icon-editor-holder--wrapper{
    transform: translateY(70px);
  }
}

.fade-icon-enter-active, .fade-icon-leave-active {
  //transition: opacity .5s;
  transition: .5s;
  transition-property: opacity,transform;
}
.fade-icon-enter, .fade-icon-leave-to /* .fade-leave-active below version 2.1.8 */ {
  opacity: 0;
  transform: translateY(70px);

}

</style>

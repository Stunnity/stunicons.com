<template>
<div class="icon-editor">
  <div class="icon-editor--wrapper" v-if="formattedSvg">
    <div class="rows row--1">
      <div class="icon">
      <div class="icon--title">
        <h5>{{icon.name}}</h5>
      </div>
      <div class="icon--svg" v-if="formattedSvg">
        <div id='svg-container' class="svg" v-html="formattedSvg">

        </div>
      </div>
      <div class="icon--download">
        <div class="icon--download--svg" @click="saveSvg">
          <copy>
            <template #icon>
              <svg width="16" height="16" viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
                <path d="M7 1C7 0.447716 7.44772 0 8 0C8.55228 0 9 0.447715 9 1V8.15777L12.2428 4.91501L13.657 6.32922L8.00012 11.9861L2.34326 6.32922L3.75748 4.91501L7 8.15753V1Z" fill="#FE4E00"/>
                <path d="M0 10H2V14H14V10H16V14C16 15.1046 15.1046 16 14 16H2C0.895431 16 0 15.1046 0 14V10Z" fill="#FE4E00"/>
              </svg>
            </template>
            <template #title>SVG</template>
          </copy>
        </div>
        <div class="icon--download--png" @click="savePng">
          <copy>
            <template #icon>
              <svg width="16" height="16" viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
                <path d="M7 1C7 0.447716 7.44772 0 8 0C8.55228 0 9 0.447715 9 1V8.15777L12.2428 4.91501L13.657 6.32922L8.00012 11.9861L2.34326 6.32922L3.75748 4.91501L7 8.15753V1Z" fill="#FE4E00"/>
                <path d="M0 10H2V14H14V10H16V14C16 15.1046 15.1046 16 14 16H2C0.895431 16 0 15.1046 0 14V10Z" fill="#FE4E00"/>
              </svg>
            </template>
            <template #title>PNG</template>
          </copy>
        </div>
      </div>
    </div>
    <div class="customize">
      <div class="customize--title">
        <h5>Customisation</h5>
      </div>
      <div class="customize--font">
        <font-size-adjuster v-model="fontSize" />
      </div>
      <div class="customize--color">
        <color-picker v-model="color"/>
      </div>
    </div>
    </div>

    <div class="rows row--2">
      <div class="icon-class">
        <div class="icon-class--title">
          <h5>Copy css</h5>
        </div>
        <div class="icon-class--copy">
          <icon-class-copy :icon-class="icon.id"/>
        </div>
      </div>
    </div>

    <div class="rows row--3">
      <div class="code">
        <div class="code--title">
          <h5>Copy svg</h5>
        </div>
        <div class="code--tab">
          <ul>
            <li :class="{active:activeTab ==='svg'}" @click="activeTab = 'svg'">SVG</li>
            <li :class="{active:activeTab ==='dataUrl'}" @click="activeTab = 'dataUrl'">Data URI</li>
          </ul>
        </div>
        <div class="code--codes" >
          <ul>
            <li v-if="activeTab ==='svg'">
              <code-highlight :code="codes.svg" lang="html" />
            </li>
            <li v-if="activeTab ==='dataUrl'">
              <code-highlight :code="codes.css" lang="css"/>
            </li>
          </ul>
        </div>
        <div class="code--copy">
          <div class="code--copy--wrapper" :data-clipboard-text="readyCode">
            <copy >
              <template #icon>
                <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="24" height="24"><path fill="none" d="M0 0h24v24H0z"/><path d="M7 6V3a1 1 0 0 1 1-1h12a1 1 0 0 1 1 1v14a1 1 0 0 1-1 1h-3v3c0 .552-.45 1-1.007 1H4.007A1.001 1.001 0 0 1 3 21l.003-14c0-.552.45-1 1.007-1H7zM5.003 8L5 20h10V8H5.003zM9 6h8v10h2V4H9v2z"/></svg>
              </template>
              <template #title>Copy</template>
            </copy>
          </div>
        </div>
      </div>
    </div>

  </div>
</div>
</template>

<script>
import Copy from "../Reusable/Copy";
import IconClassCopy from "./IconClassCopy";
import CodeHighlight from "./CodeHighlight";
import ColorPicker from "../SVG/reusable/ColorPicker";
import FontSizeAdjuster from "../SVG/reusable/FontSizeAdjuster";
import svgToEl from "../../mixins/svgToEl";
import ClipboardJS from "clipboard";
import invert from 'invert-color';

let svgIcon, clipboard;

export default {
  name: "IconEditor",
  mixins:[svgToEl],
  components: {FontSizeAdjuster, ColorPicker, CodeHighlight, IconClassCopy, Copy},
  props:{
    icon:{required:true,type:Object}
  },
  data(){
    return{
      baseSvg:'',
      fontSize:24,
      color:"",
      activeTab:'svg'
    }
  },
  watch:{
    color(val){
      this.invertIconBgColor()
      console.log(val)
    }
  },
  computed:{
    //return svg codes that are edited based on preference of the user
    formattedSvg(){
      let svg = this.svgToEl(this.baseSvg)

      // run bellow code when svg is loaded
      if(svg){

        let paths = svg.getElementsByTagName('path')
        let polygon = svg.getElementsByTagName('polygon')
        let rect = svg.getElementsByTagName('rect')

        svg.setAttribute('width',this.fontSize)
        svg.setAttribute('height',this.fontSize)
        svg.setAttribute("xmlns", "http://www.w3.org/2000/svg");
        svg.setAttribute("xmlns:xlink", "http://www.w3.org/1999/xlink");


        if(paths.length > 0 ){
          //transform all path els
          paths.forEach(pathEl => {
            pathEl.setAttribute('fill',this.color)
          })
        }

        if(polygon.length > 0){
          //transform all path els
          polygon.forEach(el => {
            el.setAttribute('fill',this.color)
          })
        }

        if(rect.length > 0){
          //transform all path els
          rect.forEach(el => {
            el.setAttribute('fill',this.color)
          })
        }


        svg = svg.outerHTML // to html element codes
        return svg.toString()
      }

    },

    //returns css and html svg codes based on formatted svg
    codes(){
      const svg = this.formattedSvg;
      const uri = encodeURIComponent(svg)

      return {svg:svg,css:`background-image: url("${uri}");`}
    },

    readyCode(){
      if(this.activeTab === 'svg')
        return this.codes.svg
      if(this.activeTab === 'dataUrl')
        return this.codes.css
    }

  },
  methods:{
    invertIconBgColor(){
      const el = document.getElementById('svg-container')
      el.style.backgroundColor = invert(this.color,{ black: '#001233', white: '#ffffff', threshold: 0.9 })
    },
    saveSvg(){
      try{
        const name = this.icon.id

        const svgData = this.formattedSvg;
        const preface = '<?xml version="1.0" standalone="no"?>\r\n';
        const svgBlob = new Blob([preface, svgData], {type:"image/svg+xml;charset=utf-8"});
        const svgUrl = URL.createObjectURL(svgBlob);

        //analytics
        this.$gtag.event('iconDownload', {
          'event_category': 'download',
          'event_label': 'svg',
          'value': this.icon.id
        })

        this.download(svgUrl, name);
      }catch(e){
        this.$sentry.captureException(e)
      }
    },

    savePng(){
      try{
        const name = this.icon.id + '.png'

        const image = new Image();
        image.src = 'data:image/svg+xml;base64,' + window.btoa(unescape(encodeURIComponent(this.formattedSvg)));

        image.onload = () => {
          const canvas = document.createElement('canvas');
          canvas.width = image.width;
          canvas.height = image.height;
          const context = canvas.getContext('2d');
          context.drawImage(image, 0, 0);

          //analytics
          this.$gtag.event('iconDownload', {
            'event_category': 'download',
            'event_label': 'png',
            'value': this.icon.id
          })

          this.download(canvas.toDataURL(),name)
        }
      }catch(e){
        this.$sentry.captureException(e)
      }
    },
    download(url,name){
      const downloadLink = document.createElement("a");
      downloadLink.href = url;
      downloadLink.download = name;
      document.body.appendChild(downloadLink);
      downloadLink.click();
      document.body.removeChild(downloadLink);

    },
  },
  async mounted(){
    svgIcon = await require(`stunicons/icons/${this.icon.src}?raw`)
    this.baseSvg = svgIcon

    clipboard = new ClipboardJS('.code--copy--wrapper')
    const self = this

    clipboard.on('success',function(e){
      //analytics
      self.$gtag.event('iconCopy', {
        'event_category': 'copy',
        'event_label': 'codes',
        'value': self.icon.id
      })
      self.$bus.$emit('iconCopy')
    })

    //analytics
    this.$gtag.event('iconOpened', {
      'event_category': 'interaction',
      'event_label': 'icon',
      'value': this.icon.id
    })
  }
}
</script>

<style lang="scss" >
.icon-editor{
  @include fit-content;

  &--wrapper{
    @apply rounded-lg;
    background-color: $bg;
    padding:3rem;
    width:fit-content;
    max-width: 50rem;
  }


  .row{

    &--1{
      @apply flex justify-between;

      .icon{
        &--title{
          h5{
            font-weight: 600;
            font-size: 1.2rem;
            text-transform: capitalize;
          }
        }

        &--svg{
          @apply my-3;
          .svg{
            @apply p-2;
            width:fit-content;
            border-radius:2px;
            svg{
              transition: 1s all ease;

              path{
                transition: 1s all ease;
              }
            }
          }
        }

        &--download{
          @apply flex;

          &--png{
            @apply ml-6;
          }

        }
      }

      .customize{
        &--title{
          font-weight: 600;
          font-size: 1.1rem;
        }

        &--font{
          @apply my-6;
        }
      }
    }

    &--2{
      .icon-class{
        @apply mt-6;

        &--title{
          @apply my-2;
          h5{
            font-size: 1.1rem;
            font-weight: 500;
          }
        }
        &--copy{}
      }
    }

    &--3{
      .code{
        &--title{
          @apply my-4;

          h5{
            font-weight: 500;
            font-size: 1.1rem;
          }
        }

        &--tab{
          ul{
            @apply flex;

            li{
              @apply mx-5 py-1;
              font-weight: 600;
              cursor: pointer;
            }

            .active{
              color:$clr-primary;
              border-bottom: 2px solid $clr-primary;
            }
          }
        }

        &--codes{
          @apply my-5;

          ul{

          }
        }

        &--copy{
          &--wrapper{
            @include fit-content;
          }
        }

      }
    }
  }
}
</style>

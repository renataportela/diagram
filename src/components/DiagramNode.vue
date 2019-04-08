<template>
   <VueDragResize 
      :w="node.width" 
      :h="node.height" 
      :x="node.left"
      :y="node.top"
      :minw="150"
      :minh="75"
      @resizestop="onResize" 
      @dragstop="onResize"   
      @resizing="onResizing"
      @dragging="onResizing"   
      @clicked="toggleActive" 
      @activated="onActivated"
      @deactivated="onDeactivated"
      @clicked.ctrl="onClickCreateLine"
      class="node" 
      :parentLimitation="true"
      :isActive="false"
      :isResizable="isResizable"
      :style="nodeStyle"
      :id="node.id" 
      :ref="'node_'+node.id" 
      :sticks="['tl', 'tr', 'br', 'bl']" 
   >            
      <div :id="'node_'+node.id" class="content-area">
         <div v-html="node.content" 
            :ref="'node_content_'+node.id" 
            tabindex="0" 
            @keyup.delete="onDeleteKeyUp">  
         </div>     
      </div>
   </VueDragResize>   
</template>

<script>
import VueDragResize from 'vue-drag-resize'

export default {
  name: "DiagramNode",
  props: ['node', 'active-node'],
  components: {
    VueDragResize
  },
  data(){
     return {
        isActive: false,
        isResizable: false
     }
  },

  computed: {
     nodeStyle(){
        return 'background-color: '+this.node.bgColor
     }
  },

  methods: {  
      onClickCreateLine(){
         if (this.activeNode != null){
            this.$emit('addLine', {
               start: this.activeNode,
               end: this.node.id
            })
            this.onDeactivated()
         }         
      },

      onDeactivated(){
         this.isResizable = false
         this.isActive = false
         this.$emit('onDeactivated')
      },

      onResize(newRect){
         newRect.nodeId = this.node.id
         this.$emit('onNodeResized', newRect)
      },

      onResizing(){
         this.$emit('onNodeResizing', this.node.id)
      },
      
      onActivated(){
         this.isActive = true
         this.$refs['node_content_'+this.node.id].focus()
         this.$emit('onActivated', this.node.id)
      },

      onDeleteKeyUp(){
         this.$emit('delete', this.node.id)
      },

      toggleActive(){
         this.isResizable = !this.isResizable
         this.isActive = !this.isActive

         if (this.isActive){
            this.$emit('onActivated', this.node.id)
         } 
         else{
            this.$emit('onDeactivated', this.node.id)
         }        
      }
  }
}
</script>

<style scoped>
.start-port{
   height: 20px;
   width: 20px;
   position: absolute;
   right: -20px;
   box-shadow: none;
   border:3px #2962ff solid;
}
.start-port.active{
   border-color:#AD0;
}
.content-area{
   display: flex;   
   flex-direction: column;
   height: 100%;
   align-content: space-between;  
   position: relative;
}
.buttons-area{
   position: absolute;
   left: 0;
   bottom: 0;
   width: 100%;
   text-align: center;
   margin-top: 5px;
}
.node{
   padding: 10px;
}
.vdr-stick{
   background-color: #fff;
   border-color:#FFF;
   border-radius: 4px;
}
.vdr-stick.start-line{
   background-color: red !important;
}
</style>


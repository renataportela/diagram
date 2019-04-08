<template>
   <div ref="diagram-area" id="diagram-area" class="diagram-area">
      <div style="text-align: right">
         <mu-button color="primary" @click="addNode" title="Add Node" style="margin-left: 10px">
            <i class="material-icons">
            web_asset
            </i> 
         </mu-button>
      </div>

      <diagram-node 
         v-for="node in nodes" 
         :key="node.id" 
         :node="node" 
         :active-node="activeNodeId"
         @onNodeResized="onNodeResized" 
         @onNodeResizing="onNodeResizing" 
         @onActivated="onNodeActivated" 
         @onDeactivated="onNodeDeactivated" 
         @addLine="onAddLine" 
         @delete="onDeleteNode">
      </diagram-node>
   </div>  
</template>

<script>
import DiagramNode from './DiagramNode.vue';

const whereId = searchId => {
   return item => item.id === searchId;
}
const NODES_STORAGE = 'nodesStorage12';

export default {
  name: "Diagram",
  
  components: {
     DiagramNode
  },

  data() {
    return {
      nodes: [],
      lines: [],
      linesByNode: {},
      isAddingLine: false,
      mousePosition: {},
      lineStarted: undefined,
      activeNodeId: null
    };
  },  

  mounted(){   
      const nodes = JSON.parse(localStorage.getItem(NODES_STORAGE));

      if (nodes == null) return;

      this.nodes = nodes;

      nodes.forEach(node => {
         this.linesByNode[node.id] = [];
      })

      this.$nextTick().then( () => {  
         const withLines = node => !!node.lines;

         nodes
            .filter(withLines)
            .forEach(node => {
               node.lines.forEach(line => {
                  this.createLine(node.id, line.nodeEndId);
               });          
         })
      })       
  },

  methods: { 
     onDeleteNode(nodeId) {
         const nodeIndex = this.nodes.findIndex(whereId(nodeId));
         const nodeLines = this.linesByNode[nodeId];

         nodeLines.forEach(line => {
            line.remove();
         })        

         delete this.linesByNode[nodeId];
         
         this.nodes.splice(nodeIndex, 1);

         if (this.nodes.length === 1){
            this.nodes[0].lines = [];
         }

         localStorage.setItem(NODES_STORAGE, JSON.stringify(this.nodes));
     },

     onAddLine(nodes) {
         this.createLine(nodes.start, nodes.end);
         const nodeIndex = this.nodes.findIndex(whereId(nodes.start));
         let storageNodes = this.nodes.slice(0);
         storageNodes[nodeIndex].lines.push({ nodeEndId: nodes.end });
         localStorage.setItem(NODES_STORAGE, JSON.stringify(storageNodes));
     },

     onNodeActivated(nodeId) {        
        this.activeNodeId = nodeId;         
     },

     onNodeDeactivated(){
        this.activeNodeId = null;
     },

     createLine(nodeStartId, nodeEndId){
         const nodeStart = document.getElementById('node_'+nodeStartId);
         const nodeEnd   = document.getElementById('node_'+nodeEndId);

         if (nodeStart && nodeEnd){
            let lineElement = new window.LeaderLine(
               nodeStart,
               nodeEnd, {
               startPlug: 'square',
               line: {
                  size: 3
               }
            });

            if (!this.linesByNode[nodeStartId]){
               this.linesByNode[nodeStartId] = [];
            }

            if (!this.linesByNode[nodeEndId]){
               this.linesByNode[nodeEndId] = [];
            }

            this.linesByNode[nodeStartId].push(lineElement);
            this.linesByNode[nodeEndId].push(lineElement);
         }         
     },

     onNodeResized(data){
         let storageNodes = this.nodes.slice(0);
         const nodeIndex = storageNodes.findIndex(whereId(data.nodeId));
         storageNodes[nodeIndex].top = data.top;
         storageNodes[nodeIndex].left = data.left;
         storageNodes[nodeIndex].width = data.width;
         storageNodes[nodeIndex].height = data.height;
         localStorage.setItem(NODES_STORAGE, JSON.stringify(storageNodes));
     },
     
     onNodeResizing(nodeId){      
         this.linesByNode[nodeId].forEach(line => line.position());
     },
     
     addNode(){
        const ids = Object.keys(this.nodes)
                        .reduce((ids, nodeIndex) => {
                           ids.push(this.nodes[nodeIndex].id);
                           return ids;
                        }, [])

        const nodeId = ids.length == 0 ? 1 : Math.max( ...ids)+1;

        this.nodes.push({
           id: nodeId,
           top: 20,
           left: 20,
           width: 100,
           height: 75,
           content: 'Node '+nodeId,
           bgColor: 'white',
           lines: []
        });
        
        this.linesByNode[nodeId] = [];

        localStorage.setItem(NODES_STORAGE, this.nodes);
     }
  }
}
</script>

<style scoped>
.diagram-area{
   padding: 20px;
   background-color:#aadd;
   min-height: 500px;
   position: relative;
}
.adding-line{
   box-shadow: 0 !important;
   background-color: '#82b1ff' !important;
}
</style>

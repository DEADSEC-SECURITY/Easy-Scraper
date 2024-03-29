<template>
  <div>
    <NodeSettings ref="node-settings" />
    <!-- Side drawer -->
    <v-navigation-drawer app permanent class="text-center">
      <h1 class="mt-3">Nodes</h1>
      <v-list class="nodes">
        <v-list-item>
          <DrawerNode name="Get Url" type="default" secondaryType="getUrl" />
          <DrawerNode name="Get Element" type="default" secondaryType="getElement" />
          <DrawerNode name="If" type="default" secondaryType="ifStatement" />
          <DrawerNode name="Save" type="default" secondaryType="saveDataa" />
          <DrawerNode name="End" type="output" secondaryType="end" />
        </v-list-item>
      </v-list>
    </v-navigation-drawer>
    <!-- Flow canvas -->
    <v-container class="pa-0">
      <div @drop="onDrop">
        <VueFlow
          v-model="elements"
          :default-zoom="1"
          :min-zoom="0.2"
          :max-zoom="4"
          @connect="onConnect"
          @dragover="onDragOver"
        >
          <Background variant="lines" />
          <Controls />
        </VueFlow>
      </div>
    </v-container>
  </div>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import type { Elements, FlowEvents, VueFlowStore } from "@braks/vue-flow";
import { Background, Controls, VueFlow, addEdge } from "@braks/vue-flow";

// Local imports
import DrawerNode from "../components/DrawerNode.vue";
import NodeSettings from "../components/NodeSettings.vue";

// CSS Styling
import "@braks/vue-flow/dist/style.css";
import "@braks/vue-flow/dist/theme-default.css";

export default defineComponent({
  name: "NewFlow",
  components: { VueFlow, Background, Controls, DrawerNode, NodeSettings },
  data() {
    return {
      instance: null as VueFlowStore | null,
      elements: [] as Elements,
      // Nodes loaded from DB have positive id but ones generated on client side have negative ids
      last_node_id: 0,
    };
  },
  mounted() {
    this.setupNewFlow();
  },
  methods: {
    // Add edges on VueFlow load
    onConnect(params: FlowEvents["connect"]) {
      addEdge(params, this.elements);
    },
    // Creates a start node centered on the canvas
    setupNewFlow() {
      // Get the drawer width
      const drawer = document.getElementsByClassName("v-navigation-drawer")[0];
      const drawerWidth = drawer.style.width.replace("px", "");
      // Get the window width
      const windowWidth = document.documentElement.clientWidth;
      // Calculate position
      const position = { x: (windowWidth - drawerWidth) / 2 - 75, y: 50 };
      // Add node
      const startNode = {
        id: "0",
        type: "input",
        label: "Start",
        position,
      };
      this.elements.push(startNode);
    },
    /* Returns the next id in line
     * Ids generated by this function will be negative meaning
     * they are new nodes and not ones loaded from the database
     */
    getID() {
      this.last_node_id -= 1;
      return JSON.stringify(this.last_node_id);
    },
    // Handle dragging from Node Menu to VueFlow
    onDrop(e: {
      dataTransfer: { getData: (path: string) => never };
      offsetX: number;
      offsetY: number;
    }) {
      // Get data from node selected
      const type = e.dataTransfer?.getData("application/flow/type");
      const secondaryType = e.dataTransfer?.getData(
        "application/flow/type/secondary"
      );
      const label = e.dataTransfer?.getData("application/flow/label");
      // Open and wait for settings popUp
      this.$refs["node-settings"].openModal(secondaryType);
      const data_ = Object();
      data_["secondaryType"] = secondaryType;
      // Create node
      const position = { x: e.offsetX - 75, y: e.offsetY - 20 };
      const newNode = {
        id: this.getID(),
        type,
        position,
        label,
        data_,
      };
      this.elements.push(newNode);
    },
    onDragOver(e: {
      preventDefault: () => void;
      dataTransfer: { dropEffect: string };
    }) {
      e.preventDefault();
      if (e.dataTransfer) {
        e.dataTransfer.dropEffect = "move";
      }
    },
  },
});
</script>

<style scoped>
.vue-flow {
  width: 100%;
  height: calc(100vh - 64px);
}
.node-actions {
  position: absolute;
  right: 10px;
  top: 10px;
  z-index: 4;
}
.nodes .node {
  position: relative;
  margin-bottom: 10px;
  cursor: grab;
  left: 50%;
  transform: translateX(-50%);
}
</style>

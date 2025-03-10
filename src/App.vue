<template>
  <div id="app">
    <div class="app">
      <h1>SimulationVuer</h1>
      <h3>Simulations run on o²S²PARC:</h3>
      <el-radio-group v-model="id" size="small">
        <el-radio-button :class="className(dataset.id)" v-for="dataset in sparcDatasets" v-bind:key="dataset.id" :label="dataset.label" :value="dataset.id" />
      </el-radio-group>
      <h3>Simulations run in the browser:</h3>
      <el-radio-group v-model="id" size="small">
        <el-radio-button :class="className(dataset.id)" v-for="dataset in pmrDatasets" v-bind:key="dataset.id" :label="dataset.label" :value="dataset.id" />
      </el-radio-group>
      <div ref="dropArea" class="drop-area" @dragenter.prevent="onDragEnter" @dragover.prevent @drop.prevent="onDrop" @dragleave.prevent="onDragLeave">
        You can drag and drop a COMBINE archive here.
      </div>
      <el-dialog v-model="dragAndDropWarningVisible" width="352">
        <span>Please only drag and drop one COMBINE archive.</span>
        <template #footer>
          <div class="dialog-footer">
            <el-button type="primary" @click="dragAndDropWarningVisible = false">
              OK
            </el-button>
          </div>
        </template>
      </el-dialog>
    </div>
    <hr />
    <div v-for="dataset in datasets()" v-bind:key="dataset.id">
      <div v-if="initialised(dataset.id)" v-show="dataset.id == id">
        <div v-if="typeof dataset.id === 'number'">
          <span>
            <strong>Dataset <a :href="datasetUrl(dataset.id)" target="_blank">{{ dataset.id }}</a>:</strong> {{ dataset.description }}
          </span>
          <hr />
        </div>
        <SimulationVuer :apiLocation="apiLocation" :id="dataset.id" style="height: 640px;" />
      </div>
    </div>
    <div v-if="id == -1">
      <div v-for="combineArchive in combineArchives" v-bind:key="combineArchive">
        <SimulationVuer :apiLocation="apiLocation" :combineArchive="combineArchive" style="height: 640px;" />
      </div>
    </div>
    <hr />
  </div>
</template>

<script>
import SimulationVuer from "./components/SimulationVuer.vue";
import { ElRadioButton, ElRadioGroup } from "element-plus";

export default {
  name: "App",
  components: {
    SimulationVuer,
    ElRadioButton,
    ElRadioGroup,
  },
  data: function () {
    return {
      apiLocation: import.meta.env.VITE_API_LOCATION,
      dropAreaCounter: 0,
      dragAndDropWarningVisible: false,
      combineArchives: [],
      sparcDatasets: [
        { id: 0, label: "0", description: "Non-simulation dataset" },
        { id: 135, label: "135", description: "Computational analysis of the human sinus node action potential - Model development and effects of mutations" },
        { id: 157, label: "157", description: "Fabbri-based composite SAN model" },
        { id: 308, label: "308", description: "Kember Cardiac Nerve Model" },
        { id: 318, label: "318", description: "Multi-scale rabbit cardiac electrophysiology models" },
        { id: 320, label: "320", description: "Multi-scale human cardiac electrophysiology models" },
      ],
      pmrDatasets: [
        { id: "workspace/b7c/rawfile/e0ae8d2d56aaaa091e23e1ee7e84cacbda1dfb6b/135.omex", label: "135", description: "COMBINE archive from PMR" },
        { id: "workspace/b7c/rawfile/e0ae8d2d56aaaa091e23e1ee7e84cacbda1dfb6b/157.omex", label: "157", description: "COMBINE archive from PMR" },
        { id: "workspace/b7c/rawfile/e0ae8d2d56aaaa091e23e1ee7e84cacbda1dfb6b/lorenz.omex", label: "Lorenz", description: "COMBINE archive from PMR" },
        { id: "workspace/b7c/rawfile/e0ae8d2d56aaaa091e23e1ee7e84cacbda1dfb6b/tt04.omex", label: "TT04", description: "COMBINE archive from PMR" },
        { id: "workspace/b7c/rawfile/e0ae8d2d56aaaa091e23e1ee7e84cacbda1dfb6b/Gee_whiz_Exported.omex", label: "GeeWhiz", description: "COMBINE archive from PMR" },
        { id: "workspace/b7c/rawfile/e0ae8d2d56aaaa091e23e1ee7e84cacbda1dfb6b/cvs.omex", label: "CVS", description: "COMBINE archive from PMR" },
      ],
      id: 0,
      ready: [],
    };
  },
  methods: {
    className(id) {
      return ((id == this.sparcDatasets[0].id) || (id == this.pmrDatasets[0].id)) ? "first-dataset" : "";
    },
    datasets() {
      return this.sparcDatasets.concat(this.pmrDatasets);
    },
    datasetUrl(id) {
      return `https://sparc.science/datasets/${id}?type=dataset`;
    },
    initialised(id) {
      if (this.ready.includes(id)) {
        return true;
      }

      if (this.id == id) {
        this.ready.push(id);
      }

      return this.ready.includes(id);
    },
    onDragEnter() {
      this.dropAreaCounter += 1;

      if (this.dropAreaCounter === 1) {
        this.$refs.dropArea.classList.add('drop-area-active');
      }
    },
    onDrop(event) {
      this.dropAreaCounter = 0;

      this.$refs.dropArea.classList.remove('drop-area-active');

      const files = event.dataTransfer.files;

      if (files.length !== 1) {
        this.dragAndDropWarningVisible = true;
      } else {
        files[0]
          .arrayBuffer()
          .then((arrayBuffer) => {
            this.id = -1;
            this.combineArchives = [new Uint8Array(arrayBuffer)];
          })
      }
    },
    onDragLeave() {
      this.dropAreaCounter -= 1;

      if (this.dropAreaCounter === 0) {
        this.$refs.dropArea.classList.remove('drop-area-active');
      }
    }
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}

.drop-area {
  position: absolute;
  top: 8px;
  right: 8px;
  width: 192px;
  height: 48px;
  border: 2px dashed #8300bf;
  background-color: #f9f2fc;
  color: #8300bf;
  border-radius: 20px;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 8px;
}

.drop-area-active {
  background-color: #8300bf;
  color: white;
}

.el-button,
.el-button:hover {
  border-color: #8300bf;
  background-color: #8300bf;
}

.el-radio-button__inner {
  background-color: #f9f2fc;
  border-color: #8300bf;
  box-shadow: -1px 0 0 0 #8300bf !important;
  color: #8300bf;
}

.first-dataset>.el-radio-button__inner {
  border-left: 1px solid #8300bf !important;
}

.el-radio-button__original-radio:checked+.el-radio-button__inner {
  background-color: #8300bf !important;
  border-color: #8300bf !important;
  color: white;
}

.el-radio-button__inner:hover {
  color: #8300bf;
}

a {
  color: #2c3e50;
}

div.app {
  text-align: center;
}

h3 {
  margin-bottom: 0;
}
</style>

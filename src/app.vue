<template>
  <div class="col">
    <div class="row">
      <div id="dropZone" v-on:drop.prevent="parseXLSX($event)" v-on:dragend="cleanup" ondragenter="event.preventDefault();" ondragover="event.preventDefault(); event.dataTransfer.dropEffect='copy'" class="col drop-box">
        <h2 class="text-center"> Drag your xlsx file here.</h2>
      </div>
    </div>
    <div class="row">
      <input type='file' id='inputFile' v-on:change="parseXLSX($event.target.files)">
      <div v-if="hasDownload">
        <a id="download"> Download Localalization JSON </a>
      </div>
    </div>   
    <div class="row">
      <div class="col json-box">
        <h2 class="text-center"> JSON Output</h2>
        <pre id="output"> </pre>
      </div>
    </div>
    <xlsx-footer></xlsx-footer>
  </div> 
</template>


<script>
import Footer from './components/footer.vue';
export default {
  data() {
    return {
      hasDownload: false,
    }
  },
  methods: {
    parseXLSX(event) {
      const XLSX = window.XLSX;
      let file = this.getFile(event);
      let workBook = null;
      let jsonData = null;
      
      if(file !== null) {
        const reader = new FileReader();
        const rABS = true;
        reader.onload = (event) => {
          const data = event.target.result; 
          if(rABS) {
            workBook = XLSX.read(data, {type: 'binary'});
            jsonData = workBook.SheetNames.reduce((initial, name) => {
              const sheet = workBook.Sheets[name];
              initial[name] = XLSX.utils.sheet_to_json(sheet);
              return initial;
            }, {});
            const dataString = JSON.stringify(jsonData, 2, 2);
            document.getElementById('output').innerHTML = dataString.slice(0, 300).concat("...");
            this.setDownload(dataString);
            
          }
        }
        if(rABS) reader.readAsBinaryString(file);
        else reader.readAsArrayBuffer(file);
      }
    },
    getFile(item) {
      if(item.dataTransfer !== undefined) {
        const dt = item.dataTransfer;
        if(dt.items) {
          if(dt.items[0].kind == 'file') {
           return dt.items[0].getAsFile();
          }
        }
      }
      else {
        return item[0];
      }
    },
    setDownload(json) {
      this.hasDownload = true;
      setTimeout(()=> {
        const el = document.getElementById("download");
        el.href = `data:text/json;charset=utf-8,${encodeURIComponent(json)}`;
        el.download = 'localization.json';
      }, 1000)
      
    },
    cleanup(event) {
      console.log("Cleaned up Event", event);
    }
  },
  components: {
    'xlsx-footer': Footer,
  }
}
</script>

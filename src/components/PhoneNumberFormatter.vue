<template>
  <!-- <div class="hello">
    <h1>{{ msg }}</h1>
  </div> -->
  <section class="container">
    <h1 class="mb-5 has-text-weight-bold d-poppins">Phone Number Formatter v1.1</h1>
    <b-field v-if="this.dropFiles.length === 0" class="m-4">
      <b-upload v-model="dropFiles" drag-drop type="is-primary">
        <section class="section">
          <div class="content has-text-centered">
            <p>
              <b-icon icon="upload" size="is-large"> </b-icon>
            </p>
            <p class="d-poppins">Drop your files here or click to upload</p>
          </div>
        </section>
      </b-upload>
    </b-field>

<b-field v-if="this.dropFiles.length !== 0" class="m-4"> 
  <div class="upload control">
    <div class="upload-draggable is-primary">
      <section class="section">
    <div class="tags" style="border-radius: 1rem; justify-content: center; margin-bottom: 0rem !important; min-width: 268px; height: 88px" v-if="this.dropFiles.length !== 0">
      <!-- <div class="d-poppins mr-3">File Uploaded</div> -->
      <span
        style="padding: 1rem; border-radius: 0.5rem; border: 1px solid #cccccc;"
        class="tag is-grey d-poppins"
      >
        {{ dropFiles.name }}
        <button
          class="delete is-small"
          type="button"
          @click="deleteDropFile()"
        ></button>
        
      </span>
      <span v-if="status == 'processed'">
        <b-tag type="is-primary is-light d-poppins" style="border-radius: 0.5rem; border: 1px solid #177d44; height: 32px;">Completed</b-tag>
      </span>
      
    </div>
    </section>
    </div>
    
    </div>
    </b-field>
    

    <b-button
      type="is-primary"
      class="d-poppins"
      @click="startProcessing"
      :disabled="this.dropFiles.length == 0 ? true : false"
      v-if="status != 'processed'"
    >
      Start Processing
    </b-button>

    <b-button
      type="is-primary"
      class="d-poppins"
      @click="downloadCsv"
      v-if="status == 'processed'"
    >
      Download
    </b-button>

    <b-button
      type="is-grey ml-3"
      class="d-poppins"
      @click="reset"
      v-if="this.dropFiles.length !== 0"
    >
      Reset
      </b-button>

      <div class="mt-5 mx-4" v-if="showProgress">
        <b-progress></b-progress>
      </div>

  </section>
</template>

<script>
export default {
  name: "PhoneNumberFormatter",
  props: {
    msg: String,
  },
  data() {
    return {
      dropFiles: [],
      jsonData: {},
      processedData: [],
      status: '',
      showProgress: false
    };
  },
  methods: {
    isNumber(param){
      return /^\d+$/.test(param);
    },
    deleteDropFile() {
      //this.dropFiles.splice(index, 1);
      this.dropFiles = [];
      this.status = 'initial'
    },
    toJson() {
      return new Promise((resolve, reject) => {
        try {

          //Get the uploaded file, azureUploader is the reference here, QUploader is the reference within our bit component.
          let file = this.dropFiles;

          //console.log("FILE", file);

          let fileReader = new FileReader();
          fileReader.onload = function (e) {
            let csv = e.target.result;

            //console.log(csv);

            var lines = csv.split("\r\n");
            //console.log("lines ", lines);

            var result = [];

            //var headers = lines[0].split(",");
            //console.log("headers ", headers);

            for (var i = 0; i < lines.length; i++) {
              var obj = {};
              var currentline = lines[i];
              //console.log("line ", i, " ", currentline);

              obj = currentline;

              if (obj !== null && obj !== "" && obj !== undefined) {
                result.push(obj);
              }
            }

            //console.log("result ", result);
            //console.log("JSON str", JSON.stringify(result));
            //console.log("Data ", this.data);

            resolve(JSON.stringify(result));
          };
          fileReader.readAsBinaryString(file);
        } catch (err) {
          reject(err);
        }
      });
    },
    async startProcessing() {

      this.showProgress = true;

      var strJson = await this.toJson();
      var Arr = JSON.parse(strJson);

      for (let i = 0; i < Arr.length; i++) {

        if (JSON.stringify(Arr[i]) == "{}"){
          Arr[i] = "null"
        }else{

          Arr[i] = Arr[i].replace("+", "").replace("-", "").split(' ').join('');
          console.log('test isNumber', this.isNumber(Arr[0]));

          if (Arr[i].charAt(0) === "1" && this.isNumber(Arr[i]) ) {
            let front1 = "60";
            Arr[i] = front1.concat(Arr[i]);

          } else if (Arr[i].charAt(0) === "0" && this.isNumber(Arr[i])) {
            let front2 = "6";
            Arr[i] = front2.concat(Arr[i]);

          } else if (Arr[i].charAt(0) === "6" && this.isNumber(Arr[i])) {
            // OK

          } else {
            let back = " [Error]"
            Arr[i] = Arr[i].concat(back);
            // Warning
            // how about landline number?
          }
        }
      }

      console.log("Uploaded Data", Arr);
      this.processedData = Arr;
      console.log("Processed Data", this.processedData);

      this.showProgress = false;
      this.status = 'processed';

    },
    downloadCsv(){
      //var jsonString = JSON.stringify(this.processedData);
      // var fields = Object.keys(json[0])
      // var replacer = function(key, value) { return value === null ? '' : value } 
      // var csv = json.map(function(row){
      //   return fields.map(function(fieldName){
      //     return JSON.stringify(row[fieldName], replacer)
      //   }).join(',')
      // })
      // csv.unshift(fields.join(',')) // add header column
      // csv = csv.join('\r\n');
      var content = this.processedData.join(', \r\n');
      //console.log( jsonString.replace('[','').replace(']','') );

      //var data = jsonString.replace('[','').replace(']','');

      let csvContent = "data:text/csv;charset=utf-8," + content;
      var encodedUri = encodeURI(csvContent);
      window.open(encodedUri);

    },
    reset(){
      this.dropFiles = [];
      this.status = 'initial'
    }

  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
.upload-draggable{
  border-radius: 1rem !important;
}
</style>

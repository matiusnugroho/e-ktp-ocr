<template>
<div class="container">
  <div class="card justify-content-center">
    <div class="card-body">
      <h5 class="card-title">KTP Reader</h5>
      <p class="card-text">Upload foto KTP yang bagus agar mudah dibaca</p>
      <div class="mb-3">
        <label for="formFile" class="form-label">Default file input example</label>
        <input class="form-control" type="file" @change="handleChange"/>
      </div>
      <img v-if="ktpImage" id="text-img" alt="Vue logo" :src="ktpImage" class="card-img">
      <button class="btn btn-primary" @click="cekNik">Proses KTP</button>
      <p class="card-text">{{statusReading}}</p>
      <div class="progress">
        <div class="progress-bar" role="progressbar" v-bind:style="progressWidth" aria-valuenow="25" aria-valuemin="0" aria-valuemax="100"></div>
      </div>
      <div v-show="parsedData">
        <p>Nama : {{parsedData.nama}}</p>
        <p>Agama : {{parsedData.agama}}</p>
        <p>Tempat/Tgi Lahir : {{parsedData["Tempat/Tgi Lahir"]}}</p>
      </div>
    </div>
  </div>
</div>
</template>
<script>
import {ref} from "vue"
import { createWorker, PSM, OEM, Tesseract} from 'tesseract.js'


  export default{
    name: "App",
    setup(){
      let ktpImage = ref('')
      let textResult = ref('')
      let progress = ref('0')
      let statusReading = ref('standby')
      let parsedData = ref({})
      let progressWidth = ref('width:0%')
      const worker = createWorker({
        logger: (m) => {
          console.log(m)
          let progressPercent = m.progress * 100
          progress.value = progressPercent
          progressWidth.value = `width:${progressPercent}%`
          statusReading .value= m.status
        },
      })

      const cekNik = async ()=>{
        textResult.value = "Sedang membaca..."
        const img = document.getElementById('text-img')
        await worker.load()
        await worker.loadLanguage('eng')
        await worker.initialize('eng', OEM.LSTM_ONLY)
        await worker.setParameters({
          tessedit_pageseg_mode: PSM.SINGLE_BLOCK,
        })
        const { data: { text } } = await worker.recognize(img)
        textResult.value = text
        parseKTP(text)
      }
      
      const handleChange = (event) => {
        ktpImage.value=URL.createObjectURL(event.target.files[0]);
      }
      const parseKTP = (text)=>{
        let data ={}
        let fields = ['nik','nama','Tempat/Tgi Lahir','agama'];
        const regex = new RegExp(fields.join('|'), 'i')
        let splittedText = text.split(regex)
        fields.forEach((key,index)=>data[key] = splittedText[index+1].match(/^([^\n]*)\n/,'')[1].replace(/[^0-9a-z ]-,/gi, "").replace(/\s\s+/g, ' ').replace(':','').trim())
        data['nik']=data['nik'].replace(/[^0-9]/gi, "")
        parsedData.value=data
        console.log(data,splittedText)
      }
      return {
        cekNik,
        ktpImage,
        handleChange,
        textResult,
        progress,
        progressWidth,
        statusReading,
        parsedData
      }
    }
    
  }
</script>
<style scoped>
  .container{
    width: 100%;
    margin: auto;
  }
  .card-img{
    width: 360px;
    height: 240px;
  }
</style>

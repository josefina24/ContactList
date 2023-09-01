<script setup>
import axios from "axios"
import Papa from "papaparse"
import { ref, watch, onMounted } from "vue"

let file = ""
const content = ref([])
const vista = ref([])
const size = ref("")
const loading = ref(false)
const uploadedContacts = ref([])
const loadCompleted = ref(false)

onMounted(() => {
  watch(content, () => {
      uploadContacts()
  })
})

const downloadSample = () => {
  fetch(`https://8j5baasof2.execute-api.us-west-2.amazonaws.com/production/tests/trucode/samples?size=${size.value}`)
    .then((response) => response.blob())
    .then((blob) => {
      var file = window.URL.createObjectURL(blob)
      var a = document.createElement("a")
      a.href = file
      a.download = "samples.csv"
      document.body.appendChild(a)
      a.click()
      a.remove()
    });
};

const handleFileUpload = (event) => {
  file = event.target.files[0]
  parseFile()
  loading.value = true
  loadCompleted.value = false
};

const parseFile = () => {
  Papa.parse(file, {
    header: false,
    skipEmptyLines: true,
    complete: function (results) {
      content.value = results.data
    }.bind(this),
  });
};

const addContact = async (data) => {
  vista.value.push(data)
  await axios.post('https://8j5baasof2.execute-api.us-west-2.amazonaws.com/production/tests/trucode/items',data)
}

const uploadContacts = async () => {
  for (const contact of content.value) {
    const contactFormatted = {
      "name": contact[0],
      "email": contact[2],
      "phone": formatNumber(contact[1].toString())
    };
    await addContact(contactFormatted)
  }
  if (content.length === vista.length){
        loading.value = false
        loadCompleted.value = true
      }
};

const formatNumber = (phone) => {
  let completedPhone = ''
  if (phone.length < 10){
    completedPhone = '0'.repeat(10 - phone.length) + phone
  } else {
    completedPhone = phone
  }
  let phoneFormatted = ''
  phoneFormatted += completedPhone.substr(0,3) + '-'
  phoneFormatted += completedPhone.substr(3,3) + '-'
  phoneFormatted += completedPhone.substr(6,4)
  return phoneFormatted
}

const getSize = (event) => {
  size.value = event.target.value
}

const getUploadedContacts = async () => {
  const response = await axios.get('https://8j5baasof2.execute-api.us-west-2.amazonaws.com/production/tests/trucode/items')
  uploadedContacts.value = response.data.items
  console.log(uploadedContacts)
}

</script>

<template>
  <div>
    <div>
      <h3>Aquí puedes descargar una lista de contactos de muestra</h3>
      <div>
        <label for="size">Ingrese un número de 1 a 100: </label>
        <input @input="getSize" v-model="size" id="size" placeholder="Ingresa el número aquí">
      </div>

      <br>
      <button @click="downloadSample">Descargar muestra</button>
    </div>

    <div>------------------------------------------------------------------</div>

    <div>
      <h3>Selecciona tu archivo CSV para subir </h3>
      <input type="file" accept=".csv" @change="handleFileUpload($event)" />
      <div v-if="loading">Cargando...</div>
      <div v-if="loadCompleted"> Todos los datos han sido cargados </div>
    </div>

    <div>------------------------------------------------------------------</div>

    <div v-if="loadCompleted">
      <h3>Presiona aqui si quieres ver la lista de contactos subidos</h3>
      <button @click="getUploadedContacts">Obtener datos subidos</button>
      <div v-for="contact in uploadedContacts">
        <div>Nombre: {{ contact.name }} - Telefono: {{ contact.phone }} - Correo: {{ contact.email }}</div>
      </div>
    </div>
  </div>
</template>

<style scoped></style>

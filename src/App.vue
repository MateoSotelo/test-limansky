<template>
  <head>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-QWTKZyjpPEjISv5WaRU9OFeRpok6YctnYmDr5pNlyT2bRjXh0JMhjY6hW+ALEwIH" crossorigin="anonymous">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons/font/bootstrap-icons.css">
  </head>
  <div>
    <nav class="navbar navbar-expand-lg navbar-dark bg-primary">
      <div class="container">
        <a class="navbar-brand">Sistema API</a>
        <form class="d-flex ms-auto me-4">
          <input class="form-control me-2" type="search" placeholder="Buscar por nombre" v-model="filtroNombre">
        </form>
      </div>
    </nav>
    <div class="container mt-4">
      <div class="row">
        <div class="col-lg-8 offset-lg-2">
          <div class="table-responsive">
            <table class="table table-bordered table-hover table-primary">
              <thead>
                <tr>
                  <th>NOMBRE</th>
                  <th class="id" @click="ordenarPor('idModule')">
                    ID
                    <i :class="['ms-2', ordenAscendente ? 'bi bi-sort-up' : 'bi bi-sort-down']"></i>
                  </th>
                  <th>DESCRIPCION</th>
                  <th class="fecha-entrada" @click="ordenarPor('entryDate')">
                    FECHA DE ENTRADA
                    <i :class="['ms-2', ordenAscendente ? 'bi bi-sort-up' : 'bi bi-sort-down']"></i>
                  </th>
                </tr>
              </thead>
              <tbody class="table-group-divider">
                <tr v-for="(mdls, i) in modulosFiltrados" :key="mdls.idModule">
                  <td>{{ mdls.name }}</td>
                  <td>{{ mdls.idModule }}</td>
                  <td>{{ mdls.description }}</td>
                  <td>{{ formatoFecha(mdls.entryDate) }}</td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>


<script setup>
import { ref, onMounted, computed } from 'vue'

//Estado
const modulos = ref([])
const filtroNombre = ref('')
const ordenAscendente = ref(true)
const columnaOrdenada = ref(null)

//Peticion GET a la API
const fetchModulos = async () => {
  try {
    const response = await fetch('https://api-evaluaciones-test.limansky.com/modules/modules')
    const data = await response.json()
    modulos.value = data
  } catch (error) {
    console.error('Error:', error)
  }
}

//Funcion que llama a fetchModulos cuando se carga el componente en el DOM
onMounted(fetchModulos)

//Cambio el formato de la fecha a dd-mm-yyyy
const formatoFecha = (dateString) => {
  const fecha = new Date(dateString)
  const dia = fecha.getDate().toString().padStart(2, '0')
  const mes = (fecha.getMonth() + 1).toString().padStart(2, '0') // Javascript indexa los meses del 0 al 11, por eso sumo +1
  const anio = fecha.getFullYear()
  return `${dia}-${mes}-${anio}`
}

//Funcion de filtros
const modulosFiltrados = computed(() => {
  return modulos.value.filter(mdls => {
    return mdls.name.toLowerCase().includes(filtroNombre.value.toLowerCase()) //Filtro por nombre
  }).sort((a, b) => {
    if (columnaOrdenada.value === 'idModule') {  // Por ID del modulo
      return ordenAscendente.value ? a.idModule - b.idModule : b.idModule - a.idModule //Si orden ascendente es true se ordena ascendente, sino descendente
    } else if (columnaOrdenada.value === 'entryDate') { //Por fecha de entrada
      const fechaA = new Date(a.entryDate)
      const fechaB = new Date(b.entryDate)
      return ordenAscendente.value ? fechaA - fechaB : fechaB - fechaA //Si orden ascendente es true se ordena ascendente, sino descendente
    }
  })
})

//Funcion para cambiar la columna por la cual ordeno
const ordenarPor = (columna) => {
  if (columna === columnaOrdenada.value) {
    ordenAscendente.value = !ordenAscendente.value //Si es la misma, altero el orden
  } else {
    columnaOrdenada.value = columna
  }
}
</script>

<style>
  th.fecha-entrada, th.id {
    cursor: pointer;
  }
</style>

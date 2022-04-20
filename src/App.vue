<template>

<button @click="getSelectedRows()">Obtener los registros seleccionados</button>
 <button v-on:click="onBtnExport()">Descargar CSV</button>
  <div class="example-header">
                    Mostrando registros:
                    <select v-on:change="onPageSizeChanged()" id="page-size">
                        <option value="10" selected="">10</option>
                        <option value="100">100</option>
                        <option value="500">500</option>
                        <option value="1000">1000</option>
                    </select>
                </div>
  <ag-grid-vue
    style="width: 100%; height: 600px"
    class="ag-theme-alpine"
    :columnDefs="columnDefs"
    :rowData="rowData.value"
    :localeText="localeText"
    :defaultColDef="defaultColDef"
    rowSelection="multiple"
    :pagination="true"
    :paginationPageSize="paginationPageSize"
    :paginationNumberFormatter="paginationNumberFormatter"
    @grid-ready="onGridReady"
  >
  </ag-grid-vue>
</template>

<script>
import {reactive,ref} from "vue";

import "ag-grid-community/dist/styles/ag-grid.css";
import "ag-grid-community/dist/styles/ag-theme-alpine.css";
import { AgGridVue } from "ag-grid-vue3"; //tabla principal
import check from './check.svg';
import { AG_GRID_LOCALE_ES } from './config_spanish.js'; // traduccion a español

export default {
  name: "App",
  components: {
    AgGridVue,
  },
  setup() {
      let rowData = reactive([]); // aqui se almacena toda la info de la tabla
      let gridApi = ref(null); // esta pendiente de toda la tabla
      let columnApi = ref(null);
      let paginationPageSize = 10; // mostramos la cantidad de resultados

    var localeText = AG_GRID_LOCALE_ES
    //Funcion para dar formato al páginado
    const paginationNumberFormatter = (params) => {
         console.log("Params", params)
      return params.value.toLocaleString();
    };
    
   

    //Exportar a Csv
     const onBtnExport = () =>{
      gridApi.value.exportDataAsCsv();
    }
    //Espera a que la data este lista
    const onGridReady = params => {

      gridApi.value = params.api;
      columnApi.value = params.columnApi;
     const updateData = (data) => {
         //Transformaciòn de los datos
        const dataTransformed = data.map(val => {
            const check = val.Cors == "yes" ? "✔️": val.Cors == "unknown" ? "©️" :"✖️"
            const data = {...val,Cors: check}
            return data
        });
        rowData.value = dataTransformed;

      };
        //Llamada a la api
      fetch("https://api.publicapis.org/entries")
          .then(result => result.json())
          .then(remoteRowData => updateData(remoteRowData.entries));
    
    };

    //Mostrando los registros del select
    const onPageSizeChanged = () => {
      var value = document.getElementById('page-size').value;
      gridApi.value.paginationSetPageSize(Number(value));
    }
    //obtenemos los datos seleccionados de la tabla
    const getSelectedRows = () => { 
      const selectedNodes = gridApi.value.getSelectedNodes();
      const selectedData = selectedNodes.map( node => node.data);
      console.log(selectedData)
      const selectedDataStringPresentation = selectedData.map( node => `${node.API} ${node.Auth}`).join(', ');
      alert(`Selected nodes: ${selectedDataStringPresentation}`);
    };

    return {
      columnDefs: [
        //headerName: Respresenta el nombre de la columna
        //field: representa el nombre de la propiedad que traemos de la API
        //sortable: puedes organizar la tabla ya sea desendente o ascendente
        //editable: Se edita la celda
        //filter: Agrea un buscador avanzado por columna.
        //checkboxSelection agrega checkboxes a la columna
        { headerName: "API", field: "API", editable: true, resizable: true, sortable: true, filter: true, checkboxSelection: true, },
        { headerName: "Description", field: "Description",editable: true,resizable: true, filter: true },
        { headerName: "Auth", field: "Auth",resizable: true, },
        { headerName: "HTTPS", field: "HTTPS", editable: true, sortable: true },
        { headerName: "Cors", field: "Cors", sortable: true, filter: true },
        { headerName: "Link", field: "Link" },
        { headerName: "Category", field: "Category" },
      ],
      rowData,
      onBtnExport,
      autoGroupColumnDef: {
       headerName: 'Model',
       field: 'model',
       cellRenderer: 'agGroupCellRenderer',
       cellRendererParams: {
         checkbox: true
       }
     },
     defaultColDef: { //EStas opciones se agregan a cada columna
        //Asigna el valor por defecto del ancho de la columna
        width: 150,
        //hace cada columna editable
        editable: true,
        //Agrega cada columna el campo para buscar
        filter: 'agTextColumnFilter',
        //habilita los filtros
        floatingFilter: true,
        //hace las columnas ajustables
        resizable: true,
      },
      onGridReady,
      getSelectedRows,
      paginationNumberFormatter,
      paginationPageSize,
      onPageSizeChanged,
      check,
      localeText
    };
  },
};
</script>

<style>
@import "~ag-grid-community/dist/styles/ag-grid.css";
@import "~ag-grid-community/dist/styles/ag-theme-alpine.css";


</style>

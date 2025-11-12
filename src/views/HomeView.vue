<template>
    <div class="container mt-2">
        <div class="general-header">
            <div class="header-app">
                <img :src="logotipo" :alt="logotipo" class="logo-app" />
                <h4 class="titulo-app">CONTROL DESCRIPCIÓN 2 PEDIDOS</h4>
            </div>
        </div>

        <!-- Sección para buscar por número -->
        <div class="seccion-carga-archivo mt-4">
            <div class="d-flex align-items-center gap-2">
                <div class="input-numero-wrapper">
                    <label for="numeroBusqueda" class="form-label fw-bold mb-2">Número de Pedido:</label>
                    <input 
                        type="text" 
                        id="numeroBusqueda" 
                        class="form-control form-control-sm" 
                        v-model="numeroBusqueda"
                        @input="validarSoloNumeros"
                        placeholder="Ingrese un número"
                        maxlength="20"
                    />
                </div>
                <button 
                    type="button" 
                    class="btn btn-primary btn-buscar btn-sm"
                    @click="buscarPorNumero"
                    :disabled="!numeroBusqueda"
                >
                    <span>🔍</span> Buscar
                </button>
            </div>
        </div>

        <!-- Sección de Resultados -->
        <div v-if="mostrarResultados" class="mt-4">
            <!-- Tabla de Documentos Actuales -->
            <div class="seccion-tabla mb-4">
                <div class="tabla-header mb-3">
                    <div class="d-flex justify-content-between align-items-center w-100">
                        <h5>📄 Referencias Actuales ({{documentosActuales.length}})</h5>
                        <button 
                            type="button" 
                            class="btn btn-success btn-sm"
                            @click="guardarDescripciones"
                            :disabled="!hayDescripcionesModificadas"
                        >
                            <span>💾</span> Guardar Descripciones
                        </button>
                    </div>
                </div>
                <div class="mb-3">
                    <input 
                        type="text" 
                        class="form-control form-control-sm" 
                        v-model="filtroCodigoActuales"
                        placeholder="🔍 Filtrar por código..."
                        style="max-width: 300px;"
                    />
                </div>
                <div class="tabla-scroll">
                    <table class="tabla-registros">
                        <thead>
                            <tr>
                                <th style="width: 12%;">Código</th>
                                <th style="width: 10%;">Cantidad</th>
                                <th style="width: 13%;">Valor Unitario</th>
                                <th style="width: 65%;">Descripción 2</th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr v-for="(doc, index) in documentosActualesFiltrados" :key="index">
                                <td>{{ doc.codigo }}</td>
                                <td>{{ doc.cantidad }}</td>
                                <td>{{ formatearMoneda(doc.valor_unitario) }}</td>
                                <td>
                                    <textarea 
                                        class="form-control form-control-sm descripcion-textarea"
                                        v-model="doc.descripcion2_editada"
                                        rows="2"
                                        placeholder="Ingrese la descripción..."
                                        @input="marcarComoModificado(doc)"
                                    ></textarea>
                                </td>
                            </tr>
                            <tr v-if="documentosActualesFiltrados.length === 0">
                                <td colspan="4" class="text-center text-muted py-4">
                                    No se encontraron documentos actuales
                                </td>
                            </tr>
                        </tbody>
                    </table>
                </div>
            </div>

            <!-- Tabla de Documentos Históricos -->
            <div class="seccion-tabla">
                <div class="tabla-header mb-3">
                    <h5>📚 Referencias Históricos ({{documentosHistoria.length}})</h5>
                </div>
                <div class="mb-3">
                    <input 
                        type="text" 
                        class="form-control form-control-sm" 
                        v-model="filtroCodigoHistoria"
                        placeholder="🔍 Filtrar por código..."
                        style="max-width: 300px;"
                    />
                </div>
                <div class="tabla-scroll">
                    <table class="tabla-registros">
                        <thead>
                            <tr>
                                <th>Código</th>
                                <th>Cantidad</th>
                                <th>Valor Unitario</th>
                                <th>Descripción 2</th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr v-for="(doc, index) in documentosHistoriaFiltrados" :key="index">
                                <td>{{ doc.codigo }}</td>
                                <td>{{ doc.cantidad }}</td>
                                <td>{{ formatearMoneda(doc.valor_unitario) }}</td>
                                <td>{{ doc.descripcion2 || 'Sin descripción' }}</td>
                            </tr>
                            <tr v-if="documentosHistoriaFiltrados.length === 0">
                                <td colspan="4" class="text-center text-muted py-4">
                                    No se encontraron documentos históricos
                                </td>
                            </tr>
                        </tbody>
                    </table>
                </div>
            </div>
        </div>

    </div>


    <!-- Modal de éxito -->
    <div class="modal fade" id="exitoModal" tabindex="-1" aria-labelledby="exitoModalLabel" aria-hidden="true" data-bs-backdrop="static" ref="exitoModal">
        <div class="modal-dialog modal-dialog-centered">
            <div class="modal-content">
                <div class="modal-header bg-success text-white">
                    <div class="d-flex align-items-center w-100">
                        <span class="me-2" style="font-size:2.5rem;line-height:1;">&#10004;</span>
                        <h5 class="modal-title flex-grow-1" id="exitoModalLabel">{{ modalTitle }}</h5>
                        <button type="button" class="btn-close btn-close-white" data-bs-dismiss="modal" aria-label="Close"></button>
                    </div>
                </div>
                <div class="modal-body text-success text-center">
                    <p>{{ msg }}</p>                    
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Cerrar</button>
                </div>
            </div>
        </div>
    </div>

    <!-- Modal de error -->
    <div class="modal fade" id="errorModal" tabindex="-1" aria-labelledby="errorModalLabel" aria-hidden="true" data-bs-backdrop="static" ref="errorModal">
        <div class="modal-dialog modal-dialog-centered">
            <div class="modal-content">
                <div class="modal-header bg-danger text-white">
                    <div class="d-flex align-items-center w-100">
                        <span class="me-2" style="font-size:2.5rem;line-height:1;">&#10006;</span>
                        <h5 class="modal-title flex-grow-1" id="errorModalLabel">Error</h5>
                        <button type="button" class="btn-close btn-close-white" data-bs-dismiss="modal" aria-label="Close"></button>
                    </div>
                </div>
                <div class="modal-body text-danger text-center">
                    <strong>{{ errorMsg }}</strong>
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Cerrar</button>
                </div>
            </div>
        </div>
    </div>
    
    <!-- Overlay de carga -->
    <div v-if="loading" class="loading-overlay">
        <div class="spinner-border text-light" role="status">
            <span class="visually-hidden"></span>
        </div>
        <p class="mt-2 text-light">{{ loading_msg }}</p>
    </div>
</template>

<script setup>

import { ref, onMounted, computed } from 'vue';
import axios from 'axios';
import { Modal } from 'bootstrap';
import logotipo from '@/assets/logotipo.png';
import apiUrl from "../../config.js";

// Variables para el manejo de la búsqueda por número
const numeroBusqueda = ref('');

// Variables para resultados de búsqueda
const mostrarResultados = ref(false);
const documentosActuales = ref([]);
const documentosHistoria = ref([]);
const filtroCodigoActuales = ref('');
const filtroCodigoHistoria = ref('');

const msg = ref('');
const errorMsg = ref('');
const modalTitle = ref('');

const modalInstance = ref(null);
const modalErrorInstance = ref(null);

const loading = ref(false);
const loading_msg = ref('');

// Computed properties para filtrar las tablas
const documentosActualesFiltrados = computed(() => {
    if (!filtroCodigoActuales.value) {
        return documentosActuales.value;
    }
    return documentosActuales.value.filter(doc => 
        doc.codigo.toLowerCase().includes(filtroCodigoActuales.value.toLowerCase())
    );
});

const documentosHistoriaFiltrados = computed(() => {
    if (!filtroCodigoHistoria.value) {
        return documentosHistoria.value;
    }
    return documentosHistoria.value.filter(doc => 
        doc.codigo.toLowerCase().includes(filtroCodigoHistoria.value.toLowerCase())
    );
});

// Computed para verificar si hay descripciones modificadas
const hayDescripcionesModificadas = computed(() => {
    return documentosActuales.value.some(doc => doc.modificado === true);
});

// Función para formatear valores monetarios
const formatearMoneda = (valor) => {
    if (valor === null || valor === undefined) return '$0.00';
    return new Intl.NumberFormat('es-CO', {
        style: 'currency',
        currency: 'COP',
        minimumFractionDigits: 2,
        maximumFractionDigits: 2
    }).format(valor);
};

// Función para validar que solo se ingresen números
const validarSoloNumeros = (event) => {
    // Eliminar cualquier carácter que no sea número
    numeroBusqueda.value = numeroBusqueda.value.replace(/[^0-9]/g, '');
};

// Función para marcar un documento como modificado
const marcarComoModificado = (doc) => {
    doc.modificado = true;
};

// Función para guardar las descripciones modificadas
const guardarDescripciones = async () => {
    // Filtrar solo los documentos que fueron modificados y tienen descripción
    const descripcionesModificadas = documentosActuales.value
        .filter(doc => doc.modificado && doc.descripcion2_editada && doc.descripcion2_editada.trim() !== '')
        .map(doc => ({
            codigo: doc.codigo,
            valor_unitario: doc.valor_unitario,
            cantidad: doc.cantidad,
            descripcion2: doc.descripcion2_editada.trim()
        }));

    if (descripcionesModificadas.length === 0) {
        errorMsg.value = 'No hay descripciones para guardar.';
        modalErrorInstance.value.show();
        return;
    }

    try {
        loading.value = true;
        loading_msg.value = 'Guardando descripciones...';

        const response = await axios.post(
            `${apiUrl}/descripcion/actualizar-descripciones`,
            {
                numero: numeroBusqueda.value,
                descripciones: descripcionesModificadas
            },
            {
                headers: {
                    Accept: "application/json",
                }
            }
        );

        if (response.status === 200) {
            msg.value = response.data.message || 'Descripciones guardadas exitosamente';
            modalTitle.value = 'Operación Exitosa';
            modalInstance.value.show();
            
            // Actualizar los documentos para reflejar los cambios guardados
            documentosActuales.value.forEach(doc => {
                if (doc.modificado) {
                    doc.descripcion2 = doc.descripcion2_editada;
                    doc.modificado = false;
                }
            });
            await buscarPorNumero(); // Refrescar los datos desde el backend
        }
    } catch (error) {
        console.error(error);
        errorMsg.value = error.response?.data?.message || 'Error al guardar las descripciones.';
        modalErrorInstance.value.show();
    } finally {
        loading.value = false;
        loading_msg.value = '';
    }
};

// Función para buscar por número
const buscarPorNumero = async () => {
    if (!numeroBusqueda.value) {
        errorMsg.value = 'Por favor, ingrese un número de pedido.';
        modalErrorInstance.value.show();
        return;
    }

    try {
        loading.value = true;
        loading_msg.value = 'Buscando...';

        // Realizar búsqueda en el backend
        const response = await axios.post(
            `${apiUrl}/descripcion/buscar`,
            {
                numero: numeroBusqueda.value
            },
            {
                headers: {
                    Accept: "application/json",
                }
            }
        );

        if (response.status === 200) {
            const data = response.data.data;
            
            // Asignar los resultados a las variables reactivas
            // Agregar campo editable y flag de modificado a cada documento actual
            documentosActuales.value = (data.documentos_actuales || []).map(doc => ({
                ...doc,
                descripcion2_editada: doc.descripcion2 || '',
                modificado: false
            }));
            documentosHistoria.value = data.documentos_historia || [];
            
            // Mostrar las tablas de resultados
            mostrarResultados.value = true;
            
            // Limpiar filtros
            filtroCodigoActuales.value = '';
            filtroCodigoHistoria.value = '';
            
            // msg.value = response.data.message || 'Búsqueda exitosa';
            // modalTitle.value = 'Operación Exitosa';
            // modalInstance.value.show();
        }
    } catch (error) {
        console.error(error);
        errorMsg.value = error.response?.data?.message || 'Error al realizar la búsqueda.';
        modalErrorInstance.value.show();
        
        // Ocultar resultados en caso de error
        mostrarResultados.value = false;
        documentosActuales.value = [];
        documentosHistoria.value = [];
    } finally {
        loading.value = false;
        loading_msg.value = '';
    }
};

// Código que se ejecuta al montar el componente
onMounted(async () => {
    modalInstance.value = new Modal(exitoModal);
    modalErrorInstance.value = new Modal(errorModal);
});

</script>

<style scoped>

.general-header {
    display: flex;
    justify-content: flex-start;
}

.header-app {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 1rem;
    margin-top: 2rem;
    margin-bottom: 1rem;
}

.logo-app {
    height: 48px;
    width: auto;
    object-fit: contain;
}

.titulo-app {
    margin-bottom: 0;
    font-weight: bold;
}

.container {
    max-width: 95vw;
    width: 98vw;
    margin: 0 auto;
}

/* Estilos para la sección de carga de archivo */
.seccion-carga-archivo {
    background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
    padding: 20px;
    border-radius: 10px;
    box-shadow: 0 2px 8px rgba(0,0,40,0.1);
}

.input-numero-wrapper {
    flex: 0 0 auto;
    max-width: 250px;
}

.input-numero-wrapper .form-label {
    color: #2c3e50;
    font-size: 0.85rem;
    margin-bottom: 4px;
}

.input-numero-wrapper .form-control-sm {
    border: 2px solid #3498db;
    border-radius: 5px;
    padding: 6px 10px;
    font-size: 0.85rem;
    height: 32px;
}

.input-numero-wrapper .form-control-sm:focus {
    border-color: #2874a6;
    box-shadow: 0 0 0 0.15rem rgba(52, 152, 219, 0.25);
}

.btn-buscar {
    margin-top: 24px;
    background: linear-gradient(90deg, #3498db 0%, #2874a6 100%);
    border: none;
    padding: 6px 14px;
    font-weight: 600;
    font-size: 0.85rem;
    border-radius: 5px;
    display: flex;
    align-items: center;
    gap: 6px;
    transition: all 0.3s ease;
    box-shadow: 0 2px 4px rgba(52, 152, 219, 0.3);
    height: 32px;
    white-space: nowrap;
}

.btn-buscar:hover:not(:disabled) {
    background: linear-gradient(90deg, #2874a6 0%, #1f618d 100%);
    transform: translateY(-1px);
    box-shadow: 0 3px 8px rgba(52, 152, 219, 0.4);
}

.btn-buscar:disabled {
    background: #95a5a6;
    cursor: not-allowed;
    opacity: 0.6;
}

.btn-buscar span {
    font-size: 1rem;
}

/* Estilos para la sección de enviar correo */
.seccion-enviar-correo {
    background: linear-gradient(135deg, #fff3e0 0%, #ffe0b2 100%);
    padding: 20px;
    border-radius: 10px;
    box-shadow: 0 2px 8px rgba(0,0,40,0.1);
}

.btn-enviar-correo {
    background: linear-gradient(90deg, #ff9800 0%, #f57c00 100%);
    border: none;
    padding: 12px 30px;
    font-weight: 600;
    font-size: 1rem;
    border-radius: 6px;
    display: inline-flex;
    align-items: center;
    gap: 10px;
    transition: all 0.3s ease;
    box-shadow: 0 2px 6px rgba(255, 152, 0, 0.3);
    color: white;
}

.btn-enviar-correo:hover:not(:disabled) {
    background: linear-gradient(90deg, #f57c00 0%, #e65100 100%);
    transform: translateY(-2px);
    box-shadow: 0 4px 10px rgba(255, 152, 0, 0.4);
    color: white;
}

.btn-enviar-correo:disabled {
    background: #95a5a6;
    cursor: not-allowed;
    opacity: 0.6;
}

.btn-enviar-correo span {
    font-size: 1.3rem;
}

/* Estilos para la sección de filtros */
.seccion-filtros {
    background: linear-gradient(135deg, #e8f5e9 0%, #c8e6c9 100%);
    padding: 20px;
    border-radius: 10px;
    box-shadow: 0 2px 8px rgba(0,0,40,0.1);
}

.seccion-filtros h5 {
    color: #2e7d32;
    font-weight: 700;
    margin-bottom: 15px;
}

.seccion-filtros .form-label {
    font-weight: 600;
    color: #1b5e20;
    font-size: 0.9rem;
    margin-bottom: 5px;
}

.seccion-filtros .form-control {
    border: 2px solid #66bb6a;
    border-radius: 6px;
    font-size: 0.9rem;
}

.seccion-filtros .form-control:focus {
    border-color: #43a047;
    box-shadow: 0 0 0 0.2rem rgba(76, 175, 80, 0.25);
}

.seccion-filtros .btn {
    font-weight: 600;
    padding: 8px 20px;
    border-radius: 6px;
    display: inline-flex;
    align-items: center;
    gap: 8px;
}

/* Estilos para la tabla de registros */
.seccion-tabla {
    background: white;
    padding: 20px;
    border-radius: 10px;
    box-shadow: 0 2px 8px rgba(0,0,40,0.1);
}

.tabla-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.tabla-header h5 {
    color: #1976d2;
    font-weight: 700;
    margin: 0;
}

.tabla-header .badge {
    font-size: 0.95rem;
    padding: 8px 15px;
}

.detalle-title{
    display: flex;
    justify-content: space-between;
}

.btn-excel {
    background-color: #d6f5e3;
    color: #217346;
    border: 1px solid #217346;
    border-radius: 5px;
    padding: 8px 18px 8px 14px;
    font-size: 1em;
    font-weight: 600;
    display: flex;
    align-items: center;
    gap: 6px;
    cursor: pointer;
    transition: background 0.2s, color 0.2s, border 0.2s;
    box-shadow: 0 2px 6px rgba(33,115,70,0.08);
}
.btn-excel:hover {
    background-color: #b6e6c9;
    color: #145c2c;
    border-color: #145c2c;
}

/* Tabla de registros - estilos modernos */
.tabla-scroll {
    max-height: 500px;
    overflow-x: auto;
    overflow-y: auto;
    border-radius: 10px;
    box-shadow: 0 2px 8px rgba(0,0,40,0.08);
}

.tabla-registros {
    width: 100%;
    border-collapse: separate;
    border-spacing: 0;
    background: white;
    border-radius: 10px;
    overflow: hidden;
    font-size: 0.8rem;
}

.tabla-registros th {
    background: linear-gradient(90deg, #1976d2 0%, #1565c0 100%);
    color: #fff;
    font-weight: 600;
    text-align: center;
    padding: 12px 10px;
    font-size: 0.8rem;
    border-bottom: 2px solid #0d47a1;
    position: sticky;
    top: 0;
    z-index: 10;
}

.tabla-registros td {
    text-align: center;
    padding: 10px 8px;
    font-size: 0.75rem;
    color: #424242;
    background: #fafafa;
    border-bottom: 1px solid #e0e0e0;
    white-space: nowrap;
}

.tabla-registros tbody tr:nth-child(even) td {
    background: #f5f5f5;
}

.tabla-registros tbody tr:hover td {
    background: #e3f2fd;
    transition: background 0.2s;
}

/* Estilos para textarea de descripción */
.descripcion-textarea {
    width: 100%;
    resize: vertical;
    min-height: 50px;
    font-size: 0.75rem;
    padding: 6px 8px;
    border: 1px solid #cbd5e0;
    border-radius: 4px;
}

.descripcion-textarea:focus {
    border-color: #3498db;
    box-shadow: 0 0 0 0.15rem rgba(52, 152, 219, 0.25);
    outline: none;
}

/* Botón de editar */
.btn-editar {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    border: none;
    padding: 6px 12px;
    font-size: 1.1rem;
    border-radius: 5px;
    cursor: pointer;
    transition: all 0.3s ease;
    box-shadow: 0 2px 4px rgba(102, 126, 234, 0.3);
}

.btn-editar:hover {
    background: linear-gradient(135deg, #764ba2 0%, #667eea 100%);
    transform: translateY(-2px);
    box-shadow: 0 4px 8px rgba(102, 126, 234, 0.4);
}

/* Estilos del modal de edición */
.modal-xl {
    max-width: 95%;
}

/* Z-index para modales de éxito y error por encima del modal de edición */
#exitoModal.modal,
#errorModal.modal {
    z-index: 1070 !important;
}

#exitoModal ~ .modal-backdrop,
#errorModal ~ .modal-backdrop {
    z-index: 1065 !important;
}

.seccion-titulo {
    color: #1976d2;
    font-weight: 700;
    border-bottom: 2px solid #1976d2;
    padding-bottom: 8px;
}

.seccion-readonly {
    background: #f5f5f5;
    padding: 20px;
    border-radius: 8px;
    border-left: 4px solid #9e9e9e;
}

.seccion-readonly .form-control {
    background-color: #e0e0e0;
    border: 1px solid #bdbdbd;
    cursor: not-allowed;
    font-weight: 500;
    color: #424242;
}

.seccion-editable {
    background: #e8f5e9;
    padding: 20px;
    border-radius: 8px;
    border-left: 4px solid #4caf50;
}

.form-control-editable {
    border: 2px solid #66bb6a;
    font-weight: 600;
}

.form-control-editable:focus {
    border-color: #43a047;
    box-shadow: 0 0 0 0.2rem rgba(76, 175, 80, 0.25);
}

/* Estilos para la paginación */
.paginacion {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 15px 0;
    flex-wrap: wrap;
    gap: 15px;
}

.paginacion-info {
    color: #616161;
    font-size: 0.9rem;
    font-weight: 500;
}

.paginacion-controles {
    display: flex;
    align-items: center;
    gap: 8px;
    flex-wrap: wrap;
}

.paginacion-controles .btn {
    padding: 6px 12px;
    font-size: 0.85rem;
    font-weight: 600;
}

.paginacion-numeros {
    font-weight: 600;
    color: #1976d2;
    padding: 0 10px;
    font-size: 0.9rem;
}

/* Estilos para mensaje de sin datos */
.sin-datos {
    padding: 40px 20px;
}

.icono-sin-datos {
    font-size: 4rem;
    display: block;
    opacity: 0.5;
}

.sin-datos h5 {
    color: #616161;
    font-weight: 600;
}

.sin-datos p {
    font-size: 0.95rem;
}

/* Overlay de carga */
.loading-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    background: rgba(44, 62, 80, 0.45);
    z-index: 9999;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
}

.loading-overlay .spinner-border {
    width: 3rem;
    height: 3rem;
    border-width: 0.35em;
}

.loading-overlay p {
    color: #fff;
    font-size: 1.15rem;
    margin-top: 1.2rem;
    text-align: center;
    text-shadow: 0 1px 4px rgba(0,0,0,0.18);
}

/* Estilos para la tabla de verificación */
.tabla-verificacion-container {
    margin-top: 20px;
    margin-bottom: 20px;
    overflow-x: auto;
}

.tabla-verificacion {
    width: 100%;
    border-collapse: collapse;
    background: white;
    box-shadow: 0 2px 8px rgba(0,0,40,0.08);
    border-radius: 8px;
    overflow: hidden;
}

.tabla-verificacion thead tr {
    background: linear-gradient(90deg, #5dade2 0%, #3498db 100%);
    color: white;
}

.tabla-verificacion th {
    padding: 14px 12px;
    text-align: center;
    font-weight: 700;
    font-size: 0.95rem;
    border: 1px solid #2e86c1;
    text-transform: uppercase;
}

.tabla-verificacion .col-aspecto {
    background: linear-gradient(90deg, #2874a6 0%, #1f618d 100%);
    width: 55%;
    text-align: left;
}

.tabla-verificacion .col-opcion {
    background: linear-gradient(90deg, #2874a6 0%, #1f618d 100%);
    width: 15%;
}

.tabla-verificacion tbody tr.seccion-header {
    background: linear-gradient(90deg, #2874a6 0%, #1f618d 100%);
    color: white;
    font-weight: bold;
}

.tabla-verificacion tbody tr.seccion-header td {
    padding: 12px 15px;
    font-size: 0.95rem;
    border: 1px solid #1b4f72;
    text-align: left;
    font-weight: 700;
}

.tabla-verificacion tbody tr:not(.seccion-header) {
    background: #ebf5fb;
}

.tabla-verificacion tbody tr:not(.seccion-header):nth-child(even) {
    background: #d6eaf8;
}

.tabla-verificacion tbody tr:not(.seccion-header):hover {
    background: #aed6f1;
    transition: background 0.2s;
}

.tabla-verificacion td {
    padding: 12px 15px;
    border: 1px solid #aed6f1;
    text-align: center;
}

.tabla-verificacion .aspecto-cell {
    text-align: left;
    color: #21618c;
    padding-left: 15px;
    font-size: 0.92rem;
    line-height: 1.5;
}

.tabla-verificacion .numero-badge {
    display: inline-block;
    background: linear-gradient(135deg, #3498db 0%, #2874a6 100%);
    color: white;
    padding: 4px 10px;
    border-radius: 4px;
    font-weight: 700;
    font-size: 0.85rem;
    margin-right: 10px;
    min-width: 35px;
    text-align: center;
}

.tabla-verificacion .opcion-cell input[type="radio"] {
    cursor: pointer;
    width: 20px;
    height: 20px;
    accent-color: #3498db;
}

.tabla-verificacion .opcion-cell {
    background: rgba(255, 255, 255, 0.7);
}
</style>
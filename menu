const readline = require("readline");
const fs = require("fs").promises;
const path = require("path");

const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout,
});

const ARCHIVO_PERSISTENCIA = path.join(__dirname, "tareas.json");
const MAX_TAREAS = 100;
const NOMBRE_USUARIO = "Olivia";

const ESTADOS = Object.freeze({
  PENDIENTE: "Pendiente",
  EN_CURSO: "En curso",
  TERMINADA: "Terminada",
  CANCELADA: "Cancelada"
});

const DIFICULTADES = Object.freeze({
  FACIL: "+--",
  MEDIA: "++-",
  DIFICIL: "+++"
});

const crearTarea = (titulo, descripcion, vencimiento) => 
  Object.freeze({
    titulo,
    descripcion,
    estado: ESTADOS.PENDIENTE,
    dificultad: DIFICULTADES.FACIL,
    vencimiento,
    creacion: new Date().toLocaleDateString("es-AR"),
    ultimaEdicion: new Date().toLocaleDateString("es-AR")
  });

const agregarTarea = (tareas, nuevaTarea) => {
  if (tareas.length >= MAX_TAREAS) {
    return tareas;
  }
  return Object.freeze([...tareas, nuevaTarea]);
};

const actualizarTarea = (tareas, index, cambios) => {
  const tareaActualizada = Object.freeze({
    ...tareas[index],
    ...cambios,
    ultimaEdicion: new Date().toLocaleDateString("es-AR")
  });
  
  return Object.freeze(
    tareas.map((tarea, i) => i === index ? tareaActualizada : tarea)
  );
};

const eliminarTarea = (tareas, index) =>
  Object.freeze(tareas.filter((_, i) => i !== index));

const filtrarPorEstado = (tareas, estado) =>
  tareas.filter(tarea => tarea.estado === estado);

const obtenerTodasLasTareas = (tareas) => [...tareas];

const buscarPorTitulo = (tareas, query) =>
  tareas.filter(tarea => 
    tarea.titulo.toLowerCase().includes(query.toLowerCase())
  );

const obtenerTarea = (tareas, index) => tareas[index];

const puedeAgregarTarea = (tareas) => tareas.length < MAX_TAREAS;

const contarPorEstado = (tareas, estado) =>
  tareas.reduce((count, tarea) => 
    tarea.estado === estado ? count + 1 : count, 0);

const mapearConIndices = (tareas) =>
  tareas.map((tarea, index) => ({ ...tarea, indiceOriginal: index }));

const parsearEstado = (input) => {
  const estados = {
    'p': ESTADOS.PENDIENTE,
    'e': ESTADOS.EN_CURSO,
    't': ESTADOS.TERMINADA,
    'c': ESTADOS.CANCELADA
  };
  return estados[input.toLowerCase()] || null;
};

const parsearDificultad = (input) => {
  const dificultades = {
    '1': DIFICULTADES.FACIL,
    '2': DIFICULTADES.MEDIA,
    '3': DIFICULTADES.DIFICIL
  };
  return dificultades[input] || null;
};

const formatearTareaLista = (tarea, numero) =>
  `[${numero}] ${tarea.titulo} | Estado: ${tarea.estado} | Vencimiento: ${tarea.vencimiento}`;

const formatearTareaDetalle = (tarea) => `
--- Detalle de la Tarea ---

Título:         ${tarea.titulo}
Descripción:    ${tarea.descripcion}
Estado:         ${tarea.estado}
Dificultad:     ${tarea.dificultad}
Vencimiento:    ${tarea.vencimiento}
Creación:       ${tarea.creacion}
Última Edición: ${tarea.ultimaEdicion}
`;

const componer = (...funciones) =>
  (valor) => funciones.reduceRight((acc, fn) => fn(acc), valor);

const mapear = (fn) => (array) => array.map(fn);

const filtrar = (predicado) => (array) => array.filter(predicado);

const existe = (predicado) => (array) => array.some(predicado);

const contar = (predicado) => (array) =>
  array.reduce((acc, item) => predicado(item) ? acc + 1 : acc, 0);

const limpiarPantalla = () => {
  console.clear();
};

const preguntar = (pregunta) =>
  new Promise((resolve) => rl.question(pregunta, resolve));

const pausar = () => preguntar("Presiona en enter para continuar");

const mostrarMensaje = (mensaje) => {
  console.log(mensaje);
};

const mostrarError = (mensaje) => {
  console.log(` ${mensaje}`);
};

const mostrarExito = (mensaje) => {
  console.log(` ${mensaje}`);
};

const mostrarMenuPrincipal = async (tareas) => {
  limpiarPantalla();
  mostrarMensaje(`¡Hola ${NOMBRE_USUARIO}!\n`);
  mostrarMensaje("¿Qué deseas hacer?\n");
  mostrarMensaje("[1] Ver Mis Tareas");
  mostrarMensaje("[2] Buscar una Tarea");
  mostrarMensaje("[3] Agregar una Tarea");
  mostrarMensaje("[0] Salir\n");

  const opcion = await preguntar("> ");
  
  switch (opcion.trim()) {
    case "1":
      return await mostrarMenuFiltrarTareas(tareas);
    case "2":
      return await buscarTareaInteractivo(tareas);
    case "3":
      return await crearTareaInteractivo(tareas);
    case "0":
      mostrarMensaje("Cerrando menu");
      rl.close();
      process.exit(0);
    default:
      mostrarError("Opcion no valida.");
      await pausar();
      return await mostrarMenuPrincipal(tareas);
  }
};

const mostrarMenuFiltrarTareas = async (tareas) => {
  limpiarPantalla();
  mostrarMensaje("¿Qué tareas deseas ver?\n");
  mostrarMensaje("[1] Todas");
  mostrarMensaje("[2] Pendientes");
  mostrarMensaje("[3] En curso");
  mostrarMensaje("[4] Terminadas");
  mostrarMensaje("[0] Volver\n");

  const opcion = await preguntar("> ");
  
  switch (opcion.trim()) {
    case "1":
      return await listarTareasInteractivo(tareas, null);
    case "2":
      return await listarTareasInteractivo(tareas, ESTADOS.PENDIENTE);
    case "3":
      return await listarTareasInteractivo(tareas, ESTADOS.EN_CURSO);
    case "4":
      return await listarTareasInteractivo(tareas, ESTADOS.TERMINADA);
    case "0":
      return await mostrarMenuPrincipal(tareas);
    default:
      mostrarError("Opcion no valida.");
      await pausar();
      return await mostrarMenuFiltrarTareas(tareas);
  }
};

const listarTareasInteractivo = async (tareas, filtroEstado) => {
  limpiarPantalla();
  
  const titulo = filtroEstado 
    ? ` Tareas ${filtroEstado} `
    : "Todas tus tareas ";
  
  mostrarMensaje(titulo);
  
  const tareasFiltradas = filtroEstado 
    ? filtrarPorEstado(tareas, filtroEstado)
    : obtenerTodasLasTareas(tareas);
  
  if (tareasFiltradas.length === 0) {
    mostrarMensaje("No hay tareas para mostrar");
    await pausar();
    return await mostrarMenuFiltrarTareas(tareas);
  }
  
  const tareasConIndices = mapearConIndices(tareasFiltradas);
  
  tareasConIndices.forEach((tarea, i) => {
    mostrarMensaje(formatearTareaLista(tarea, i + 1));
  });
  
  const seleccion = await preguntar("¿Ver detalles de alguna tarea?  ");
  const numero = parseInt(seleccion);
  
  if (numero > 0 && numero <= tareasConIndices.length) {
    const indiceOriginal = tareasConIndices[numero - 1].indiceOriginal;
    return await mostrarDetalleTareaInteractivo(tareas, indiceOriginal);
  }
  
  return await mostrarMenuFiltrarTareas(tareas);
};

const buscarTareaInteractivo = async (tareas) => {
  limpiarPantalla();
  const query = await preguntar("Introduce el titulo de la tarea: ");
  
  const resultados = buscarPorTitulo(tareas, query);
  
  if (resultados.length === 0) {
    mostrarMensaje("No se encontraron tareas.");
    await pausar();
    return await mostrarMenuPrincipal(tareas);
  }
  
  mostrarMensaje(" Resultados ");
  
  const resultadosConIndices = mapearConIndices(resultados);
  
  resultadosConIndices.forEach((tarea, i) => {
    mostrarMensaje(`[${i + 1}] ${tarea.titulo}`);
  });
  
  const seleccion = await preguntar("Desea ver alguna tarea): ");
  const numero = parseInt(seleccion);
  
  if (numero > 0 && numero <= resultadosConIndices.length) {
    const indiceOriginal = resultadosConIndices[numero - 1].indiceOriginal;
    return await mostrarDetalleTareaInteractivo(tareas, indiceOriginal);
  }
  
  return await mostrarMenuPrincipal(tareas);
};

const crearTareaInteractivo = async (tareas) => {
  if (!puedeAgregarTarea(tareas)) {
    mostrarError("No se pueden agregar mas tareas.");
    await pausar();
    return await mostrarMenuPrincipal(tareas);
  }
  
  limpiarPantalla();
  
  const titulo = await preguntar("Nombre de la tarea: ");
  const descripcion = await preguntar("Descripcion: ");
  const vencimiento = await preguntar("Fecha de vencimiento (DD/MM/AAAA): ");
  
  const nuevaTarea = crearTarea(titulo, descripcion, vencimiento);
  const tareasActualizadas = agregarTarea(tareas, nuevaTarea);
  
  mostrarExito(`¡Tarea '${titulo}' creada exitosamente!`);
  await pausar();
  
  return await mostrarMenuPrincipal(tareasActualizadas);
};

const mostrarDetalleTareaInteractivo = async (tareas, index) => {
  const tarea = obtenerTarea(tareas, index);
  
  limpiarPantalla();
  mostrarMensaje(formatearTareaDetalle(tarea));
  mostrarMensaje("\n[E] Editar | [D] Eliminar | [0] Volver");
  
  const opcion = await preguntar("> ");
  
  if (opcion.toLowerCase() === "e") {
    return await editarTareaInteractivo(tareas, index);
  } else if (opcion.toLowerCase() === "d") {
    return await eliminarTareaInteractivo(tareas, index);
  } else {
    return await mostrarMenuPrincipal(tareas);
  }
};

const editarTareaInteractivo = async (tareas, index) => {
  const tarea = obtenerTarea(tareas, index);
  const cambios = {};
  
  const descripcion = await preguntar(
    `Nueva descripción (ENTER para mantener actual: ${tarea.descripcion}):\n> `
  );
  if (descripcion.trim() !== "") {
    cambios.descripcion = descripcion;
  }
  
  const estadoInput = await preguntar(
    "Nuevo estado [P]endiente/[E]n curso/[T]erminada/[C]ancelada:\n> "
  );
  const nuevoEstado = parsearEstado(estadoInput);
  if (nuevoEstado) {
    cambios.estado = nuevoEstado;
  }
  
  const dificultadInput = await preguntar(
    "Nueva dificultad [1] fácil/[2] media/[3] difícil:\n> "
  );
  const nuevaDificultad = parsearDificultad(dificultadInput);
  if (nuevaDificultad) {
    cambios.dificultad = nuevaDificultad;
  }
  
  const vencimiento = await preguntar(
    `Nuevo vencimiento (actual ${tarea.vencimiento}):\n> `
  );
  if (vencimiento.trim() !== "") {
    cambios.vencimiento = vencimiento;
  }
  
  const tareasActualizadas = actualizarTarea(tareas, index, cambios);
  
  mostrarExito("Tarea actualizada");
  await pausar();
  
  return await mostrarDetalleTareaInteractivo(tareasActualizadas, index);
};

const eliminarTareaInteractivo = async (tareas, index) => {
  const confirmacion = await preguntar("¿Quieres eliminarla? : ");
  
  if (confirmacion.toLowerCase() === "s") {
    const tareasActualizadas = eliminarTarea(tareas, index);
    mostrarExito("Tarea eliminada");
    await pausar();
    return await mostrarMenuPrincipal(tareasActualizadas);
  }
  
  return await mostrarMenuPrincipal(tareas);
};

const iniciarAplicacion = async () => {
  const tareasIniciales = Object.freeze([]);
  await mostrarMenuPrincipal(tareasIniciales);
};

iniciarAplicacion();

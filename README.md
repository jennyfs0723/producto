# producto
Triggers y Funciones

DESCRIPCION DEL PROYECTO: "LogiTrans es un sistema de gestión logística diseñado para registrar envíos, asignar rutas óptimas, verificar disponibilidad de conductores y vehículos, y analizar estadísticas de tiempos, costos e incidencias. El proyecto busca optimizar la operación de transporte y brindar información clara para la toma de decisiones."

INSTRUCCIONES DE USO:

1. Ejecutar el script en MySQL para crear las tablas, vistas y procedimientos.
2. Insertar los seeders
3. Ejecutar los triggers y las funciones

ESTRUCTURA DE LOS MODULOS IMPLEMENTADOS:
Triggers
Gestión de Envíos: El trigger TR_ActualizarEstadoEnvio sincroniza automáticamente el estado global del envío con el último reporte de seguimiento.
Licencia conductor: TR_VerificarLicenciaConductor actúa como un filtro de seguridad, validando la categoría de la licencia contra el tipo de vehículo.
km vehiculo: Los triggers TR_ActualizarKilometrajeVehiculo y TR_GenerarAlertaMantenimiento automatizan el mantenimiento de la flota, acumulando el recorrido y disparando alertas de servicio (aceite, general o incidentes).
Costos de la operacion: TR_CalcularCostosRuta registra los gastos asociados (combustible y base) en las tablas de auditoría para control administrativo.
FUNCIONES:
asignaciones: FN_ObtenerVehiculoOptimo y FN_VerificarCapacidadVehiculo permiten filtrar la flota por capacidad de carga y disponibilidad técnica.
Cálculos: FN_CalcularDistanciaRuta y FN_EstimarTiempoEntrega sirve para proyectar kilómetros y horas de llegada.
Tarifa: FN_CalcularCostoEnvio formula el cobrod e acuerdo a la urgencia del servicio.

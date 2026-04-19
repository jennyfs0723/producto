CREATE DATABASE logitrans;

USE logitrans;

CREATE TABLE vehiculo( 
vehiculoID INT NOT NULL AUTO_INCREMENT, 
registro_unico VARCHAR (255) NOT NULL UNIQUE, 
marca VARCHAR (255) NOT NULL, 
modelo VARCHAR(255) NOT NULL, 
anio YEAR NOT NULL, 
capacidad_carga DECIMAL (10,2) NOT NULL, 
consumo_promedio_combustible DECIMAL(10,2) NOT NULL, 
numero_placa VARCHAR(255) NOT NULL UNIQUE , 
fecha_adquisicion DATE NOT NULL, 
km_actual DECIMAL (10,2) NOT NULL,
fecha_soat DATE NOT NULL,
fecha_tecnomecanica DATE NOT NULL, 
estado_operativo ENUM('activo','en mantenimiento'), 
tipo_vehiculo ENUM('camion','furgoneta','motocicleta'),
transporte_especial ENUM ('si', 'no') NOT NULL DEFAULT 'no', 
PRIMARY KEY(vehiculoID));

CREATE TABLE conductores( 
conductorID INT NOT NULL AUTO_INCREMENT, 
codigo_empleado VARCHAR (255) NOT NULL UNIQUE, 
nombres VARCHAR (255) NOT NULL, apellidos VARCHAR(255) NOT NULL, 
documento_identidad VARCHAR(255) NOT NULL UNIQUE, 
fecha_nacimiento DATE NOT NULL, direccion VARCHAR(255) NOT NULL, 
telefono VARCHAR(255) NOT NULL, tipo_licencia ENUM('C2','A1','A2'), 
fecha_vencimiento_licencia DATE NOT NULL, anios_experiencia INT NOT NULL, 
certificacion_especial VARCHAR(255), estado_conductor ENUM('disponible','en ruta','descanso') NOT NULL, 
PRIMARY KEY(conductorID));


-- Tabla intermedia para relación muchos a muchos 
CREATE TABLE conductor_vehiculo 
( conductor_vehiculoID INT NOT NULL AUTO_INCREMENT, 
conductorID INT NOT NULL, vehiculoID INT NOT NULL, 
PRIMARY KEY (conductor_vehiculoID), 
CONSTRAINT fk_cv_conductor FOREIGN KEY (conductorID) REFERENCES conductores(conductorID) ON DELETE CASCADE,
 CONSTRAINT fk_cv_vehiculo FOREIGN KEY (vehiculoID) REFERENCES vehiculo(vehiculoID) ON DELETE CASCADE);

CREATE TABLE cliente( 
clienteID INT NOT NULL AUTO_INCREMENT, 
codigo_unico VARCHAR(255) NOT NULL UNIQUE, 
tipo_cliente ENUM('individual','empresarial') NOT NULL, 
nombre_o_razon_social VARCHAR(255) NOT NULL, 
rut_o_dni VARCHAR(255) NOT NULL UNIQUE, 
direccion_fiscal VARCHAR(255) NOT NULL, 
direccion_recogida VARCHAR(255) NOT NULL, 
telefono VARCHAR (255) NOT NULL, 
email VARCHAR(255) NOT NULL, 
persona_contacto VARCHAR(255) NOT NULL, 
termino_pago VARCHAR (255) NOT NULL, 
clasificacion_por_envio VARCHAR(255) NOT NULL, 
PRIMARY KEY (clienteID));

CREATE TABLE envios( 
envioID INT NOT NULL AUTO_INCREMENT, 
numero_guia VARCHAR (255) NOT NULL UNIQUE, 
fecha_hora_recepcion DATETIME NOT NULL, 
id_cliente INT NOT NULL, 
id_conductor INT NOT NULL, 
id_vehiculo INT NOT NULL, 
remitente VARCHAR (255) NOT NULL, 
destinatario VARCHAR(255) NOT NULL, 
direccion_origen VARCHAR(255) NOT NULL, 
direccion_destino VARCHAR (255) NOT NULL, 
tipo_servicio ENUM('normal','express','mismo dia'), 
descripcion_contenido VARCHAR(255) NOT NULL, 
peso DECIMAL (10,2) NOT NULL, 
dimensiones VARCHAR(255) NOT NULL, 
valor_declarado DECIMAL (10,2) NOT NULL, 
costo DECIMAL (10,2) NOT NULL, 
forma_pago VARCHAR (255) NOT NULL, 
estado_actual ENUM('recibido','en transito','entregado') NOT NULL, 
instrucciones_especiales VARCHAR (255), 
cuidados_especiales ENUM ('si','no') NOT NULL DEFAULT 'no',
ruta_distribucion INT NOT NULL,
tiempo_estimado_entrega DATETIME NULL,
duracion_estimada_hora INT NULL, 
PRIMARY KEY(envioID), 
CONSTRAINT fk_id_cliente FOREIGN KEY (id_cliente) REFERENCES cliente(clienteID) ON DELETE CASCADE, 
CONSTRAINT fk_fkconductor_id FOREIGN KEY (id_conductor) REFERENCES conductores(conductorID) ON DELETE CASCADE, 
CONSTRAINT fk_fkvehiculo_id FOREIGN KEY (id_vehiculo) REFERENCES vehiculo(vehiculoID) ON DELETE CASCADE );

CREATE TABLE ruta_distribucion( 
ruta_distribucionID INT NOT NULL AUTO_INCREMENT, 
codigo VARCHAR(255) NOT NULL UNIQUE, 
nombre VARCHAR(255) NOT NULL, 
zona_geografica VARCHAR(255) NOT NULL, 
puntos_entrega_habituales VARCHAR(255) NOT NULL, 
distancia_total_km DECIMAL(10,2) NOT NULL, 
tiempo_estimado INT NOT NULL, 
id_vehiculo INT NOT NULL, 
id_conductor INT NOT NULL, 
horario_programado VARCHAR (255) NOT NULL, 
centro_distribucionID INT NOT NULL, 
PRIMARY KEY(ruta_distribucionID), 
CONSTRAINT fk_vehiculoid FOREIGN KEY (id_vehiculo) REFERENCES vehiculo(vehiculoID) ON DELETE CASCADE , 
CONSTRAINT fk_conductorid FOREIGN KEY (id_conductor) REFERENCES conductores(conductorID) ON DELETE CASCADE);

CREATE TABLE seguimiento_envios( 
seguimiento_enviosID INT NOT NULL auto_increment, 
id_envio INT NOT NULL, 
id_centro_distribucion INT NOT NULL, 
id_conductor INT NOT NULL, 
fecha_hora TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP, 
ubicacion VARCHAR (255) NOT NULL, 
actividad_realizada ENUM('recogida','transito','almacen','reparto','entregado') NOT NULL,
empleado_responsable VARCHAR (255) NOT NULL, 
observaciones VARCHAR(255), 
coordenadas_gps VARCHAR(255) NOT NULL, 
PRIMARY KEY (seguimiento_enviosID), 
CONSTRAINT fk_enviosid FOREIGN KEY (id_envio) REFERENCES envios(envioID) ON DELETE CASCADE , 
CONSTRAINT fk_conductor FOREIGN KEY (id_conductor) REFERENCES conductores(conductorID) ON DELETE CASCADE );

CREATE TABLE centros_distribucion( 
centro_distribucionID INT NOT NULL AUTO_INCREMENT, 
codigo VARCHAR (255) NOT NULL UNIQUE, nombre VARCHAR (255) NOT NULL,
direccion VARCHAR(255) NOT NULL, coordenadas VARCHAR(255) NOT NULL, 
tamaño_almacen VARCHAR(255) NOT NULL, capacidad_max DECIMAL(10,2) NOT NULL, 
zonas_que_atiende VARCHAR(255) NOT NULL, personal_asignado VARCHAR(255) NOT NULL, 
horario_operacion VARCHAR (255) NOT NULL, responsable VARCHAR(255) NOT NULL, 
PRIMARY KEY(centro_distribucionID) );

ALTER TABLE seguimiento_envios ADD CONSTRAINT fk_seguimiento_enviosid FOREIGN KEY (id_centro_distribucion) 
REFERENCES centros_distribucion(centro_distribucionID)ON DELETE CASCADE ;

ALTER TABLE ruta_distribucion ADD CONSTRAINT fk_centro_distribucionid FOREIGN KEY 
(centro_distribucionID) REFERENCES centros_distribucion(centro_distribucionID) ON DELETE CASCADE;

CREATE TABLE mantenimiento_vehiculo( 
mantenimiento_vehiculoID INT NOT NULL AUTO_INCREMENT, 
codigo_orden VARCHAR (255) NOT NULL UNIQUE, 
id_vehiculo INT NOT NULL, tipo ENUM('preventivo','correctivo'), 
fecha_programada DATE NOT NULL, 
fecha_ejecucion DATE NOT NULL, 
kilometraje DECIMAL (10,2) NOT NULL, 
repuestos_usados VARCHAR(255) NOT NULL, 
costo DECIMAL (10,2) NOT NULL, 
taller_o_tecnico VARCHAR (255) NOT NULL, 
tiempo_fuera_servicio VARCHAR (255) NOT NULL, 
descripcion_trabajo_realizado VARCHAR(255) NOT NULL, 
PRIMARY KEY (mantenimiento_vehiculoID), 
CONSTRAINT fk_vehiculo FOREIGN KEY (id_vehiculo) REFERENCES vehiculo(vehiculoID) ON DELETE CASCADE );

CREATE TABLE consumo_combustible( 
consumo_combustibleID INT NOT NULL AUTO_INCREMENT, 
vehiculo INT NOT NULL, 
conductor INT NOT NULL, 
fecha DATE NOT NULL, 
cantidad DECIMAL (10,2) NOT NULL, 
tipo_combustible ENUM('corriente','extra','diesel','gas natural') NOT NULL, 
costo DECIMAL (10,2) NOT NULL, 
estacion_servicio VARCHAR (255) NOT NULL, 
km_momento_carga DECIMAL (10,2) NOT NULL, 
PRIMARY KEY(consumo_combustibleID), 
CONSTRAINT fk_vehiculos_id FOREIGN KEY (vehiculo) REFERENCES vehiculo(vehiculoID)ON DELETE CASCADE , 
CONSTRAINT fk_conductoresid FOREIGN KEY (conductor) REFERENCES conductores(conductorID)ON DELETE CASCADE );

CREATE TABLE incidentes_durante_servicio( 
incidentes_servicioID INT NOT NULL AUTO_INCREMENT, 
vehiculo_id INT NOT NULL, conductor_id INT NOT NULL, 
codigo VARCHAR (255) NOT NULL UNIQUE, 
fecha_hora DATETIME NOT NULL, 
tipo_incidente VARCHAR (255) NOT NULL, 
ubicacion VARCHAR(255) NOT NULL, 
descripcion TEXT NOT NULL, 
causas TEXT NOT NULL, 
consecuencias VARCHAR (255) NOT NULL, 
medidas_tomadas VARCHAR (255) NOT NULL, 
estado_resolucion VARCHAR (255) NOT NULL, 
PRIMARY KEY (incidentes_servicioID),
CONSTRAINT fk_vehiculo_id1 FOREIGN KEY (vehiculo_id) REFERENCES vehiculo(vehiculoID)ON DELETE CASCADE , 
CONSTRAINT fk_conductor_id1 FOREIGN KEY (conductor_id) REFERENCES conductores(conductorID)ON DELETE CASCADE );

INSERT INTO vehiculo(
    registro_unico, marca, modelo, anio, capacidad_carga, consumo_promedio_combustible,
    numero_placa, fecha_adquisicion, km_actual, fecha_soat, fecha_tecnomecanica,
    estado_operativo, tipo_vehiculo, transporte_especial
)
VALUES
('C-001','mercedes benz','atego 1726',2018,12,28.2,'FHF388','2023-01-15',320500,'2026-05-10','2026-06-15','activo','camion','si'),
('C-002','chevrolet','nhr euro',2022,3.2,12.5,'FKX358','2022-05-10',150300,'2025-12-01','2026-01-20','activo','camion','no'),
('M-001','yamaha','r15 v4',2021,3.0,2.5,'MPU57E','2022-04-06',54500,'2026-03-01','2026-02-15','activo','motocicleta','no'),
('C-003','volkswagen','worker',2015,18.0,32,'LZR365','2015-07-12',500000,'2024-12-01','2025-01-10','en mantenimiento','camion','si'),
('F-001','renault','kangoo express',2023,0.75,8.7,'SRL856','2022-05-23',80000,'2026-07-01','2026-07-15','activo','furgoneta','si'),
('M-002','akt','cr4 200 pro',2022,0.20,3.0,'AMA40G','2020-06-18',150300,'2025-11-20','2025-12-05','en mantenimiento','motocicleta','no'),
('C-004','mercedes benz','atego 1726',2020,12,28.2,'MDX863','2020-07-15',280420,'2026-04-01','2026-04-10','activo','camion','si'),
('F-002','nissan','nv200',2019,1.2,9.5,'QZM530','2018-03-02',380000,'2025-09-15','2025-10-01','en mantenimiento','furgoneta','si'),
('C-005','mercedes benz','atego 1726',2018,12,28.2,'NQD185','2018-05-12',640000,'2026-02-01','2026-02-20','activo','camion','no'),
('F-003','nissan','nv200',2018,1.2,9.5,'NJD262','2018-05-12',582000,'2026-08-01','2026-08-20','activo','furgoneta','si');




INSERT INTO conductores(codigo_empleado,nombres,apellidos,documento_identidad,fecha_nacimiento,direccion, 
telefono,tipo_licencia, fecha_vencimiento_licencia,anios_experiencia, certificacion_especial, estado_conductor) 
VALUES 
('E-001','Carlos Javier', 'Mesa osorio', '10548562354','1991-01-01','cra 80a # 39 - 2', '3015964785', 'C2', 
'2028-01-05', 5,NULL, 'descanso'), ('E-002', 'Julian' , 'Ocampo Toro', '516584123', '1964-11-28', 
'cll 9 sur # 79 c - 139', '3002807596', 'A2','2030-07-23', 7,NULL, 'disponible'), 
('E-003', 'Simon', 'Ramirez Ruiz', '630158945', '1997-07-23', 'cra 29 B # 39 - 02', '3011895859', 'C2', 
'2030-06-05', 2,NULL, 'en ruta'), ('E-004', 'Juan Camilo', 'Ortega Sanchez', '1037635133', '1994-06-29', 
'cra 82 # 9a sur - 79', '3002807450', 'C2', '2027-05-02', 4,NULL, 'disponible'), 
('E-005', 'Luis Carlos', 'Arboleda Torres', '54962547', '1984-04-12', 'cll 9 a sur # 39 - 72', '3117745652', 'A2', 
'2026-08-20', 8,NULL, 'descanso'), ('E-006', 'Sergio', 'Lopera Quintana', '84632580', '2000-05-02', 
'calle 10 # 45 - 67', '3158562874', 'C2', '2040-02-02', 4,NULL, 'en ruta'), 
('E-007','Carlos','Ramirez','1034567890','1985-06-12','Cra 45 #12-34','3004567890','C2','2027-06-12', 12,
'Transporte de mercancías peligrosas','disponible'), ('E-008','Maria','Gomez','1045678901','1990-03-25',
'cll 10 # 21 - 37','3012345678','A1','2026-04-25', 8,NULL, 'en ruta'), 
('E-009','Juan','Gomez','1056789012','1988-11-05','Calle 10 #45-67','3029876543','A2','2028-11-05', 10,
'Certificación internacional de carga','descanso'), 
('E-010', 'Lucas', 'Castañeda Reginfo', '1035214785', '1988-12-12','cra 80 # 39 - 12', '3058541234', 'C2', 
'2028-05-08', 10, 'Certificación internacional de carga', 'descanso');

INSERT INTO conductor_vehiculo (conductorID, vehiculoID) 
VALUES 
(1, 1), (2, 3), (3, 2), (4, 7), (5, 6), (6, 9), (7, 4), (8, 3), (9, 6), (10,2);

INSERT INTO cliente(codigo_unico,tipo_cliente,nombre_o_razon_social,rut_o_dni,direccion_fiscal,
direccion_recogida, telefono,email, persona_contacto, termino_pago, clasificacion_por_envio) 
VALUES 
('CL-001','individual', 'Ramiro Ponzo', '1056985456', ' cll 10 b # 29 - 82', 'cll 10 # 35 - 57', '3116874238', 
'rponzo@mail.com', 'Ramiro ponzo', 'Contado', 'bajo volumen'), 
('CL-002', 'empresarial', 'Textiles antioquia SA', '521496587-1', 'cra 83 # 79 - 135', 'cll 11 a # 54 - 19', 
'3114489596','textilesant@mail.com', 'Carlos Ramirez', 'Anticipado', 'alto volumen' ), 
('CL-003', 'empresarial', 'Best Choice', '830114072-1', 'cra 43 f # 16 a - 32', 'cll 24 # 95 - 12', '6045957527', 
'medellin@bestchoiceltda.com', 'Luisa Tejada', 'Contado', 'alto volumen' ), 
('CL-004', 'individual', 'Luisa Sanchez', '1037657688', 'cra 29 B # 39 sur -02', 'cra 29 B # 39 sur - 02', 
'3011895859','Lusanchez@mail.com', 'Luisa Sanchez', 'Anticipado','bajo volumen'), 
('CL-005','empresarial', 'Juniper', '9001700375-1', ' cra 100 5 169', 'Carrera 43a, Calle 7 Sur - 170', 
'326874169', 'domicilios@juniper.com', 'Carlos Gomez', 'Credito Empresarial', 'medio volumen'), 
('CL-006','individual','Camila Castrillon','519684567', 'cra 82 # 38 sur 02',' cra 82 # 38 sur 02', '3126895703', 
'ccastrillon@mail.com', 'Camila Castrillon', 'Contado', 'bajo volumen' ), 
('CL-007', 'individual', 'Jesus Ochoa', '1037965874', 'cll 10 # 40 - 35', 'cll 10 # 40 - 35', 
'3002826597', 'ochoaje@mail.com', 'Jesus Ochoa', 'Anticipado', 'bajo volumen' ), 
('CL-008', 'empresarial', 'J&R Distribuidora licores', '901586336-1', 'Cll 79B Sur # 50-150', ' Cra. 39 #8-17', 
'314256978', 'pedidos@jr.com', 'Camilo Gaviria', 'Credito Empresarial', 'alto volumen'), 
('CL-009', 'empresarial' , 'Dislicores', '890916575-4', 'Cra 43A # 25A – 45','Cra 43A # 25A – 45', '3011564231', 
'pedidos@dislicores.com','Juliana Salazar', 'Anticipado' , 'medio volumen' ), 
('CL-010', 'individual', 'Lorena Cortes', '10376398564', 'transv 32 sur # 79 - 81','transv 32 sur # 79 - 81', 
'3136987452', 'lsuarez@mail.com', 'Lorena Cortes', 'contado', 'bajo volumen' );

INSERT INTO centros_distribucion(codigo,nombre,direccion,coordenadas,tamaño_almacen,capacidad_max, 
zonas_que_atiende, personal_asignado, horario_operacion, responsable) 
VALUES 
('CD-001','Centro Medellín Norte','Cra 50 #45-67, Medellín','6.2518,-75.5636','500 m',5000,'Zona Norte Medellín',
20,'08:00-20:00','Laura Gómez'), 
('CD-002','Centro Bogotá Sur','Av. 68 #20-30, Bogotá','4.6097,-74.0817', '800 m',8000,'Zona Sur Bogotá',30, 
'07:00-19:00','Carlos Ramírez'), 
('CD-003','Centro Cali Principal','Calle 13 #100-25, Cali','3.4516,-76.5320','600 m',6000,'Zona Centro Cali',60, 
'09:00-21:00','Marta López'), 
('CD-004', 'Itagui Sur', 'cra 52d # 76 - 67', '6.175-65.380', '500 m', 4000, 'Zona Sur Medellin',20,
'09:00-21:00', 'Consuelo ALvarez'), 
('CD-005', 'Centro Medellin', 'Cra 29 # 80 - 35', '5.2630-65.4587','1000 m', 8000, 'Centro Medellin', 40, 
'08:00-20:00', 'Jairo Grisales'), 
('CD-006', 'Centro Pasto', 'Cll 8 # 30 - 46', '8.3255-68.6534', '500 m', 6000, 'Centro Pasto', 15, 
'09:00-20:00', 'Amparo Cadavid'), 
('CD-007', 'Cetro Armenia', 'Cra 56 # 65 - 70', '5.6576-63.4698', '800 m', 7000, 'Centro Armenia', 20, 
'09:00-20:00', 'Nicolas Quintana'), 
('CD-008', 'Centro Santa Marta','Cll 56 # 85 - 24','6.8526-65.7863','500 m', 6000,'Centro Santa Marta',20,
'08:00-20:00', 'Rocio Ramirez'), 
('CD-009', 'Poblado Medellin', 'cll 10 # 12 -25','10.1250-87.5634','300 m', 300, 'Poblado Medellin', 10, 
'07:00-18:00','Mauricio Morales' ), 
('CD-010', 'Centro Bogota Norte', 'cra 86 # 15 -54', '8.6155-49.5218', '1000 m', 8000, 'Centro Bogota Norte', 
40,'07:00-18:00','Lucia Cardenas');

INSERT INTO ruta_distribucion(codigo,nombre,zona_geografica,puntos_entrega_habituales,distancia_total_km, 
tiempo_estimado,id_vehiculo,id_conductor,horario_programado,centro_distribucionID) 
VALUES 
('RD-001','Ruta Medellín Norte','Medellín Norte','Bello, Copacabana, Girardota',45,90,1,1,'08:00-12:00',1), 
('RD-002','Ruta Bogotá Sur','Bogotá Sur','Kennedy, Bosa, Soacha',60,120,2,2,'07:00-11:00',2), 
('RD-003','Ruta Cali Centro','Cali Centro','San Nicolás, El Prado, Versalles',25,60,5,3,'09:00-13:00',3), 
('RD-004','Ruta Itagüí Sur','Itagüí','Itagüí, La Estrella, Sabaneta',30,75,4,4,'10:00-14:00',4), 
('RD-005','Ruta Medellín Centro','Medellín Centro','La Candelaria, Boston, Prado',20,50,3,5,'08:00-12:00',5), 
('RD-006','Ruta Pasto','Pasto','Centro, San Juan, Catambuco',40,100,6,6,'09:00-13:00',6), 
('RD-007','Ruta Armenia','Armenia','Centro, La Tebaida, Montenegro',35,85,8,7,'07:00-11:00',7), 
('RD-008','Ruta Santa Marta','Santa Marta','Centro, Taganga, Gaira',50,110,9,8,'08:00-12:00',8), 
('RD-009','Ruta Poblado Medellín','Medellín Poblado','El Poblado, Envigado, Laureles',28,70,10,9,'10:00-14:00',9), 
('RD-010','Ruta Bogotá Norte','Bogotá Norte','Usaquén, Suba, Chía',55,115,2,10,'07:00-11:00',10);

INSERT INTO envios(
    numero_guia,fecha_hora_recepcion,id_cliente,id_conductor,id_vehiculo,remitente,destinatario,
    direccion_origen,direccion_destino,tipo_servicio,descripcion_contenido,peso,dimensiones,
    valor_declarado,costo,forma_pago,estado_actual,instrucciones_especiales,cuidados_especiales,
    ruta_distribucion,tiempo_estimado_entrega,duracion_estimada_hora
)
VALUES
('G-001','2026-01-10 08:30:00',1,1,1,'Carlos Ramírez','Ana Gómez','Calle 10 #20-30, Medellín',
'Calle 50 #40-25, Bogotá','express','Documentos legales',2.50,'30x20x10 cm',500000,25000,'tarjeta',
'en transito','Entregar en portería','si',1,'2026-01-10 10:30:00',24),
('G-002','2026-01-11 09:15:00',2,2,3,'Textiles Antioquia','Juan Pérez','Carrera 15 #45-60, Cali',
'Calle 80 #12-34, Medellín','normal','Paquete de ropa',5.20,'60x40x30 cm',800000,40000,'efectivo','recibido',
NULL,'no',2,'2026-01-11 12:15:00',72),
('G-003','2026-02-12 14:00:00',3,3,2,'María López','Pedro Torres','Av. Siempre Viva 123, Bucaramanga',
'Calle 100 #20-10, Bogotá','mismo dia','Electrodoméstico pequeño',12.00,'80x60x50 cm',1500000,70000,
'transferencia','en transito','Frágil, manejar con cuidado','si',1,'2026-02-12 15:00:00',6),
('G-004','2026-03-13 10:45:00',1,4,7,'Carlos Ramírez','Empresa ABC','Calle 25 #30-40, Medellín',
'Calle 70 #15-20, Cartagena','express','Papelería corporativa',3.00,'40x30x20 cm',300000,20000,
'tarjeta','en transito',NULL,'si',3,'2026-03-13 12:45:00',24),
('G-005','2026-03-14 16:20:00',2,5,6,'Textiles Antioquia','Luis Fernández','Carrera 50 #25-60, Cali',
'Calle 90 #30-40, Bogotá','normal','Cajas de repuestos',20.00,'100x80x60 cm',2000000,120000,'efectivo',
'en transito','Requiere montacargas','si',2,'2026-03-14 20:20:00',96),
('G-006','2026-03-15 11:00:00',3,6,9,'María López','Ana Gómez','Av. Libertad 456, Bucaramanga',
'Calle 40 #25-30, Medellín','mismo dia','Flores y regalos',1.50,'25x25x25 cm',200000,15000,'transferencia',
'entregado','Entregar personalmente','si',1,NULL,5),
('G-007','2024-02-16 13:30:00',4,7,4,'Luisa Sánchez','Pedro Torres','Calle 12 #34-56, Bogotá',
'Calle 22 #45-67, Cali','express','Equipo de oficina',8.00,'70x50x40 cm',1200000,60000,'tarjeta',
'en transito',NULL,'si',3,'2024-02-16 15:30:00',24),
('G-008','2026-02-28 09:50:00',5,8,3,'Juan Pérez','Restaurante El Charro','Carrera 60 #70-80, Medellín',
'Calle 33 #44-55, Bucaramanga','normal','Libros y material educativo',10.00,'90x60x40 cm',500000,30000,
'efectivo','recibido','Caja frágil','si',2,'2026-02-28 12:50:00',72),
('G-009','2024-02-18 15:10:00',2,9,6,'Textiles Antioquia','Carlos Ramírez','Calle 77 #88-99, Cali',
'Calle 11 #22-33, Bogotá','express','Medicamentos',4.00,'50x40x30 cm',1000000,50000,'transferencia',
'en transito','Entrega urgente','si',3,'2024-02-18 17:10:00',24),
('G-010','2026-03-19 17:25:00',1,10,2,'Carlos Ramírez','María López','Carrera 20 #30-40, Medellín',
'Calle 55 #66-77, Cartagena','mismo dia','Paquete de alimentos',15.00,'100x70x50 cm',700000,35000,'efectivo',
'en transito','Mantener refrigerado','si',1,'2026-03-19 18:25:00',8);



INSERT INTO seguimiento_envios(id_envio,id_centro_distribucion,id_conductor,fecha_hora,ubicacion,
actividad_realizada, empleado_responsable,observaciones,coordenadas_gps) 
VALUES 
(1,1,1,'2026-01-10 09:00:00','Centro Medellín Norte','recogida','Laura Gómez','Paquete recibido en bodega', 
'6.2518,-75.5636'), (2,2,2,'2026-01-11 10:00:00','Centro Bogotá Sur','transito','Carlos Ramírez',
'Salida hacia destino', '4.6097,-74.0817'), 
(3,3,3,'2026-02-12 15:00:00','Centro Cali Principal','almacen','Marta López','Guardado temporal en bodega', 
'3.4516,-76.5320'), 
(4,4,4,'2026-03-13 11:30:00','Itagüí Sur','reparto','Consuelo Álvarez','Ruta iniciada','6.175,-65.380'), 
(5,5,5,'2026-03-14 17:00:00','Centro Medellín','transito','Jairo Grisales','En camino a destino', 
'5.2630,-65.4587'), 
(6,6,6,'2026-03-15 12:00:00','Centro Pasto','almacen','Amparo Cadavid','Esperando despacho', '8.3255,-68.6534'), 
(7,7,7,'2024-02-16 14:00:00','Centro Armenia','reparto','Nicolás Quintana','Entrega programada', 
'5.6576,-63.4698'), 
(8,8,8,'2026-02-28 10:30:00','Centro Santa Marta','transito','Rocío Ramírez','En ruta hacia cliente', 
'6.8526,-65.7863'), 
(9,9,9,'2024-02-18 16:00:00','Poblado Medellín','reparto','Mauricio Morales','Entrega en curso', 
'10.1250,-87.5634'), 
(10,10,10,'2026-03-19 18:00:00','Centro Bogotá Norte','entregado','Lucía Cárdenas','Paquete entregado al cliente', 
'8.6155,-49.5218');

INSERT INTO mantenimiento_vehiculo(codigo_orden,id_vehiculo,tipo,fecha_programada,fecha_ejecucion,kilometraje, 
repuestos_usados,costo, taller_o_tecnico,tiempo_fuera_servicio,descripcion_trabajo_realizado) 
VALUES 
('MV-001',1,'preventivo','2023-06-01','2025-06-02',320500,'Filtro de aceite',250000,'Taller Norte',2, 
'Cambio de aceite y filtros'), 
('MV-002',2,'preventivo','2023-07-10','2025-07-11',150300,'Pastillas de freno',400000,'Taller Sur',1, 
'Revisión de frenos'), 
('MV-003',3,'preventivo','2023-08-05','2025-08-05',54500,'Bujías',150000,'Moto Taller',1, 
'Cambio de bujías y ajuste general'), 
('MV-004',4,'correctivo','2023-09-15','2025-09-18',500000,'Caja de cambios',2500000,'Taller Central', 5,
'Reparación de transmisión'), 
('MV-005',5,'preventivo','2023-10-20','2025-10-21',80000,'Filtro de aire',180000,'Taller Express', 1,
'Mantenimiento básico y limpieza'), 
('MV-006',6,'correctivo','2023-11-02','2025-11-04',150300,'Carburador',600000,'Moto Taller',2 ,
'Reparación de carburador'), 
('MV-007',7,'preventivo','2023-12-12','2026-01-12',280420,'Aceite motor',220000,'Taller Norte',1, 
'Cambio de aceite y revisión general'), 
('MV-008',8,'correctivo','2024-01-08','2026-01-10',380000,'Suspensión trasera',800000,'Taller Sur',3, 
'Reparación de suspensión'), 
('MV-009',9,'preventivo','2024-02-14','2025-02-15',640000,'Filtro combustible',300000,'Taller Central',2, 
'Cambio de filtros y revisión'), 
('MV-010',10,'preventivo','2024-03-05','2026-03-06',582000,'Sistema eléctrico',1200000,'Taller Express',1, 
'Revisión eléctrica general');

INSERT INTO consumo_combustible
(vehiculo,conductor,fecha,cantidad,tipo_combustible,costo,estacion_servicio, km_momento_carga) 
VALUES 
(1,1,'2026-01-10',50,'corriente',350000,'Estación Norte',319800), 
(2,2,'2026-01-15',30,'extra',210000,'Estación Sur',150000), 
(3,3,'2026-01-20',12,'corriente',72000,'Estación Centro',54000), 
(4,4,'2026-02-05',60,'gas natural',420000,'Estación Central',499000), 
(5,5,'2026-02-12',25,'corriente',175000,'Estación Express',79000), 
(6,6,'2026-02-18',10,'extra',60000,'Estación Moto',149800), 
(7,7,'2026-02-25',55,'corriente',385000,'Estación Norte',279500), 
(8,8,'2026-03-05',28,'gas natural',196000,'Estación Sur',379000), 
(9,9,'2026-03-12',65,'extra',455000,'Estación Central',639000), 
(10,10,'2026-03-20',30,'corriente',210000,'Estación Express',581000);

INSERT INTO incidentes_durante_servicio(vehiculo_id,conductor_id,codigo,fecha_hora,tipo_incidente,ubicacion, 
descripcion,causas,consecuencias, medidas_tomadas,estado_resolucion) 
VALUES 
(1,1,'IS-001','2026-01-12 09:30:00','averia','Medellin - Autopista Norte', 'Falla en frenos',
'Desgaste de pastillas','Retraso en entregas','Reemplazo de frenos','resuelto'), 
(1,1,'IS-002','2026-02-20 15:00:00','multa','Medellin - Centro', 'Multa por estacionamiento indebido',
'Desconocimiento de norma','Sanción económica','Capacitación al conductor', 'resuelto'), 
(2,2,'IS-003','2026-01-18 14:20:00','accidente','Bogota - Calle 80', 'Colision con otro vehículo',
'Exceso de velocidad','Daños menores','Reporte a aseguradora','en proceso'), 
(4,4,'IS-004','2026-02-02 11:10:00','avería','Itagüi', 'Problema en transmisión','Falta de mantenimiento', 
'Vehiculo detenido','Traslado a taller','pendiente'), 
(4,4,'IS-005','2026-03-05 09:00:00','retraso','Envigado', 'Entrega tardia','Congestion vehicular', 
'Cliente inconforme','Reprogramacion de entrega','resuelto'), 
(6,6,'IS-006','2026-02-15 13:30:00','accidente','Bucaramanga', 'Choque con motocicleta',
'Distraccion del conductor','Lesiones leves','Atencion medica y reporte','en proceso'), 
(7,7,'IS-007','2026-02-22 10:20:00','avería','Armenia', 'Falla electrica','Corto circuito',
'Vehiculo fuera de servicio','Cambio de cableado','pendiente'), 
(7,7,'IS-008','2026-03-10 16:00:00','multa','Armenia - Centro', 'Multa por exceso de velocidad',
'Conductor apurado','Sancion económica','Capacitacion adicional','resuelto'), 
(9,9,'IS-009','2026-03-08 15:40:00','accidente','Medellín - Poblado', 'Colision con poste',
'Condiciones climaticas','Daños en carrocería','Reparacion en taller','en proceso'), 
(9,9,'IS-010','2026-03-08 15:40:00','accidente','Medellín - Poblado', 'Colision con poste',
'Condiciones climaticas','Daños en carroceria','Reparacion en taller','en proceso');

CREATE TABLE AuditoriaTriggers (
    AuditoriaID INT PRIMARY KEY AUTO_INCREMENT,
    tabla_afectada VARCHAR (50),
    registroID INT,
    Accion VARCHAR(20),
    ValorAnterior VARCHAR(255),
    ValorNuevo VARCHAR(255),
    Usuario VARCHAR(50),
    FechaHora DATETIME
);


-- Triggers:

-- 1. TR_ActualizarEstadoEnvio: Actualiza el estado de un envío tras registrar un seguimiento.

DELIMITER $$

CREATE TRIGGER TR_ActualizarEstadoEnvio
AFTER INSERT ON seguimiento_envios
FOR EACH ROW -- garantiza que se actualice de forma individual el envio por cada seguimiento afectado
BEGIN
    DECLARE estado_prev VARCHAR(255);

    SELECT estado_actual INTO estado_prev
    FROM envios
    WHERE envioID = NEW.id_envio;

-- actualizar estado de envio
    UPDATE envios
    SET estado_actual = NEW.actividad_realizada
    WHERE envioID = NEW.id_envio;

-- Registrar auditoria 
    INSERT INTO AuditoriaTriggers (tabla_afectada, RegistroID, Accion, ValorAnterior, ValorNuevo, Usuario, 
	 FechaHora)
    VALUES ('envios',
            NEW.id_envio,
            'UPDATE',
            estado_prev,
            NEW.actividad_realizada,
            USER(),
            NOW());
END$$

DELIMITER ;


-- hacemos insert 
INSERT INTO seguimiento_envios (
    id_envio, id_centro_distribucion, id_conductor, fecha_hora, ubicacion, actividad_realizada, 
	 empleado_responsable, observaciones, coordenadas_gps
) VALUES (
    1, 3, 2, NOW(), 'Medellín - Centro', 'entregado', 'Juan Pérez', 'Paquete recibido sin novedades', 
	 '6.2500,-75.5700'
);


-- revisamos la auditoria
SELECT * FROM AuditoriaTriggers WHERE RegistroID = 1 ORDER BY FechaHora DESC;

-- 2. TR_VerificarLicenciaConductor: Verifica que la licencia del conductor asignado sea válida para el vehículo.

DELIMITER $$

CREATE TRIGGER TR_VerificarLicenciaConductor 
BEFORE INSERT ON conductor_vehiculo 
FOR EACH ROW 
BEGIN
    DECLARE lic VARCHAR(10);
    DECLARE tipo_v VARCHAR(20);

   
    SELECT tipo_licencia INTO lic FROM conductores WHERE conductorID = NEW.conductorID;
    SELECT tipo_vehiculo INTO tipo_v FROM vehiculo WHERE vehiculoID = NEW.vehiculoID;

-- Validacion de licencia por tipo
    IF (tipo_v = 'motocicleta' AND lic NOT IN ('A1','A2')) OR 
       (tipo_v IN ('camion', 'furgoneta') AND lic <> 'C2') THEN
        
        SIGNAL SQLSTATE '45000' 
        SET MESSAGE_TEXT = 'Licencia incompatible';
    ELSE
        -- Log simple
        INSERT INTO AuditoriaTriggers (tabla_afectada, registroID, Accion, ValorNuevo, Usuario)
        VALUES ('conductor_vehiculo', NEW.conductorID, 'INSERT', tipo_v, USER());
    END IF;
END $$

DELIMITER ;



-- insert valido
INSERT INTO conductor_vehiculo (conductorID, vehiculoID)
VALUES (2, 3);  -- Julian (A2) con motocicleta Yamaha R15

-- insert invalido
INSERT INTO conductor_vehiculo (conductorID, vehiculoID)
VALUES (8, 1);


SELECT * FROM AuditoriaTriggers ORDER BY FechaHora DESC;


-- 3. TR_ActualizarKilometrajeVehiculo: Actualiza el kilometraje del vehículo tras completar una ruta.

DELIMITER $$

CREATE TRIGGER TR_ActualizarKilometrajeVehiculo 
AFTER UPDATE ON seguimiento_envios 
FOR EACH ROW 
BEGIN
    DECLARE vehiculo_id INT;
    DECLARE v_km_recorrido DECIMAL(10,2);
    DECLARE v_km_anterior DECIMAL(10,2);

-- Detectar cambio de estado a entregado
    IF NEW.actividad_realizada = 'entregado' AND OLD.actividad_realizada != 'entregado' THEN
        
-- Obtener vehículo y distancia 
        SELECT rd.id_vehiculo, IFNULL(SUM(rd.distancia_total_km), 0) 
        INTO vehiculo_id, v_km_recorrido
        FROM envios e 
        INNER JOIN ruta_distribucion rd ON e.ruta_distribucion = rd.ruta_distribucionID 
        WHERE e.envioID = NEW.id_envio;

-- Sacar el km actual antes de sumar
        SELECT km_actual INTO v_km_anterior FROM vehiculo WHERE vehiculoID = vehiculo_id;

-- Actualización directa
        UPDATE vehiculo 
        SET km_actual = km_actual + v_km_recorrido 
        WHERE vehiculoID = vehiculo_id;

        INSERT INTO AuditoriaTriggers (tabla_afectada, registroID, Accion, ValorAnterior, ValorNuevo, Usuario)
        VALUES ('vehiculo', vehiculo_id, 'KM_UPDATE', v_km_anterior, v_km_anterior + v_km_recorrido, USER());

    END IF;
END $$

DELIMITER ;

-- realizar el update de actividad para verificar funcionamiento del trigger
UPDATE seguimiento_envios SET actividad_realizada = 'entregado' WHERE id_envio = 1;


-- Ver el kilometraje actualizado del vehículo
SELECT vehiculoID, km_actual FROM vehiculo WHERE vehiculoID = 1;

-- Ver el registro en auditoría
SELECT * FROM AuditoriaTriggers WHERE tabla_afectada = 'vehiculo' ORDER BY FechaHora DESC;

-- 4. TR_CalcularCostosRuta: Calcula automáticamente los costos asociados a una ruta.

DELIMITER $$

CREATE TRIGGER TR_CalcularCostosRuta 
AFTER UPDATE ON seguimiento_envios 
FOR EACH ROW 
BEGIN
    DECLARE v_costo_envio DECIMAL(12,2);
    DECLARE v_costo_combustible DECIMAL(12,2);
    DECLARE v_km_ruta DECIMAL(10,2);
    DECLARE v_total DECIMAL(12,2);

    SELECT e.costo, IFNULL(SUM(r.distancia_total_km), 0)
    INTO v_costo_envio, v_km_ruta
    FROM envios e
    LEFT JOIN ruta_distribucion rd ON e.ruta_distribucion = rd.ruta_distribucionID
    WHERE e.envioID = NEW.id_envio;

    -- Costo de combustible por vehículo
    SELECT IFNULL(SUM(cc.costo), 0) 
    INTO v_costo_combustible
    FROM consumo_combustible cc
    INNER JOIN envios e ON cc.vehiculo = e.id_vehiculo
    WHERE e.envioID = NEW.id_envio;

    SET v_total = v_costo_envio + v_costo_combustible;

    INSERT INTO AuditoriaTriggers (tabla_afectada, registroID, Accion, ValorNuevo, Usuario)
    VALUES ('envios', NEW.id_envio, 'CALCULO_COSTOS', v_total, USER());
END $$

DELIMITER ;

-- lanzamos update de envio por medio del id
UPDATE seguimiento_envios SET id_envio = 4 WHERE id_envio = 4;

-- verificamos el costo del envio por medio del id 
SELECT envioID, costo FROM envios WHERE envioID = 4;

-- verificamos si aparece el trigger en la auditoriaTriggers
SELECT * FROM AuditoriaTriggers WHERE tabla_afectada = 'envios' ORDER BY FechaHora DESC;


-- 5. TR_GenerarAlertaMantenimiento: Genera alertas cuando un vehículo requiere mantenimiento.

DELIMITER $$

CREATE TRIGGER TR_GenerarAlertaMantenimiento 
AFTER UPDATE ON vehiculo 
FOR EACH ROW 
BEGIN 
    DECLARE v_ultimo_aceite DECIMAL(10,2); 
    DECLARE v_pendientes INT; 
    DECLARE v_alerta VARCHAR(50); 

    SELECT IFNULL(MAX(kilometraje), 0) INTO v_ultimo_aceite 
    FROM mantenimiento_vehiculo 
    WHERE id_vehiculo = NEW.vehiculoID AND tipo = 'cambio_aceite'; 

    SELECT COUNT(*) INTO v_pendientes 
    FROM incidentes_durante_servicio 
    WHERE vehiculo_id = NEW.vehiculoID AND estado_resolucion = 'pendiente'; 

-- alertas 
    IF NEW.km_actual >= 20000 THEN 
        SET v_alerta = 'Mantenimiento general'; 
    ELSEIF (NEW.km_actual - v_ultimo_aceite) >= 5000 THEN 
        SET v_alerta = 'Cambio de aceite'; 
    ELSEIF v_pendientes > 0 THEN 
        SET v_alerta = 'Mantenimiento por incidentes'; 
    END IF; 

    IF v_alerta IS NOT NULL THEN 
        INSERT INTO AuditoriaTriggers (tabla_afectada, registroID, Accion, ValorNuevo, Usuario) 
        VALUES ('vehiculo', NEW.vehiculoID, 'ALERTA', v_alerta, USER()); 
    END IF; 
END $$ 

DELIMITER ;
-- update simulando que el vehiculo ha recorrido mas distancia para generar la alerta
UPDATE vehiculo SET km_actual = km_actual + 500 WHERE vehiculoID = 1;


-- verificamos que aparezca en auditoria 
SELECT * FROM AuditoriaTriggers WHERE tabla_afectada = 'vehiculo' ORDER BY FechaHora DESC;

-- Funciones:

-- 1. FN_CalcularDistanciaRuta: Calcula la distancia total de una ruta específica.

DELIMITER $$

CREATE FUNCTION FN_CalcularDistanciaRuta(RutaID INT) 
RETURNS DECIMAL(10,2) 
DETERMINISTIC 
BEGIN 
    DECLARE v_distancia DECIMAL(10,2);

    SELECT IFNULL(SUM(distancia_total_km), 0) INTO v_distancia 
    FROM ruta_distribucion 
    WHERE ruta_distribucionID = RutaID;

    RETURN v_distancia;
END $$

DELIMITER ;

-- probamos si la funcion se ejecuta correctamente

SELECT FN_CalcularDistanciaRuta(4) AS distancia_total;

-- 2. FN_ObtenerVehiculoOptimo: Sugiere el vehículo óptimo según características del envío.

DELIMITER $$

CREATE FUNCTION FN_ObtenerVehiculoOptimo(p_envioID INT) 
RETURNS INT 
DETERMINISTIC 
BEGIN 
    DECLARE v_id_optimo INT; 
    DECLARE v_cuidados ENUM('si','no'); 
    DECLARE v_peso DECIMAL(10,2); 

    SELECT cuidados_especiales, peso 
    INTO v_cuidados, v_peso 
    FROM envios 
    WHERE envioID = p_envioID; 

    SELECT vehiculoID INTO v_id_optimo 
    FROM vehiculo 
    WHERE estado_operativo = 'activo' 
      AND capacidad_carga >= v_peso 
      AND (v_cuidados = 'no' OR transporte_especial = 'si') 
    ORDER BY km_actual ASC 
    LIMIT 1; 

    RETURN v_id_optimo; 
END $$

DELIMITER ;


-- ejecutamos la funcion 
SELECT FN_ObtenerVehiculoOptimo(1) AS vehiculo_sugerido;



-- 3. FN_EstimarTiempoEntrega: Estima el tiempo de entrega considerando ruta, tráfico y paradas.

DELIMITER $$

CREATE FUNCTION FN_EstimarTiempoEntrega(p_envioID INT) 
RETURNS DATETIME 
DETERMINISTIC 
BEGIN 
    DECLARE v_inicio TIME; 
    DECLARE v_min_extra INT; 
    DECLARE v_fecha DATE; 
    DECLARE v_rango VARCHAR(50); 

    SELECT DATE(e.fecha_hora_recepcion), rd.tiempo_estimado, rd.horario_programado 
    INTO v_fecha, v_min_extra, v_rango 
    FROM envios e 
    JOIN ruta_distribucion rd ON e.ruta_distribucion = rd.ruta_distribucionID 
    WHERE e.envioID = p_envioID; 

    SET v_inicio = CAST(SUBSTRING_INDEX(v_rango, '-', 1) AS TIME); 

    RETURN TIMESTAMP(v_fecha, v_inicio) + INTERVAL v_min_extra MINUTE; 
END $$

DELIMITER ;


-- ejecutamos la funcion 
SELECT FN_EstimarTiempoEntrega(1) AS tiempo_estimado_entrega;


-- 4. FN_CalcularCostoEnvio: Calcula el costo total de un envío según distancia, peso y urgencia.

DELIMITER $$

CREATE FUNCTION FN_CalcularCostoEnvio(p_envioID INT) 
RETURNS DECIMAL(10,2) 
DETERMINISTIC 
BEGIN 
    DECLARE v_distancia, v_peso DECIMAL(10,2); 
    DECLARE v_tipo ENUM('normal','express','mismo_dia'); 
    DECLARE v_total DECIMAL(10,2); 

    SELECT r.distancia_total_km, e.peso, e.tipo_servicio 
    INTO v_distancia, v_peso, v_tipo 
    FROM envios e 
    JOIN ruta_distribucion r ON e.ruta_distribucion = r.ruta_distribucionID 
    WHERE e.envioID = p_envioID; 

    SET v_total = (v_distancia * 1000) + (v_peso * 500); 

-- Multiplicadores por servicio
    IF v_tipo = 'express' THEN 
        SET v_total = v_total * 1.5; 
    ELSEIF v_tipo = 'mismo_dia' THEN 
        SET v_total = v_total * 2; 
    END IF; 

    RETURN v_total; 
END $$

DELIMITER ;


-- ejecutar la funcion
SELECT FN_CalcularCostoEnvio(2) AS costo_total_envio;

-- 5. FN_VerificarCapacidadVehiculo: Verifica si un vehículo tiene capacidad para un envío específico.

DELIMITER $$

CREATE FUNCTION FN_VerificarCapacidadVehiculo(id_e INT, id_v INT) 
RETURNS VARCHAR(2)
DETERMINISTIC
BEGIN
    DECLARE v_peso DECIMAL(10,2);
    DECLARE v_capacidad DECIMAL(10,2);

-- Solo traemos los valores directos
    SELECT peso INTO v_peso FROM envios WHERE envioID = id_e;
    SELECT capacidad_carga INTO v_capacidad FROM vehiculo WHERE vehiculoID = id_v;

    IF v_capacidad >= v_peso THEN 
        RETURN 'SI';
    END IF;

    RETURN 'NO';
END $$

DELIMITER ;

-- Verificar la funcion: determinamos si el vehículo 3 puede transportar el envío 5
SELECT FN_VerificarCapacidadVehiculo(5, 3) AS capacidad_valida;


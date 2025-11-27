# UIII-Act-7-models.py

<img width="1243" height="923" alt="dentista_1" src="https://github.com/user-attachments/assets/feaf846f-2514-45be-b9c9-e5476d34f29d" />

```
Paciente_Dental {
	id_paciente integer pk increments unique
	nombre varchar(100)
	apellido varchar(100)
	fecha_nacimiento date
	genero char(1)
	direccion varchar(255)
	telefono varchar(20)
	email varchar(100)
	num_seguro_dental varchar(50)
	fecha_registro date
	historial_medico_previo text
}

Odontologo {
	id_odontologo integer pk increments unique
	nombre varchar(100)
	apellido varchar(100)
	especialidad varchar(100)
	telefono varchar(20)
	email varchar(100)
	licencia_dental varchar(50)
	fecha_contratacion date
	salario decimal(10,2)
	turno varchar(50)
}

Cita_Dental {
	id_cita integer pk increments unique
	id_paciente integer *> Paciente_Dental.id_paciente
	id_odontologo integer *> Odontologo.id_odontologo
	fecha_cita date
	hora_cita time
	motivo_cita text
	estado_cita varchar(50)
	comentarios text
	tipo_tratamiento varchar(100)
}

Tratamiento_Dental {
	id_tratamiento integer pk increments unique
	nombre_tratamiento varchar(100)
	descripcion text
	costo_promedio decimal(10,2)
	duracion_estimada_minutos integer
	es_invasivo boolean
	requiere_anestesia boolean
	materiales_comunes text
}

Historial_Tratamiento {
	id_historial integer pk increments unique
	id_cita integer *> Cita_Dental.id_cita
	id_paciente integer *> Paciente_Dental.id_paciente
	id_odontologo integer *> Odontologo.id_odontologo
	id_tratamiento integer *> Tratamiento_Dental.id_tratamiento
	fecha_realizacion datetime
	notas_tratamiento text
	costo_final decimal(10,2)
	piezas_dentales_afectadas text
}

Factura_Dental {
	id_factura integer pk increments unique
	id_paciente integer *> Paciente_Dental.id_paciente
	fecha_emision date
	total_factura decimal(10,2)
	estado_pago varchar(50)
	metodo_pago varchar(50)
	fecha_vencimiento date
	id_cita_asociada integer *> Cita_Dental.id_cita
	descuento_aplicado decimal(5,2)
}

Proveedor_Dental {
	id_proveedor integer pk increments unique
	nombre_empresa varchar(150)
	nombre_contacto varchar(100)
	telefono varchar(20)
	email varchar(100)
	direccion varchar(255)
	rfc varchar(20)
	tipo_productos varchar(100)
	activo boolean
}

Material_Odontologico {
	id_material integer pk increments unique
	nombre_material varchar(255)
	descripcion text
	stock_actual integer
	fecha_caducidad date
	id_proveedor integer *> Proveedor_Dental.id_proveedor
	id_odontologo integer *> Odontologo.id_odontologo
	costo_unitario decimal(10,2)
	tipo_material varchar(50)
	ubicacion_almacen varchar(100)
}

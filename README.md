# UIII-Act-7-models.py

<img width="1243" height="923" alt="dentista_1" src="https://github.com/user-attachments/assets/feaf846f-2514-45be-b9c9-e5476d34f29d" />

## models.py

```
from django.db import models
from django.core.validators import RegexValidator
from django.utils.translation import gettext_lazy as _

# --- 1. Modelo de Paciente ---

class PacienteDental(models.Model):
    """
    Modelo para almacenar información de los pacientes del consultorio dental.
    """
    GENERO_CHOICES = [
        ('M', 'Masculino'),
        ('F', 'Femenino'),
        ('O', 'Otro'),
    ]
    
    nombre = models.CharField(max_length=100, help_text="Nombre del paciente")
    apellido = models.CharField(max_length=100, help_text="Apellido del paciente")
    fecha_nacimiento = models.DateField(help_text="Fecha de nacimiento")
    genero = models.CharField(max_length=1, choices=GENERO_CHOICES)
    direccion = models.CharField(max_length=255, help_text="Dirección completa")
    telefono = models.CharField(max_length=20, help_text="Teléfono de contacto")
    email = models.EmailField(max_length=100, help_text="Correo electrónico")
    num_seguro_dental = models.CharField(max_length=50, blank=True, null=True, 
                                         help_text="Número de seguro dental")
    fecha_registro = models.DateField(auto_now_add=True, help_text="Fecha de registro en el sistema")
    historial_medico_previo = models.TextField(blank=True, null=True, 
                                               help_text="Historial médico relevante")
    
    class Meta:
        verbose_name = "Paciente Dental"
        verbose_name_plural = "Pacientes Dentales"
        ordering = ['apellido', 'nombre']
    
    def __str__(self):
        return f"{self.nombre} {self.apellido}"


# --- 2. Modelo de Odontólogo ---

class Odontologo(models.Model):
    """
    Modelo para almacenar información de los odontólogos del consultorio.
    """
    nombre = models.CharField(max_length=100, help_text="Nombre del odontólogo")
    apellido = models.CharField(max_length=100, help_text="Apellido del odontólogo")
    especialidad = models.CharField(max_length=100, help_text="Especialidad dental")
    telefono = models.CharField(max_length=20, help_text="Teléfono de contacto")
    email = models.EmailField(max_length=100, help_text="Correo electrónico")
    licencia_dental = models.CharField(max_length=50, unique=True, 
                                       help_text="Número de licencia profesional")
    fecha_contratacion = models.DateField(help_text="Fecha de contratación")
    salario = models.DecimalField(max_digits=10, decimal_places=2, 
                                  help_text="Salario mensual")
    turno = models.CharField(max_length=50, help_text="Turno de trabajo (matutino, vespertino, etc.)")
    
    class Meta:
        verbose_name = "Odontólogo"
        verbose_name_plural = "Odontólogos"
        ordering = ['apellido', 'nombre']
    
    def __str__(self):
        return f"Dr. {self.nombre} {self.apellido} - {self.especialidad}"


# --- 3. Modelo de Tratamiento Dental ---

class TratamientoDental(models.Model):
    """
    Catálogo de tratamientos dentales disponibles.
    """
    nombre_tratamiento = models.CharField(max_length=100, unique=True, 
                                         help_text="Nombre del tratamiento")
    descripcion = models.TextField(help_text="Descripción detallada del tratamiento")
    costo_promedio = models.DecimalField(max_digits=10, decimal_places=2, 
                                        help_text="Costo promedio del tratamiento")
    duracion_estimada_minutos = models.PositiveIntegerField(
        help_text="Duración estimada en minutos"
    )
    es_invasivo = models.BooleanField(default=False, help_text="¿Es un procedimiento invasivo?")
    requiere_anestesia = models.BooleanField(default=False, help_text="¿Requiere anestesia?")
    materiales_comunes = models.TextField(blank=True, null=True, 
                                         help_text="Materiales comúnmente utilizados")
    
    class Meta:
        verbose_name = "Tratamiento Dental"
        verbose_name_plural = "Tratamientos Dentales"
        ordering = ['nombre_tratamiento']
    
    def __str__(self):
        return self.nombre_tratamiento


# --- 4. Modelo de Cita Dental ---

class CitaDental(models.Model):
    """
    Modelo para gestionar las citas de los pacientes.
    """
    ESTADO_CHOICES = [
        ('PROGRAMADA', 'Programada'),
        ('CONFIRMADA', 'Confirmada'),
        ('COMPLETADA', 'Completada'),
        ('CANCELADA', 'Cancelada'),
        ('NO_ASISTIO', 'No asistió'),
    ]
    
    paciente = models.ForeignKey(PacienteDental, on_delete=models.CASCADE, 
                                 related_name='citas')
    odontologo = models.ForeignKey(Odontologo, on_delete=models.CASCADE, 
                                   related_name='citas')
    fecha_cita = models.DateField(help_text="Fecha de la cita")
    hora_cita = models.TimeField(help_text="Hora de la cita")
    motivo_cita = models.TextField(help_text="Motivo de la consulta")
    estado_cita = models.CharField(max_length=50, choices=ESTADO_CHOICES, 
                                   default='PROGRAMADA')
    comentarios = models.TextField(blank=True, null=True, 
                                  help_text="Comentarios adicionales")
    tipo_tratamiento = models.CharField(max_length=100, blank=True, null=True,
                                       help_text="Tipo de tratamiento a realizar")
    
    class Meta:
        verbose_name = "Cita Dental"
        verbose_name_plural = "Citas Dentales"
        ordering = ['-fecha_cita', '-hora_cita']
        unique_together = ['odontologo', 'fecha_cita', 'hora_cita']
    
    def __str__(self):
        return f"Cita: {self.paciente} - {self.fecha_cita} {self.hora_cita}"


# --- 5. Modelo de Historial de Tratamiento ---

class HistorialTratamiento(models.Model):
    """
    Registro histórico de tratamientos realizados.
    """
    cita = models.ForeignKey(CitaDental, on_delete=models.CASCADE, 
                            related_name='historiales')
    paciente = models.ForeignKey(PacienteDental, on_delete=models.CASCADE, 
                                related_name='historiales')
    odontologo = models.ForeignKey(Odontologo, on_delete=models.CASCADE, 
                                  related_name='historiales')
    tratamiento = models.ForeignKey(TratamientoDental, on_delete=models.CASCADE, 
                                   related_name='historiales')
    fecha_realizacion = models.DateTimeField(auto_now_add=True, 
                                            help_text="Fecha y hora de realización")
    notas_tratamiento = models.TextField(help_text="Notas del procedimiento realizado")
    costo_final = models.DecimalField(max_digits=10, decimal_places=2, 
                                     help_text="Costo final del tratamiento")
    piezas_dentales_afectadas = models.TextField(
        help_text="Piezas dentales tratadas (ej: 16, 17, 18)"
    )
    
    class Meta:
        verbose_name = "Historial de Tratamiento"
        verbose_name_plural = "Historiales de Tratamientos"
        ordering = ['-fecha_realizacion']
    
    def __str__(self):
        return f"{self.tratamiento.nombre_tratamiento} - {self.paciente} ({self.fecha_realizacion.date()})"


# --- 6. Modelo de Factura Dental ---

class FacturaDental(models.Model):
    """
    Modelo para gestionar las facturas de los pacientes.
    """
    ESTADO_PAGO_CHOICES = [
        ('PENDIENTE', 'Pendiente'),
        ('PAGADO', 'Pagado'),
        ('PARCIAL', 'Pago parcial'),
        ('VENCIDO', 'Vencido'),
    ]
    
    METODO_PAGO_CHOICES = [
        ('EFECTIVO', 'Efectivo'),
        ('TARJETA', 'Tarjeta'),
        ('TRANSFERENCIA', 'Transferencia'),
        ('CHEQUE', 'Cheque'),
        ('SEGURO', 'Seguro dental'),
    ]
    
    paciente = models.ForeignKey(PacienteDental, on_delete=models.CASCADE, 
                                related_name='facturas')
    fecha_emision = models.DateField(auto_now_add=True, help_text="Fecha de emisión")
    total_factura = models.DecimalField(max_digits=10, decimal_places=2, 
                                       help_text="Monto total de la factura")
    estado_pago = models.CharField(max_length=50, choices=ESTADO_PAGO_CHOICES, 
                                  default='PENDIENTE')
    metodo_pago = models.CharField(max_length=50, choices=METODO_PAGO_CHOICES, 
                                  blank=True, null=True)
    fecha_vencimiento = models.DateField(help_text="Fecha límite de pago")
    cita_asociada = models.ForeignKey(CitaDental, on_delete=models.SET_NULL, 
                                     null=True, blank=True, related_name='facturas')
    descuento_aplicado = models.DecimalField(max_digits=5, decimal_places=2, 
                                            default=0.00, 
                                            help_text="Descuento en porcentaje")
    
    class Meta:
        verbose_name = "Factura Dental"
        verbose_name_plural = "Facturas Dentales"
        ordering = ['-fecha_emision']
    
    def __str__(self):
        return f"Factura #{self.id} - {self.paciente} - ${self.total_factura}"
    
    def calcular_total_con_descuento(self):
        """Calcula el total con descuento aplicado"""
        descuento = self.total_factura * (self.descuento_aplicado / 100)
        return self.total_factura - descuento


# --- 7. Modelo de Proveedor Dental ---

class ProveedorDental(models.Model):
    """
    Modelo para gestionar los proveedores de materiales e insumos dentales.
    """
    nombre_empresa = models.CharField(max_length=150, unique=True, 
                                     help_text="Nombre de la empresa proveedora")
    nombre_contacto = models.CharField(max_length=100, 
                                      help_text="Nombre del contacto principal")
    telefono = models.CharField(max_length=20, help_text="Teléfono del proveedor")
    email = models.EmailField(max_length=100, help_text="Correo electrónico")
    direccion = models.CharField(max_length=255, help_text="Dirección del proveedor")
    rfc = models.CharField(max_length=20, unique=True, blank=True, null=True,
                          help_text="RFC del proveedor")
    tipo_productos = models.CharField(max_length=100, 
                                     help_text="Tipo de productos que suministra")
    activo = models.BooleanField(default=True, help_text="¿Proveedor activo?")
    
    class Meta:
        verbose_name = "Proveedor Dental"
        verbose_name_plural = "Proveedores Dentales"
        ordering = ['nombre_empresa']
    
    def __str__(self):
        return self.nombre_empresa


# --- 8. Modelo de Material Odontológico ---

class MaterialOdontologico(models.Model):
    """
    Modelo para gestionar el inventario de materiales odontológicos.
    """
    nombre_material = models.CharField(max_length=255, help_text="Nombre del material")
    descripcion = models.TextField(help_text="Descripción del material")
    stock_actual = models.PositiveIntegerField(default=0, 
                                              help_text="Cantidad disponible en inventario")
    fecha_caducidad = models.DateField(blank=True, null=True, 
                                      help_text="Fecha de caducidad")
    proveedor = models.ForeignKey(ProveedorDental, on_delete=models.SET_NULL, 
                                 null=True, related_name='materiales')
    odontologo = models.ForeignKey(Odontologo, on_delete=models.SET_NULL, 
                                  null=True, blank=True, related_name='materiales',
                                  help_text="Odontólogo responsable del material")
    costo_unitario = models.DecimalField(max_digits=10, decimal_places=2, 
                                        help_text="Costo por unidad")
    tipo_material = models.CharField(max_length=50, 
                                    help_text="Tipo/Categoría del material")
    ubicacion_almacen = models.CharField(max_length=100, 
                                        help_text="Ubicación en el almacén")
    
    class Meta:
        verbose_name = "Material Odontológico"
        verbose_name_plural = "Materiales Odontológicos"
        ordering = ['nombre_material']
    
    def __str__(self):
        return f"{self.nombre_material} (Stock: {self.stock_actual})"
    
    def esta_por_caducar(self, dias=30):
        """Verifica si el material está por caducar en los próximos días especificados"""
        from datetime import date, timedelta
        if self.fecha_caducidad:
            fecha_limite = date.today() + timedelta(days=dias)
            return self.fecha_caducidad <= fecha_limite
        return False
    
    def necesita_reorden(self, minimo=10):
        """Verifica si el stock está bajo y necesita reordenarse"""
        return self.stock_actual <= minimo

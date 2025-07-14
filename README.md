
# 🛠️ Gestión de Incidencias – App Android

## 📌 Descripción del Proyecto

La aplicación **Gestión de Incidencias** está diseñada para facilitar el **registro, seguimiento y control de incidentes** dentro de una organización.  
Permite a los usuarios iniciar sesión según su rol (empleado, supervisor o administrador), **reportar accidentes o fallas**, y monitorear el estado de cada incidencia.  
El sistema organiza la información por **zonas**, **tipos de incidentes** y **usuarios responsables**, brindando una herramienta clara y eficiente para mejorar la comunicación y la atención de problemas operativos.

---

## 👨‍💻 Integrantes del grupo

- Jampier Orlando Tuya Barja  
- Henry Erick Quispe Bojorquez  
- Andre Stefano León De La Cruz  
- Ronald Maihuire Becerra

---

-- Database: GloriaConnect_Grupo4

-- Crear la tabla rol
CREATE TABLE rol (
    id_rol SERIAL PRIMARY KEY,
    descripcion_rol VARCHAR(60) NOT NULL,
    activo BOOLEAN NOT NULL DEFAULT TRUE -- Para desactivación lógica
);


-- Crear la tabla permiso
CREATE TABLE permiso (
    id_permiso SERIAL PRIMARY KEY,
    descripcion_permiso VARCHAR(60) NOT NULL,
    activo BOOLEAN NOT NULL DEFAULT TRUE -- Para desactivación lógica
);


-- Crear la tabla rol permiso
CREATE TABLE rol_permiso (
    id_rol INT NOT NULL,
    id_permiso INT NOT NULL,
    PRIMARY KEY (id_rol, id_permiso),
    FOREIGN KEY (id_rol) REFERENCES rol(id_rol),
    FOREIGN KEY (id_permiso) REFERENCES permiso(id_permiso)
);


-- Crear la tabla usuario
CREATE TABLE usuario (
    id_usuario SERIAL PRIMARY KEY,
    id_rol INT NOT NULL DEFAULT 1,
    nombres VARCHAR(100) NOT NULL,
    apellidos VARCHAR(100) NOT NULL,
    correo VARCHAR(100) NOT NULL UNIQUE CHECK (correo ~* '^[^@]+@[^@]+\.[^@]+$'),
    contrasena BYTEA NOT NULL, -- BCRYPT o similar
    intentos_fallidos INT DEFAULT 0,
    ultimo_intento_fallido TIMESTAMP NULL,
	verificado BOOLEAN NOT NULL DEFAULT FALSE, -- Confirmación de cuenta por correo
    token_verificacion VARCHAR(100), -- Código/token temporal para confirmar registro
    fecha_verificacion TIMESTAMP, -- Fecha en que confirmó
    activo BOOLEAN NOT NULL DEFAULT TRUE, -- Para desactivación lógica
    FOREIGN KEY (id_rol) REFERENCES rol(id_rol)
);


-- Crear la tabla sexo
CREATE TABLE sexo (
    id_sexo SERIAL PRIMARY KEY,
    sexo VARCHAR(20) NOT NULL CHECK (sexo IN ('Masculino', 'Femenino')),
    activo BOOLEAN NOT NULL DEFAULT TRUE -- Para desactivación lógica
);


-- Crear la tabla estado civil
CREATE TABLE estado_civil (
    id_estado_civil SERIAL PRIMARY KEY,
    estado VARCHAR(20) NOT NULL CHECK (estado IN ('Soltero(a)', 'Casado(a)', 'Divorciado(a)', 'Viudo(a)')),
    activo BOOLEAN NOT NULL DEFAULT TRUE -- Para desactivación lógica
);


-- Crear la tabla departamento
CREATE TABLE departamento (
    id_departamento SERIAL PRIMARY KEY,
    nombre_departamento VARCHAR(50) NOT NULL,
    activo BOOLEAN NOT NULL DEFAULT TRUE -- Para desactivación lógica
);


-- Crear la tabla provincia
CREATE TABLE provincia (
    id_provincia SERIAL PRIMARY KEY,
    nombre_provincia VARCHAR(50) NOT NULL,
    id_departamento INT NOT NULL,
    activo BOOLEAN NOT NULL DEFAULT TRUE, -- Para desactivación lógica
    FOREIGN KEY (id_departamento) REFERENCES departamento(id_departamento)
);


-- Crear la tabla distrito
CREATE TABLE distrito (
    id_distrito SERIAL PRIMARY KEY,
    nombre_distrito VARCHAR(50) NOT NULL,
    id_provincia INT NOT NULL,
    activo BOOLEAN NOT NULL DEFAULT TRUE, -- Para desactivación lógica
    FOREIGN KEY (id_provincia) REFERENCES provincia(id_provincia)
);


-- Crear la tabla empleado
CREATE TABLE empleado (
    id_empleado SERIAL PRIMARY KEY,
    id_usuario INT NOT NULL,
    tipo_documento VARCHAR(3) NOT NULL CHECK (tipo_documento IN ('DNI', 'CE')),
    numero_documento VARCHAR(12) NOT NULL UNIQUE,
    apellido_paterno VARCHAR(50) NOT NULL,
    apellido_materno VARCHAR(50),
    nombres VARCHAR(100) NOT NULL,
    id_sexo INT NOT NULL,
    id_estado_civil INT NOT NULL,
    direccion VARCHAR(100) NOT NULL,
    id_departamento INT NOT NULL,
    id_provincia INT NOT NULL,
    id_distrito INT NOT NULL,
    discapacidad CHAR(2) NOT NULL CHECK (discapacidad IN ('Si', 'No')),
    descripcion_discapacidad VARCHAR(255),
    telefono VARCHAR(9) UNIQUE CHECK (telefono ~ '^[0-9]{9}$'),
    celular VARCHAR(9) UNIQUE CHECK (celular ~ '^[0-9]{9}$'),
    edad INT NOT NULL CHECK (edad BETWEEN 18 AND 100),
    url_foto VARCHAR(255), -- URL en cloud
    fecha_ingreso DATE DEFAULT CURRENT_DATE,
    activo BOOLEAN DEFAULT FALSE -- El empleado necesita ser aprobado manualmente
    FOREIGN KEY (id_usuario) REFERENCES usuario(id_usuario),
    FOREIGN KEY (id_sexo) REFERENCES sexo(id_sexo),
    FOREIGN KEY (id_estado_civil) REFERENCES estado_civil(id_estado_civil),
    FOREIGN KEY (id_departamento) REFERENCES departamento(id_departamento),
    FOREIGN KEY (id_provincia) REFERENCES provincia(id_provincia),
    FOREIGN KEY (id_distrito) REFERENCES distrito(id_distrito)
);


-- Crear la tabla experiencia laboral
CREATE TABLE experiencia_laboral (
    id_experiencia SERIAL PRIMARY KEY,
    id_empleado INT NOT NULL,
    fecha_inicio DATE NOT NULL,
    fecha_fin DATE NOT NULL,
    cargo VARCHAR(100) NOT NULL,
    empresa VARCHAR(100) NOT NULL,
    descripcion_logros TEXT,
    url_certificado VARCHAR(255), -- URL si el archivo está en Firebase, S3, etc.
    activo BOOLEAN NOT NULL DEFAULT TRUE, -- Para desactivación lógica
    FOREIGN KEY (id_empleado) REFERENCES empleado(id_empleado),
    CHECK (fecha_fin >= fecha_inicio)
);


-- Crear la tabla incidencia
CREATE TABLE incidencia (
    id_incidencia SERIAL PRIMARY KEY,
    id_empleado INT NOT NULL,
    titulo VARCHAR(100) NOT NULL,
    descripcion TEXT NOT NULL,
    fecha_reporte TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    estado VARCHAR(20) DEFAULT 'Pendiente' CHECK (estado IN ('Pendiente', 'En Proceso', 'Resuelto', 'Cancelado')),
    tipo VARCHAR(50) NOT NULL CHECK (tipo IN ('Accidente', 'Falla Técnica', 'Operativa', 'Otra')),
    prioridad VARCHAR(20) CHECK (prioridad IN ('Alta', 'Media', 'Baja')),
    url_archivo VARCHAR(255), -- Archivo externo (ej: foto del incidente)
    FOREIGN KEY (id_empleado) REFERENCES empleado(id_empleado)
);


-- Crear la tabla historial estado incidencia
CREATE TABLE historial_estado_incidencia (
    id_historial SERIAL PRIMARY KEY,
    id_incidencia INT NOT NULL,
    estado_anterior VARCHAR(20) NOT NULL,
    estado_nuevo VARCHAR(20) NOT NULL,
    fecha_cambio TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    usuario_responsable INT,
    comentario TEXT,
    FOREIGN KEY (id_incidencia) REFERENCES incidencia(id_incidencia),
    FOREIGN KEY (usuario_responsable) REFERENCES usuario(id_usuario)
);


-- Crear la tabla permiso laboral
CREATE TABLE permiso_laboral (
    id_permiso_laboral SERIAL PRIMARY KEY,
    id_empleado INT NOT NULL,
    motivo TEXT NOT NULL,
    fecha_solicitud TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    fecha_inicio DATE NOT NULL,
    fecha_fin DATE NOT NULL,
    estado VARCHAR(20) DEFAULT 'Pendiente' CHECK (estado IN ('Pendiente', 'Aprobado', 'Rechazado', 'Cancelado')),
    url_documento VARCHAR(255), -- PDF u otro archivo como sustento
    FOREIGN KEY (id_empleado) REFERENCES empleado(id_empleado),
    CHECK (fecha_fin >= fecha_inicio)
);


-- Crear la tabla historial estado permiso laboral
CREATE TABLE historial_estado_permiso_laboral (
    id_historial SERIAL PRIMARY KEY,
    id_permiso_laboral INT NOT NULL,
    estado_anterior VARCHAR(20) NOT NULL,
    estado_nuevo VARCHAR(20) NOT NULL,
    fecha_cambio TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    usuario_responsable INT,
    comentario TEXT,
    FOREIGN KEY (id_permiso_laboral) REFERENCES permiso_laboral(id_permiso_laboral),
    FOREIGN KEY (usuario_responsable) REFERENCES usuario(id_usuario)
);


-- Crear la tabla reporte area
CREATE TABLE reporte_area (
    id_reporte SERIAL PRIMARY KEY,
    id_usuario INT NOT NULL,
    titulo VARCHAR(100) NOT NULL,
    contenido TEXT NOT NULL,
    fecha_reporte TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    periodo VARCHAR(20) NOT NULL, -- Ej: "Mayo 2025"
    tipo VARCHAR(50) CHECK (tipo IN ('Producción', 'Mantenimiento', 'Seguridad', 'Calidad', 'Otro')),
	estado VARCHAR(20) DEFAULT 'Enviado' CHECK (estado IN ('Enviado', 'Observado', 'Corregido', 'Aprobado')),
    url_archivo VARCHAR(255), -- Documento o gráfico asociado
    FOREIGN KEY (id_usuario) REFERENCES usuario(id_usuario)
);


-- Crear la tabla historial estado reporte area
CREATE TABLE historial_estado_reporte_area (
    id_historial SERIAL PRIMARY KEY,
    id_reporte INT NOT NULL,
    estado_anterior VARCHAR(20) NOT NULL,
    estado_nuevo VARCHAR(20) NOT NULL,
    fecha_cambio TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    usuario_responsable INT,
    comentario TEXT,
    FOREIGN KEY (id_reporte) REFERENCES reporte_area(id_reporte),
    FOREIGN KEY (usuario_responsable) REFERENCES usuario(id_usuario)
);


-- Crear la tabla administrativo
CREATE TABLE administrativo (
    id_admin SERIAL PRIMARY KEY,
    id_usuario INT NOT NULL UNIQUE,
    cargo VARCHAR(50), -- Ej: "Supervisor de Producción", "Jefe RRHH"
    area VARCHAR(50),  -- Ej: "Producción", "RRHH"
    telefono VARCHAR(9) CHECK (telefono ~ '^[0-9]{9}$'),
    activo BOOLEAN DEFAULT TRUE, -- Para desactivación lógica
    FOREIGN KEY (id_usuario) REFERENCES usuario(id_usuario)
);


-- Crear la tabla auditoria general
CREATE TABLE auditoria_general (
    id_auditoria SERIAL PRIMARY KEY,
    tabla_afectada VARCHAR(50) NOT NULL,
    operacion VARCHAR(10) NOT NULL CHECK (operacion IN ('INSERT', 'UPDATE', 'DELETE')),
    fecha_operacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    id_usuario_app INT NULL,  -- Usuario autenticado desde app (si existe)
    usuario_bd TEXT DEFAULT current_user, -- Usuario real de PostgreSQL que hizo la operación
    datos_anteriores XML,
    datos_nuevos XML
);


-- Crear función de auditoría en cualquier tabla
CREATE OR REPLACE FUNCTION fn_auditoria_general() RETURNS TRIGGER AS $$
DECLARE
    datos_old XML;
    datos_new XML;
    usuario_app_id INT;
BEGIN
    -- Obtener el usuario autenticado desde la app (si se ha seteado)
    BEGIN
        usuario_app_id := current_setting('app.current_user_id')::INT;
    EXCEPTION WHEN OTHERS THEN
        usuario_app_id := NULL;
    END;

    -- Auditoría por tipo de operación
    IF (TG_OP = 'DELETE') THEN
        SELECT xmlelement(name fila, xmlforest(OLD.*)) INTO datos_old;
        INSERT INTO auditoria_general (
            tabla_afectada, operacion, datos_anteriores, id_usuario_app
        )
        VALUES (
            TG_TABLE_NAME, TG_OP, datos_old, usuario_app_id
        );

    ELSIF (TG_OP = 'INSERT') THEN
        SELECT xmlelement(name fila, xmlforest(NEW.*)) INTO datos_new;
        INSERT INTO auditoria_general (
            tabla_afectada, operacion, datos_nuevos, id_usuario_app
        )
        VALUES (
            TG_TABLE_NAME, TG_OP, datos_new, usuario_app_id
        );

    ELSIF (TG_OP = 'UPDATE') THEN
        SELECT xmlelement(name fila, xmlforest(OLD.*)) INTO datos_old;
        SELECT xmlelement(name fila, xmlforest(NEW.*)) INTO datos_new;
        INSERT INTO auditoria_general (
            tabla_afectada, operacion, datos_anteriores, datos_nuevos, id_usuario_app
        )
        VALUES (
            TG_TABLE_NAME, TG_OP, datos_old, datos_new, usuario_app_id
        );
    END IF;

    RETURN NULL;
END;
$$ LANGUAGE plpgsql;


-- Crear Trigger para auditoria Usuario
CREATE TRIGGER tr_usuario_auditoria
AFTER INSERT OR UPDATE OR DELETE ON usuario
FOR EACH ROW EXECUTE FUNCTION fn_auditoria_general();


-- Crear Trigger para auditoria Empleado
CREATE TRIGGER tr_empleado_auditoria
AFTER INSERT OR UPDATE OR DELETE ON empleado
FOR EACH ROW EXECUTE FUNCTION fn_auditoria_general();


-- Crear Trigger para auditoria Experiencia laboral
CREATE TRIGGER tr_experiencia_laboral_auditoria
AFTER INSERT OR UPDATE OR DELETE ON experiencia_laboral
FOR EACH ROW EXECUTE FUNCTION fn_auditoria_general();


-- Crear Trigger para auditoria Incidencia
CREATE TRIGGER tr_incidencia_auditoria
AFTER INSERT OR UPDATE OR DELETE ON incidencia
FOR EACH ROW EXECUTE FUNCTION fn_auditoria_general();


--Crear Trigger para auditoria Permiso laboral
CREATE TRIGGER tr_permiso_laboral_auditoria
AFTER INSERT OR UPDATE OR DELETE ON permiso_laboral
FOR EACH ROW EXECUTE FUNCTION fn_auditoria_general();


-- Crear Trigger para auditoria Reporte área
CREATE TRIGGER tr_reporte_area_auditoria
AFTER INSERT OR UPDATE OR DELETE ON reporte_area
FOR EACH ROW EXECUTE FUNCTION fn_auditoria_general();


-- Crear Trigger para auditoria Administrativo
CREATE TRIGGER tr_administrativo_auditoria
AFTER INSERT OR UPDATE OR DELETE ON administrativo
FOR EACH ROW EXECUTE FUNCTION fn_auditoria_general();


-- -- Habilita la función unaccent(), que permite realizar búsquedas insensibles a tildes y mayúsculas con ILIKE y consultas como unaccent(columna) ILIKE unaccent(:valor)
CREATE EXTENSION IF NOT EXISTS unaccent;


-- Asegurar que pgcrypto esté habilitado (requerido para funciones como gen_salt, crypt, etc. al encriptar contraseñas)
CREATE EXTENSION IF NOT EXISTS pgcrypto;

---

## 🗂️ Interfaces de la App

- `LoginActivity` → Ingreso por rol (empleado, supervisor o admin)  
- `HomeActivity` → Menú principal con navegación a funcionalidades  
- `ReportarIncidenciaActivity` → Registrar un nuevo incidente  
- `HistorialActivity` → Lista y estado de incidencias por usuario  
- `DetalleIncidenciaActivity` → Ver información detallada + acciones

---

## 🚀 Funcionalidades

✅ Login seguro por correo y contraseña  
✅ Reporte de incidentes con descripción y selección de maquinaria/zona  
✅ Registro de imágenes (adjunto)  
✅ Seguimiento del estado de cada incidencia  
✅ Carga inicial de datos (roles, zonas, maquinarias)  
✅ Interfaz intuitiva y modular

---

Diseño De La APK

![Diseño del la ap](https://github.com/user-attachments/assets/a1a40c30-fa64-4031-b71b-cc1159a84f4d)

## ▶️ Ejecución del Proyecto

Postam: https://documenter.getpostman.com/view/39799790/2sB2x6kX6r#d27f152d-657f-4daa-87b5-612923fa85a0

---

Estructura De la Base de Datos 


![BASE DE DATOS](https://github.com/user-attachments/assets/3e3850ca-1091-425a-9db1-a86705d707f5)



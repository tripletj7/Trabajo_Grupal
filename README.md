🛠️ Gestión de Incidencias – App Android

Aplicación Android nativa desarrollada con Java y SQLite, que permite a los trabajadores de una empresa reportar, registrar y dar seguimiento a incidencias como accidentes o fallas de maquinaria.
Incluye control de roles, zonas y estados de cada caso para una mejor gestión interna.

👨‍💻 Integrantes del grupo
Jampier 
# 🛠️ Gestión de Incidencias – App Android

Aplicación Android nativa desarrollada con **Java** y **SQLite**, que permite a los trabajadores de una empresa **reportar, registrar y dar seguimiento a incidencias** como accidentes o fallas de maquinaria.  
Incluye control de roles, zonas y estados de cada caso para una mejor gestión interna.

---

## 👨‍💻 Integrantes del grupo

- Jampier Orlando Tuya Barja  
- Henry Erick Quispe Bojorquez  
- Andre Stefano León De La Cruz  
- Ronald Maihuire Becerra

---

## 🧠 Estructura de la Base de Datos

```sql
-- Roles de usuario
CREATE TABLE roles (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(50) NOT NULL UNIQUE
);

-- Usuarios con login
CREATE TABLE usuarios (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    rol_id INT NOT NULL,
    FOREIGN KEY (rol_id) REFERENCES roles(id)
);

-- Zonas o áreas de trabajo
CREATE TABLE zonas (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL
);

-- Maquinarias por zona
CREATE TABLE maquinarias (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    codigo_interno VARCHAR(50),
    zona_id INT,
    estado ENUM('Operativo', 'Malogrado', 'En revisión') DEFAULT 'Operativo',
    FOREIGN KEY (zona_id) REFERENCES zonas(id)
);

-- Registro de incidencias
CREATE TABLE incidencias (
    id INT AUTO_INCREMENT PRIMARY KEY,
    tipo ENUM('Accidente', 'Maquinaria', 'Otro') NOT NULL,
    descripcion TEXT NOT NULL,
    fecha_reporte DATETIME DEFAULT CURRENT_TIMESTAMP,
    estado ENUM('Pendiente', 'En proceso', 'Resuelto') DEFAULT 'Pendiente',
    usuario_id INT NOT NULL,
    maquinaria_id INT,
    zona_id INT,
    FOREIGN KEY (usuario_id) REFERENCES usuarios(id),
    FOREIGN KEY (maquinaria_id) REFERENCES maquinarias(id),
    FOREIGN KEY (zona_id) REFERENCES zonas(id)
);

-- Archivos adjuntos (ej. fotos)
CREATE TABLE archivos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    incidencia_id INT NOT NULL,
    ruta_archivo VARCHAR(255) NOT NULL,
    tipo_archivo VARCHAR(50),
    FOREIGN KEY (incidencia_id) REFERENCES incidencias(id)
);

-- Acciones realizadas
CREATE TABLE acciones (
    id INT AUTO_INCREMENT PRIMARY KEY,
    incidencia_id INT NOT NULL,
    usuario_id INT NOT NULL,
    descripcion TEXT,
    fecha DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (incidencia_id) REFERENCES incidencias(id),
    FOREIGN KEY (usuario_id) REFERENCES usuarios(id)
);

-- Notificaciones internas
CREATE TABLE notificaciones (
    id INT AUTO_INCREMENT PRIMARY KEY,
    incidencia_id INT,
    usuario_id INT NOT NULL,
    mensaje TEXT NOT NULL,
    leido BOOLEAN DEFAULT FALSE,
    fecha DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (incidencia_id) REFERENCES incidencias(id),
    FOREIGN KEY (usuario_id) REFERENCES usuarios(id)
);
```

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


![estructura](https://github.com/user-attachments/assets/d73f87e1-4256-4aa9-bcae-077d09e23d5d)


## 📌 Descripción del Proyecto

La aplicación **Gestión de Incidencias** está diseñada para facilitar el **registro, seguimiento y control de incidentes** dentro de una organización.  
Permite a los usuarios iniciar sesión según su rol (empleado, supervisor o administrador), **reportar accidentes o fallas**, y monitorear el estado de cada incidencia.  
El sistema organiza la información por **zonas**, **tipos de incidentes** y **usuarios responsables**, brindando una herramienta clara y eficiente para mejorar la comunicación y la atención de problemas operativos.


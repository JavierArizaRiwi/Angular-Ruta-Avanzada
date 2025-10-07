# Manejo Profesional de Excepciones con Enums en Java

## Introducción

En este documento se presenta una estructura profesional y escalable para el manejo de excepciones en aplicaciones Java, aplicable a proyectos con **arquitectura por capas** (Presentación, Negocio, Datos, Dominio).  
El enfoque combina el uso de **enums** para centralizar los códigos de error, mensajes y severidades, con una jerarquía de excepciones personalizada para mejorar la trazabilidad, la mantenibilidad y la claridad del código.

---

## 1. Concepto general

Una excepción es un evento que interrumpe el flujo normal de ejecución de un programa. En sistemas empresariales, es fundamental que los errores sean controlados, identificables y comunicados de forma consistente.

Este enfoque busca estandarizar los errores mediante:

- Jerarquía de excepciones personalizadas.
- Uso de `enum` para definir códigos, mensajes y severidad.
- Traducción de errores técnicos a errores de negocio.
- Centralización de mensajes para mantenimiento y localización.

---

## 2. Jerarquía de excepciones

Las excepciones se dividen en dos grandes grupos: **Negocio** e **Infraestructura**.

| Tipo | Descripción | Ejemplos |
|------|--------------|-----------|
| **Negocio** | Errores que violan reglas del dominio o validaciones. | Datos inválidos, duplicados, inexistentes |
| **Infraestructura** | Errores del entorno técnico (BD, red, archivos, etc.) | `SQLException`, fallos de conexión, transacciones |

Jerarquía recomendada:

```
MiniTiendaException (RuntimeException)
├─ NegocioException
│  ├─ DatoInvalidoException
│  ├─ DuplicadoException
│  └─ ProductoNoEncontradoException
└─ InfraestructuraException
   └─ PersistenciaException
```

---

## 3. Enum de códigos de error

El `enum` centraliza los códigos, mensajes y niveles de severidad.  
Esto permite tener un catálogo unificado de errores que puede integrarse con logs, auditorías o servicios REST.

```java
public enum ErrorCode {

    // --- Negocio / Validaciones ---
    INVALID_DATA("E001", "Datos inválidos o incompletos", Severity.WARN),
    DUPLICATE_RESOURCE("E002", "El recurso ya existe", Severity.WARN),
    RESOURCE_NOT_FOUND("E003", "Recurso no encontrado", Severity.INFO),

    // --- Persistencia / Infraestructura ---
    DB_ERROR("E100", "Error general de base de datos", Severity.ERROR),
    DB_CONNECTION_FAILED("E101", "Error de conexión con la base de datos", Severity.FATAL),
    TRANSACTION_ERROR("E102", "Error durante la transacción", Severity.ERROR),

    // --- Otros ---
    UNKNOWN_ERROR("E999", "Error desconocido", Severity.FATAL);

    private final String code;
    private final String message;
    private final Severity severity;

    ErrorCode(String code, String message, Severity severity) {
        this.code = code;
        this.message = message;
        this.severity = severity;
    }

    public String getCode() { return code; }
    public String getMessage() { return message; }
    public Severity getSeverity() { return severity; }
}
```

Enum auxiliar para severidad:

```java
public enum Severity {
    INFO, WARN, ERROR, FATAL
}
```

---

## 4. Clase base de excepciones

```java
public abstract class MiniTiendaException extends RuntimeException {
    private final ErrorCode errorCode;

    protected MiniTiendaException(ErrorCode errorCode) {
        super(errorCode.getMessage());
        this.errorCode = errorCode;
    }

    protected MiniTiendaException(ErrorCode errorCode, Throwable cause) {
        super(errorCode.getMessage(), cause);
        this.errorCode = errorCode;
    }

    public ErrorCode getErrorCode() { return errorCode; }
    public String getCode() { return errorCode.getCode(); }
    public Severity getSeverity() { return errorCode.getSeverity(); }

    @Override
    public String toString() {
        return String.format("[%s] %s - %s",
                errorCode.getSeverity(), errorCode.getCode(), getMessage());
    }
}
```

---

## 5. Excepciones específicas

```java
public class DatoInvalidoException extends MiniTiendaException {
    public DatoInvalidoException() {
        super(ErrorCode.INVALID_DATA);
    }
}

public class DuplicadoException extends MiniTiendaException {
    public DuplicadoException() {
        super(ErrorCode.DUPLICATE_RESOURCE);
    }
}

public class PersistenciaException extends MiniTiendaException {
    public PersistenciaException(Throwable cause) {
        super(ErrorCode.DB_ERROR, cause);
    }
}
```

---

## 6. Aplicación por capa

| Capa | Responsabilidad | Ejemplo de excepción |
|------|------------------|----------------------|
| **Presentación (UI)** | Captura excepciones y muestra mensajes amigables. | Manejo de `MiniTiendaException` con JOptionPane |
| **Negocio (Service)** | Valida datos y lanza excepciones personalizadas. | `DatoInvalidoException`, `DuplicadoException` |
| **Datos (DAO)** | Traduce errores técnicos (SQL, I/O) a `PersistenciaException`. | Captura de `SQLException` |
| **Dominio (Model)** | Define las reglas e invariantes. | Validaciones dentro del constructor o métodos |

---

## 7. Ejemplo de uso completo

### Validación en la capa de negocio

```java
if (precio <= 0 || stock < 0) {
    throw new DatoInvalidoException();
}
```

### Traducción en la capa DAO

```java
catch (SQLException e) {
    if (e.getErrorCode() == 1062) {
        throw new DuplicadoException();
    } else {
        throw new PersistenciaException(e);
    }
}
```

### Manejo en la capa de presentación

```java
try {
    servicio.agregarProducto(nombre, precio, stock);
} catch (MiniTiendaException e) {
    JOptionPane.showMessageDialog(null,
        String.format("%s: %s", e.getCode(), e.getMessage()),
        e.getSeverity().name(),
        JOptionPane.ERROR_MESSAGE);
}
```

---

## 8. Ventajas del uso de Enums en Excepciones

| Beneficio | Descripción |
|------------|-------------|
| **Centralización** | Todos los errores se definen en un solo lugar. |
| **Legibilidad** | Códigos y mensajes claros y estandarizados. |
| **Internacionalización (i18n)** | Fácil integración con `ResourceBundle`. |
| **Trazabilidad** | Cada error tiene código y severidad identificable. |
| **Integración REST** | Se puede devolver el `ErrorCode` en respuestas JSON. |

---

## 9. Ejemplo de respuesta JSON (API REST)

```json
{
  "errorCode": "E002",
  "message": "El recurso ya existe",
  "severity": "WARN"
}
```

---

## 10. Conclusión

El uso de enums para el manejo de excepciones permite crear una base sólida y profesional para sistemas empresariales.  
Este enfoque mejora la **consistencia**, **mantenibilidad** y **seguridad**, al tiempo que proporciona una estructura preparada para escalar hacia frameworks como **Spring Boot**, **Jakarta EE** o **Micronaut**.

---

**Autor:** Javier Ariza – Formación Avanzada en Arquitectura Java
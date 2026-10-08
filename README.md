# ARQUITECTURA-DE-DATOS
# ADR-001: Uso de PostgreSQL (Supabase) como motor de base de datos de AutoCheck

| Campo | Valor |
|---|---|
| **Estado** | Aceptada |
| **Fecha** | 2026-10-08 |
| **Proyecto** | AutoCheck: preperitaje vehicular con IA |
| **Equipo** | Leni Santiago Murillo Sterling, Duban Camilo Tibaquira Arenas, Jhon Kevyn Barragan Vasquez |
| **Curso** | Arquitectura de Software (NRC 60-95404), UNIMINUTO |

## Contexto

AutoCheck es una aplicación web que permite a un usuario registrar su vehículo, subir fotografías, responder un cuestionario y recibir un análisis preliminar (posibles daños y rango de valor) generado con la API de Gemini, antes de pagar un peritaje profesional.

En el modelado con C4 (Niveles 1, 2 y 3) quedó definido que el contenedor **API/Backend** (Node.js + Express) necesita persistir:

- **Usuarios** y su sesión.
- **Vehículos** (marca, modelo, año, kilometraje, placa), siempre asociados a un usuario.
- **Respuestas del cuestionario** y **resultados del análisis** (daños detectados y rango de valor), asociados a un vehículo.
- **Fotografías**, que son archivos pesados y se guardan aparte; en la base de datos solo se almacena la referencia (URL).

Restricciones y factores que influyen en la decisión:

- Los datos tienen **relaciones claras** (usuario → vehículos → fotos, respuestas y resultados) y requieren integridad referencial.
- Es un **proyecto académico**: sin presupuesto, tiempo limitado a un semestre, por lo que se necesita una opción con **capa gratuita** y poca configuración.
- Se requiere también **autenticación** y **almacenamiento de archivos**, y el equipo prefiere no construirlos desde cero.
- Se necesita proteger datos personales y fotografías de los usuarios (requisito no funcional RNF03).

## Decisión

Se usará **PostgreSQL** como motor de base de datos relacional, **provisto a través de Supabase** (servicio BaaS), junto con **Supabase Storage** para las fotografías y **Supabase Auth** para el registro e inicio de sesión.

El API/Backend se comunicará con PostgreSQL mediante **SQL** y con Storage mediante **HTTPS**. Las imágenes no se guardan en la base de datos: solo su URL.

## Consecuencias

### Positivas

- El modelo relacional encaja de forma natural con las entidades del dominio y garantiza integridad con llaves foráneas.
- Una sola plataforma cubre base de datos, autenticación y almacenamiento, lo que reduce el tiempo de desarrollo.
- La capa gratuita de Supabase es suficiente para el alcance académico.
- SQL estándar: el equipo puede migrar a otro PostgreSQL (propio o en la nube) con poco esfuerzo si el proyecto crece.

### Negativas y riesgos

- **Dependencia de un proveedor externo** (Supabase): una caída o un cambio en su capa gratuita afectaría al sistema.
- Los límites de la capa gratuita (almacenamiento, ancho de banda, proyectos pausados por inactividad) pueden afectar demostraciones prolongadas.
- Los cambios de esquema requieren **migraciones**, lo que añade disciplina de versionado que un enfoque NoSQL flexible evitaría.
- Los datos personales y fotos quedan alojados en un tercero, por lo que hay que configurar bien las reglas de acceso (Row Level Security) y las políticas de privacidad.

## Alternativas consideradas

| Alternativa | Motivo de descarte |
|---|---|
| **Firebase (Firestore)** | NoSQL: obliga a duplicar o anidar datos para representar las relaciones; menos natural para consultas como "todos los preperitajes de un usuario". |
| **MongoDB** | Mismo inconveniente de modelo documental, y exigiría resolver por separado autenticación y almacenamiento. |
| **SQLite** | Muy simple, pero no está pensado para varios usuarios concurrentes ni para un servicio web desplegado. |
| **PostgreSQL autoadministrado** | Más control, pero requiere montar y mantener el servidor, sin presupuesto ni tiempo para ello. |

## Referencias

- Modelo C4 del proyecto (informe técnico, Niveles 1 a 3) y contrato de API definido en la Fase 2.
- Diagrama PlantUML asociado: `ADR-001-componentes.puml`.
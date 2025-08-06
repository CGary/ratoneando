# Guía de Arquitectura Full-Stack: Puertos, Adaptadores y Slicing Vertical

## 1\. Fundamentos: Protegiendo lo que Realmente Importa

### 1.1. ¿Por qué necesitamos una Arquitectura?

En el desarrollo de software moderno, es fácil caer en la trampa de acoplar nuestra lógica de negocio directamente a los detalles de implementación. Esto ocurre tanto en el frontend (acoplamiento a frameworks de UI como React/Svelte, APIs del navegador) como en el backend (acoplamiento al framework web como Django/FastAPI, al ORM, o a protocolos específicos).

Esto genera problemas comunes:

  - **Acoplamiento Fuerte**: Cambiar de framework o reemplazar una librería se convierte en una tarea titánica.
  - **Lógica de Negocio Dispersa**: Las reglas importantes del negocio terminan esparcidas entre componentes, controladores y manejadores de eventos, haciendo difícil su comprensión y mantenimiento.
  - **Dificultad para Testear**: Probar la lógica de negocio requiere levantar un entorno completo (un navegador, un servidor web, una base de datos), mockear APIs y lidiar con un entorno complejo, lo que resulta en tests lentos y frágiles.

El objetivo principal de esta arquitectura es **proteger la lógica de negocio de los detalles de implementación**. Queremos que nuestro núcleo de negocio sea independiente del framework, de cómo se obtienen los datos y de cómo se presentan al usuario.

### 1.2. Puertos y Adaptadores (La "Arquitectura Hexagonal")

Imagina tu aplicación como una ciudadela. Dentro de los muros está lo más valioso: las reglas de negocio. Todo lo demás (la UI, la base de datos, las APIs externas) está en el "mundo exterior".

  - **Puertos**: Son las puertas de la ciudadela. No son la tecnología en sí, sino la **definición del contrato** de cómo se interactúa con el mundo exterior. En nuestro código, un puerto es una `interfaz` (TypeScript) o una `clase base abstracta` (Python).
  - **Adaptadores**: Son las implementaciones concretas que se conectan a esos puertos.
      - **Adaptadores de Entrada (Driving)**: Inician una acción en la aplicación (ej. un componente de UI, un controlador de API).
      - **Adaptadores de Salida (Driven)**: Son invocados por la aplicación para interactuar con el exterior (ej. un cliente HTTP, una implementación de repositorio con un ORM).

<!-- end list -->

```mermaid
graph TD
    subgraph Frontend (TypeScript)
        A[UI Components <br/>(Adaptador de Entrada)] --> B{Caso de Uso Frontend};
        B --> C[Puerto de Repositorio <br/>(e.g., AppRepository)];
        C --> D{Adaptador de Salida <br/>(Cliente HTTP)};
    end

    subgraph Backend (Python)
        G[Repositorio ORM <br/>(Adaptador de Salida)] --> F{Puerto de Repositorio <br/>(e.g., AppRepository)};
        F --> E{Caso de Uso Backend};
        E --> H[Handler/View <br/>(Adaptador de Entrada)];
    end

    subgraph "Red (Contrato API)"
        D -- Petición JSON --> I(API Endpoint <br/> e.g., /api/v1/apps);
        I -- Respuesta JSON --> D;
    end

    H -- Invoca --> E;
    I -- Delega a --> H;

    subgraph "Base de Datos"
      G -- Accede a --> J[DB];
    end

    style A fill:#cde4ff
    style D fill:#cde4ff
    style G fill:#cde4ff
    style H fill:#cde4ff
```

### 1.3. Las Tres Capas Fundamentales

  - **Dominio (El Núcleo)**: Contiene las reglas de negocio más puras y los **Puertos**. No depende de NADA.
  - **Aplicación (El Orquestador)**: Contiene los **casos de uso** que coordinan los flujos de trabajo. Depende únicamente del Dominio.
  - **Infraestructura (El Mundo Exterior)**: Contiene todo lo demás: UI, controladores, ORMs, etc. Aquí viven los **Adaptadores**. Depende de la Aplicación y del Dominio.

### 1.4. La Regla de la Dependencia: El Flujo Unidireccional

Esta es la regla más importante: **Las dependencias solo pueden apuntar hacia adentro.**

**Infraestructura → Aplicación → Dominio**

Esta regla garantiza que el núcleo de nuestra aplicación permanezca desacoplado y protegido.

## 2\. Organización del Código: Del Caos al Orden

### 2.1. Vertical Slicing: Organizando por Concepto de Negocio

Organizamos el código por **concepto de negocio** o **funcionalidad** (un "slice"), no por tipo técnico. Cada slice contiene su propio dominio, aplicación e infraestructura.

### 2.2. La Estructura de Directorios Propuesta

Para un proyecto full-stack, una estructura de monorepo es ideal.

```
monorepo/
|-- apps/
|   |-- frontend/               # Aplicación Frontend (TypeScript: Svelte/React)
|   `-- backend/                # Aplicación Backend (Python: Django/FastAPI)
|
|-- project_name/
|       |   |-- billing/        # Slice de Negocio: "Facturación"
|       |   |   |-- domain/
|       |   |   |-- application/
|       |   |   `-- infrastructure/
|       |   `-- users/          # Slice de Negocio: "Usuarios"
|       |-- ...
|       `-- manage.py
|
`-- packages/
    `-- shared-domain/          # El "Shared Kernel" con tipos de TypeScript
        `-- src/
            `-- index.ts        # Tipos y entidades compartidas (ej. UserInfo)
```

> **Nota**: El paquete `shared-domain` contiene definiciones de tipos en TypeScript que pueden ser usadas por el frontend y como una referencia para los contratos de API del backend (ej. para validar DTOs), asegurando la consistencia entre ambos.

## 3\. Guía de Implementación Práctica (Frontend)

Se ilustra la creación de la funcionalidad de navegación usando Svelte.

#### Paso 1: Definir el Dominio

```typescript
// packages/shared-domain/src/index.ts
export type AppId = 'anst' | 'cmd' | 'delfi' | 'erp';

export interface AppInfo {
  id: AppId;
  name: string;
  urlRedirect: string;
}

// apps/frontend/src/navigation/domain/appRepository.ts
import type { AppInfo } from 'shared-domain';

export type AppRepository = {
  getApps: () => Promise<AppInfo[]>;
};
```

#### Paso 2: Crear el Caso de Uso

```typescript
// apps/frontend/src/navigation/application/getApps.ts
import type { AppRepository } from '../domain/appRepository';
import type { AppInfo } from 'shared-domain';

export const getAppsUseCase = (repository: AppRepository) => async (): Promise<AppInfo[]> => {
  return await repository.getApps();
};
```

#### Paso 3: Implementar el Adaptador de Salida

```typescript
// apps/frontend/src/navigation/infrastructure/staticAppRepository.ts
import type { AppInfo } from 'shared-domain';
import type { AppRepository } from '../domain/appRepository';

const appsDefault: AppInfo[] = [/* ...lista de apps... */];

export const staticAppRepository: AppRepository = {
  getApps: async () => Promise.resolve(appsDefault),
};
```

#### Paso 4: Crear la Raíz de Composición (Composition Root)

```typescript
// apps/frontend/src/navigation/infrastructure/context.ts
import { getContext, setContext } from 'svelte';
import { getAppsUseCase } from '../application/getApps';
import { staticAppRepository } from './staticAppRepository';

const APPS_CONTEXT_KEY = Symbol('apps_context');

export function provideAppsContext() {
  const repository = staticAppRepository;
  const getApps = getAppsUseCase(repository);
  setContext(APPS_CONTEXT_KEY, getApps);
}

export function useAppsContext() {
  return getContext<ReturnType<typeof getAppsUseCase>>(APPS_CONTEXT_KEY);
}
```

#### Paso 5: Usar en el Adaptador de Entrada (UI)

```html
<script lang="ts">
  import { useAppsContext } from '../infrastructure/context';
  const getApps = useAppsContext();
</script>

{#await getApps()}
  {/await}
```

## 4\. Guía de Implementación Práctica (Backend)

Ejemplo paralelo en Python para registrar un usuario.

#### Paso 1: Definir el Dominio

```python
# apps/backend/src/users/domain/user.py
from dataclasses import dataclass

@dataclass
class User:
    id: int | None
    email: str

# apps/backend/src/users/domain/user_repository.py
import abc
from .user import User

class AbstractUserRepository(abc.ABC):
    @abc.abstractmethod
    def add(self, user: User): ...
```

#### Paso 2: Crear el Caso de Uso

```python
# apps/backend/src/users/application/register_user.py
from ..domain.user import User
from ..domain.user_repository import AbstractUserRepository

def register_user_use_case(email: str, repo: AbstractUserRepository):
    # ...lógica de negocio...
    new_user = User(id=None, email=email)
    repo.add(new_user)
```

#### Paso 3: Implementar el Adaptador de Salida

```python
# apps/backend/src/users/infrastructure/django_user_repository.py
from django.contrib.auth.models import User as DjangoUser
from ..domain.user import User
from ..domain.user_repository import AbstractUserRepository

class DjangoUserRepository(AbstractUserRepository):
    def add(self, user: User):
        DjangoUser.objects.create_user(username=user.email, email=user.email)
```

#### Paso 4: Usar en el Adaptador de Entrada y Raíz de Composición

```python
# apps/backend/src/users/infrastructure/api_views.py
from rest_framework.views import APIView
from ..application.register_user import register_user_use_case
from .django_user_repository import DjangoUserRepository

class RegisterUserView(APIView):
    def post(self, request):
        # --- Raíz de Composición ---
        repo = DjangoUserRepository()
        # -------------------------
        register_user_use_case(email=request.data['email'], repo=repo)
        # ...
```

### 4.1. Patrones Avanzados de Backend

  - **Inyección de Dependencias Explícita**: El fichero `api_views.py` actúa como nuestra **Raíz de Composición (Composition Root)**, donde "cableamos" las implementaciones concretas. Para proyectos más grandes, se puede considerar el uso de un contenedor de inyección de dependencias como `dependency-injector` para gestionar este grafo de dependencias de forma más robusta.
  - **Patrón Unit of Work**: Para casos de uso que necesitan ser atómicos (o todo o nada), se usa el patrón **Unit of Work**. Se define un puerto en el dominio (`UnitOfWork`) que tiene métodos como `commit()` y `rollback()`, y expone los repositorios. El caso de uso lo recibe, y un adaptador de entrada (o un decorador) se encarga de gestionar la transacción, manteniendo la lógica de negocio limpia.

## 5\. Comunicación y Sincronización Frontend-Backend

### 5.1. Contratos de API como Puertos

Los Data Transfer Objects (DTOs) que cruzan la red son parte del dominio compartido. Usa **OpenAPI/Swagger** para describir endpoints y modelos.

### 5.2. Sincronización de Tipos

**Ejemplo Práctico de openapi-typescript**: Si tu backend genera un `openapi.json`, sincronizar el frontend es tan simple como ejecutar:

```sh
npx openapi-typescript ./openapi.json --output ./packages/shared-domain/src/api-types.ts
```

Esto genera automáticamente las interfaces TypeScript para todos los DTOs y endpoints, eliminando la duplicación manual.

### 5.3. Evolución: El Patrón BFF (Backend for Frontend)

A veces, una única API no es óptima para todas las interfaces (web, móvil). El patrón **BFF** introduce una capa intermedia, un backend específico para un frontend, que consume la API principal y la adapta a las necesidades de la UI.

## 6\. Estrategia de Testing Holística

### 6.1. Testear el Caso de Uso (Backend)

```python
# tests/users/test_register_user.py
from unittest.mock import Mock
from ...apps.backend.src.users.application.register_user import register_user_use_case

def test_register_new_user_successfully():
    mock_repo = Mock()
    register_user_use_case(email="test@example.com", repo=mock_repo)
    mock_repo.add.assert_called_once()
```

### 6.2. Testear el Componente de UI (Frontend)

Se testea el componente inyectando un caso de uso mockeado a través del contexto, probando la UI sin depender de la infraestructura real.

### 6.3. Testing End-to-End (Playwright / Cypress)

Finalmente, los tests E2E validan el flujo completo simulando a un usuario real en el navegador. Estos tests verifican que la **integración entre todos los adaptadores** (UI, HTTP, ORM) funciona correctamente.

## 7\. Gobernanza y Adopción

### 7.1. Manteniendo la Integridad

  - **Configurar un Linter**: Usa ESLint (Frontend) y `import-linter` (Python) para prohibir importaciones que violen la regla de la dependencia.
  - **Revisiones de Código**: Son la herramienta principal para mantener la disciplina.

### 7.2. Documentando Decisiones con ADRs

Para mantener la integridad a largo plazo, documenta por qué se toman ciertas decisiones. Utiliza **Architectural Decision Records (ADRs)**, documentos cortos que registran una decisión, su contexto y sus consecuencias. Esto ayuda a los nuevos miembros del equipo a entender el razonamiento y evita que las decisiones clave se reviertan sin una buena razón.

## 8\. Fuentes y Lecturas Recomendadas

Para profundizar en los conceptos presentados en esta guía, se recomiendan los siguientes recursos:

### Arquitectura Hexagonal / Puertos y Adaptadores

  - **Hexagonal Architecture** - El artículo original de Alistair Cockburn que introdujo el concepto. Es la fuente primaria y fundamental.
  - **Ports and Adapters Pattern** - Una descripción concisa en el C2 Wiki, ideal para una comprensión rápida.

### Arquitectura Limpia y Principios Relacionados

  - **The Clean Architecture** - El influyente artículo de Robert C. Martin (Uncle Bob) que popularizó la idea de las capas concéntricas y la regla de la dependencia.
  - **The Dependency Inversion Principle** - Un artículo de Martin Fowler que explica uno de los principios SOLID clave que sustenta esta arquitectura.

### Vertical Slicing

  - **Vertical Slice Architecture** - Artículo de Jimmy Bogard que argumenta a favor de organizar el código por funcionalidad en lugar de por capas técnicas.
  - **Feature-Sliced Design** - Una metodología muy bien documentada y específica para frontend que aplica los principios de Vertical Slicing de una manera estructurada.

### Diseño Guiado por el Dominio (DDD)

  - **Domain-Driven Design Distilled** por Vaughn Vernon - Un libro conciso y accesible que sirve como una excelente introducción a los conceptos clave de DDD.
  - **DDD Community** - Un recurso comunitario con artículos, videos y discusiones sobre DDD.

### Enfoque Funcional

  - **Domain Modeling Made Functional** por Scott Wlaschin - Aunque enfocado en F\#, este libro es una obra maestra para entender cómo aplicar los principios de DDD en un paradigma funcional, utilizando tipos para modelar el dominio y evitar estados inválidos.

### Inyección de Dependencias y Contextos

  - **Dependency injection in Svelte for fun and profit** por Kyle Nazario - Aunque es un artículo que se centra en el *patrón* Composition Root (Raíz de Composición) en Svelte.
  - El *artículo* cita una definición muy famosa y práctica de la inyección de dependencias:
    > "Dependency Injection" is a 25-dollar term for a 5-cent concept... [it] means giving an object its instances variables. Really. That's it.
    > —James Shore.
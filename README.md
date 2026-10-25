# Guía de Inicio Rápido de TestSprite (TestSprite Quickstart en Español)

TestSprite es una plataforma avanzada de pruebas de software impulsada por agentes de inteligencia artificial autónomos que generan, ejecutan y mantienen pruebas de extremo a extremo (E2E) tanto para interfaces de usuario (Frontend UI) como para servicios backend (API REST / GraphQL).

Esta guía técnica está diseñada para desarrolladores que desean integrar TestSprite en sus flujos de trabajo locales, integración continua (CI/CD) o entornos de desarrollo asistidos por IA mediante el protocolo MCP (Model Context Protocol).

---

## 1. Conceptos Fundamentales de la Arquitectura

A diferencia de los frameworks tradicionales donde el desarrollador redacta manualmente cada assertion (`expect()`, `assert`), TestSprite adopta un enfoque semántico centrado en especificaciones:

* **PRD-Driven Testing:** Utiliza un Documento de Requisitos del Producto (PRD en Markdown, texto plano o PDF) para mapear la intención real del negocio.
* **Grafo de Flujo de Funcionalidades (Feature Map Flow Graph):** Descompone automáticamente el sistema en nodos de interacción y casos de uso estructurados.
* **Generación de Código Real:**
  * **Proyectos de UI:** Genera suites completas en **Playwright** (TypeScript/JavaScript).
  * **Proyectos de API:** Genera scripts de prueba modulares en **Python** con inspección profunda de headers, status codes y payloads JSON.
* **Modo MCP (Model Context Protocol):** Permite invocar la suite de pruebas directamente desde editores modernos como Cursor, Windsurf o Claude Code.

---

## 2. Requisitos Previos

Antes de comenzar, asegúrate de contar con:
* Una cuenta activa en [TestSprite](https://testsprite.com).
* Node.js v18.0.0 o superior (para pruebas de UI y CLI).
* Python 3.10 o superior (para pruebas de API backend).
* Acceso a tu repositorio de código o a la URL del entorno de staging/desarrollo.

---

## 3. Configuración del Primer Proyecto

### Paso 3.1: Creación del Proyecto en el Dashboard
1. Ingresa a tu panel de control de TestSprite y haz clic en **Create Tests** (o **New Project**).
2. Asigna un nombre a tu proyecto (por ejemplo, `ecommerce-web-app`).
3. Define la **URL base** de la aplicación (por ejemplo: `https://staging.tuapp.com` o `http://localhost:3000`).

### Paso 3.2: Carga del PRD (Especificación de Requisitos)
Para que el agente de pruebas entienda la lógica de negocio y no solo la estructura estática del DOM, sube tu PRD:
```markdown
# Especificación del Módulo de Autenticación y Checkout
## 1. Inicio de Sesión
- Los usuarios registrados ingresan con email y contraseña.
- Se debe mostrar un mensaje de error claro en caso de credenciales inválidas.
- La sesión persistirá mediante un token Bearer en almacenamiento seguro.

## 2. Carrito de Compras
- Los usuarios pueden agregar productos con variaciones de cantidad.
- El subtotal y los impuestos deben actualizarse dinámicamente antes del checkout.
```
*Nota: Si estás realizando una prueba de concepto rápida, puedes omitir la carga del PRD, pero se recomienda firmemente incluirlo para lograr una cobertura óptima.*

---

## 4. Selección del Tipo de Prueba y Ejecución

### Opción A: Pruebas de Interfaz de Usuario (UI con Playwright)
TestSprite analiza el árbol DOM de tu aplicación, identifica selectores estables (accesibilidad, roles ARIA, test IDs) y crea suites reproducibles.

Ejemplo de código Playwright generado por el agente:
```typescript
import { test, expect } from '@playwright/test';

test.describe('Flujo de Autenticación y Perfil', () => {
  test('debe iniciar sesión con credenciales válidas y redirigir al dashboard', async ({ page }) => {
    await page.goto('https://staging.tuapp.com/login');
    
    // El agente utiliza selectores de accesibilidad resilientes
    await page.getByLabel('Correo electrónico').fill('usuario.demo@empresa.com');
    await page.getByLabel('Contraseña').fill('PruebaSegura123!');
    await page.getByRole('button', { name: 'Iniciar Sesión' }).click();

    // Verificación de redirección y carga de métricas
    await expect(page).toHaveURL(/.*\/dashboard/);
    await expect(page.getByTestId('user-welcome-banner')).toBeVisible();
  });
});
```

### Opción B: Pruebas de Backend / API (Python)
Para servicios backend, TestSprite lee tu esquema OpenAPI / Swagger o hace un escaneo de endpoints, generando validaciones completas:

```python
import pytest
import requests

BASE_URL = "https://api.staging.tuapp.com/v1"

def test_create_order_authenticated(auth_headers):
    payload = {
        "items": [{"item_id": "prod_9921", "quantity": 2}],
        "currency": "USD"
    }
    
    response = requests.post(f"{BASE_URL}/orders", json=payload, headers=auth_headers)
    assert response.status_code == 201
    
    data = response.json()
    assert data["status"] == "pending"
    assert data["total_amount"] > 0
    assert "order_id" in data
```

---

## 5. Revisión, Ejecución y Análisis de Resultados

Una vez que el agente compile el plan de pruebas:
1. Revisa la lista de casos generados en el panel. Puedes editar, aprobar o desactivar casos específicos.
2. Haz clic en **Run all** para despachar la ejecución en la infraestructura cloud de TestSprite.
3. El panel transmitirá en tiempo real el estado de cada prueba:
   * **Passed (Aprobada):** El flujo cumplió con las tolerancias y assertions.
   * **Failed (Fallida):** La prueba falló; TestSprite proporciona automáticamente captura de pantalla, traza de red, logs de consola y una **causa raíz sugerida por IA** con la corrección recomendada.
   * **Blocked (Bloqueada):** Un paso previo impidió la ejecución (por ejemplo, fallo en la fase de login previo al checkout).

---

## 6. Integración Continua (CI/CD) con GitHub Actions

Para asegurar que cada Pull Request se valide automáticamente antes de fusionarse a la rama principal, crea el siguiente flujo en `.github/workflows/testsprite.yml`:

```yaml
name: TestSprite Automated QA Suite

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  testsprite-run:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout del Repositorio
        uses: actions/checkout@v4

      - name: Configurar Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 18

      - name: Ejecutar TestSprite Suite
        env:
          TESTSPRITE_API_KEY: ${{ secrets.TESTSPRITE_API_KEY }}
          TESTSPRITE_PROJECT_ID: ${{ secrets.TESTSPRITE_PROJECT_ID }}
        run: |
          npx @testsprite/cli run --project "$TESTSPRITE_PROJECT_ID" --wait --ci
```

---

## 7. Uso de TestSprite mediante MCP (Model Context Protocol)

Si utilizas entornos como Cursor o Claude Code, puedes controlar TestSprite directamente con lenguaje natural sin salir del editor:

1. Agrega el servidor MCP en tu archivo `.mcp.json`:
```json
{
  "mcpServers": {
    "testsprite": {
      "command": "npx",
      "args": ["-y", "@testsprite/mcp@latest"],
      "env": {
        "TESTSPRITE_API_KEY": "tu_clave_api_aqui"
      }
    }
  }
}
```
2. Ejecuta comandos directamente en el chat del asistente:
   * `"@testsprite analiza el archivo src/routes/payments.ts y genera una suite de pruebas para los casos de error 400 y 402"`
   * `"@testsprite ejecuta todas las pruebas del proyecto y muéstrame los fallos detectados"`

---

## 8. Resumen y Mejores Prácticas

1. **Mantén tu PRD actualizado:** El agente refleja con exactitud la documentación provista.
2. **Utiliza atributos `data-testid`:** Facilita la generación de selectores altamente estables frente a cambios estéticos de CSS.
3. **Automatiza en Staging:** Conecta las pruebas en ramas de pre-producción para capturar regresiones silenciosas antes del despliegue final.

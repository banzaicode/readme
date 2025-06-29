# 🚀 Elección entre Minimal API vs Controllers en .NET Core

## 🔍 Contexto

Al desarrollar una API RESTful en .NET Core, una de las decisiones clave es elegir entre utilizar **Minimal API** (disponible desde .NET 6) o una arquitectura basada en **Controllers (MVC)**. Esta elección impacta directamente en la escalabilidad, mantenibilidad y rendimiento del proyecto.

---

## 🔵 Minimal API

### ✅ Ventajas

* **Menor boilerplate**: Ideal para microservicios, prototipos y MVPs.
* **Mejor rendimiento**: Evita capas intermedias como `Controller` o `ActionInvoker`.
* **Moderno y expresivo**: Aprovecha C# moderno (`top-level statements`, lambdas, etc.).
* **Fácil de integrar con Azure Functions o AWS Lambda**.

### ❌ Desventajas

* **Difícil de escalar**: En proyectos grandes puede volverse desordenado.
* **Menor separación de responsabilidades**.
* **Limitaciones con Model Binding avanzado y filtros personalizados**.

---

## 🔵 API con Controllers (MVC)

### ✅ Ventajas

* **Estructura tradicional y mantenible**: Ideal para equipos grandes y proyectos a largo plazo.
* **Soporte para filtros (`ActionFilter`, `ResultFilter`)**.
* **Mejor integración con herramientas como Swagger y Postman**.
* **Organización clara para aplicar patrones como CQRS, Mediator, etc.**
* **Validaciones complejas y model binding más robusto**.

### ❌ Desventajas

* **Verboso y más burocrático**.
* **Puede ser innecesariamente complejo para APIs pequeñas**.

---

## 🧐 Cuál deberías elegir?

| Escenario                                                        | Recomendación                              |
| ---------------------------------------------------------------- | ------------------------------------------ |
| MVP o prueba de concepto                                         | ✅ **Minimal API**                          |
| Microservicio pequeño o serverless                               | ✅ **Minimal API**                          |
| Proyecto grande o con equipo numeroso                            | ✅ **Controllers**                          |
| Requiere filtros, validaciones complejas o arquitectura en capas | ✅ **Controllers**                          |
| Búsqueda de rendimiento extremo y simplicidad                    | ✅ **Minimal API** (con buena organización) |

---

## 📊 Tendencia actual (junio 2025)

> “**Usar Minimal API bien estructurada para microservicios o endpoints técnicos, y Controllers para APIs medianas o grandes.**”

Incluso es común combinar ambos enfoques:

* `Minimal API` para endpoints como health checks o tareas técnicas.
* `Controllers` para el core funcional de la API.

## Ejemplos

### 🟦 Minimal API

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddScoped<IMyService, MyService>();

var app = builder.Build();

app.MapGet("/api/hello/{name}", (string name, IMyService svc) =>
{
    var result = svc.SayHello(name);
    return Results.Ok(new { message = result });
});

app.Run();

public interface IMyService
{
    string SayHello(string name);
}

public class MyService : IMyService
{
    public string SayHello(string name) => $"Hola {name}!";
}
```

### 🟦 API con Controllers

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllers();
builder.Services.AddScoped<IMyService, MyService>();
var app = builder.Build();

app.MapControllers();
app.Run();

// Controllers/HelloController.cs
[ApiController]
[Route("api/[controller]")]
public class HelloController : ControllerBase
{
    private readonly IMyService _svc;

    public HelloController(IMyService svc)
    {
        _svc = svc;
    }

    [HttpGet("{name}")]
    public IActionResult GetHello(string name)
    {
        var result = _svc.SayHello(name);
        return Ok(new { message = result });
    }
}

// Services/MyService.cs
public interface IMyService
{
    string SayHello(string name);
}

public class MyService : IMyService
{
    public string SayHello(string name) => $"Hola {name}!";
}
```

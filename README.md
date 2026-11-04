# blazor-app-ing
blazor-app-ing

                                                  PASOS A SEGUIR: 

Crear el proyecto con C# y Blazor WebAssembly en Visual Studio 2022
Si quieres hacerlo con C# en .NET 8:

1. ¿Qué plantilla elegir en Visual Studio 2022?
Abre Visual Studio 2022 y selecciona Crear un proyecto nuevo.

Busca la plantilla "Aplicación Blazor WebAssembly" (Blazor WebAssembly Standalone App / Aplicación Blazor WebAssembly independiente).

En la versión de Framework, selecciona .NET 8.0 (Soporte a largo plazo).

En tipo de autenticación deja Ninguno y desmarca las opciones de backend servidor (debe ser Standalone / independiente).

2. Estructura de archivos requerida
En la solución que te crea Visual Studio, el componente donde harás el formulario estará dentro de Pages/Index.razor (o un nuevo componente .razor que crees).

Para generar el archivo de texto y descargarlo desde C# en Blazor WebAssembly sin depender de librerías externas pesadas, tu archivo Pages/Index.razor tendría una estructura como esta:

@page "/"
@inject IJSRuntime JS

<h3>Calculadora y Generador de Archivo</h3>

<div class="form-group mb-3">
    <label>Valor A:</label>
    <input type="number" class="form-control" @bind="valorA" />
</div>

<div class="form-group mb-3">
    <label>Valor B:</label>
    <input type="number" class="form-control" @bind="valorB" />
</div>

<button class="btn btn-primary" @click="CalcularYDescargar">Calcular y Descargar TXT</button>

@if (resultado.HasValue)
{
    <p class="mt-3"><strong>Resultado:</strong> @resultado</p>
}

@code {
    private double valorA;
    private double valorB;
    private double? resultado;

    private async Task CalcularYDescargar()
    {
        // 1. Aplicar la fórmula matemática
        resultado = (valorA + valorB) * 1.19; // Ejemplo: suma + IVA

        // 2. Construir el texto del archivo
        string contenidoTxt = $"REPORTE DE CÁLCULO\n" +
                              $"-------------------\n" +
                              $"Valor A: {valorA}\n" +
                              $"Valor B: {valorB}\n" +
                              $"Resultado Final: {resultado}\n" +
                              $"Fecha: {DateTime.Now:yyyy-MM-dd HH:mm:ss}";

        // 3. Disparar la descarga mediante JavaScript interop
        var fileName = "Resultado_Calculo.txt";
        var fileBytes = System.Text.Encoding.UTF8.GetBytes(contenidoTxt);
        var base64 = Convert.ToBase64String(fileBytes);

        await JS.InvokeVoidAsync("downloadFileFromBase64", fileName, base64);
    }
}

Y en tu archivo wwwroot/index.html (dentro del <head> o antes del </body>), agregas esta pequeña función helper en JS para procesar la descarga que le ordena C#:

<script>
  window.downloadFileFromBase64 = (fileName, byteBase64) => {
    const link = document.createElement('a');
    link.download = fileName;
    link.href = 'data:application/octet-stream;base64,' + byteBase64;
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
  };
</script>


3. ¿Cómo publicar (Deploy) Blazor WebAssembly en GitHub Pages?
En Visual Studio, haz clic derecho en el proyecto -> Publicar (Publish).

Elige Carpeta (Folder) como destino.

Al publicar, .NET creará los archivos finales compilados dentro de bin/Release/net8.0/publish/wwwroot/. Esos archivos de la carpeta wwwroot son los que se suben a GitHub Pages.

Dos pasos obligatorios para GitHub Pages:

Crear .nojekyll: Agrega un archivo completamente vacío con el nombre .nojekyll en la raíz de tu repositorio de GitHub para evitar que borre carpetas internas de .NET (como _framework).

Ajustar la etiqueta Base: En el index.html compilado, si tu repositorio en GitHub se llama mi-herramienta, debes cambiar <base href="/" /> por <base href="/mi-herramienta/" />.

# Build an MCP server

> Reference: https://modelcontextprotocol.io/docs/develop/build-server

> Get started building your own server to use in Claude for Desktop and other clients.

In this tutorial, we'll build a simple MCP weather server and connect it to a host, Claude for Desktop.

### What we'll be building

We'll build a server that exposes two tools: `get_alerts` and `get_forecast`. Then we'll connect the server to an MCP host (in this case, Claude for Desktop):

<Frame>
  <img src="https://mintcdn.com/mcp/4ZXF1PrDkEaJvXpn/images/current-weather.png?fit=max&auto=format&n=4ZXF1PrDkEaJvXpn&q=85&s=dce7b2f8a06c20ba358e4bd2e75fa4c7" data-og-width="2780" width="2780" data-og-height="1849" height="1849" data-path="images/current-weather.png" data-optimize="true" data-opv="3" srcset="https://mintcdn.com/mcp/4ZXF1PrDkEaJvXpn/images/current-weather.png?w=280&fit=max&auto=format&n=4ZXF1PrDkEaJvXpn&q=85&s=bbb19f34c5df59f66bc6bbb75d2bc5ed 280w, https://mintcdn.com/mcp/4ZXF1PrDkEaJvXpn/images/current-weather.png?w=560&fit=max&auto=format&n=4ZXF1PrDkEaJvXpn&q=85&s=2392d7e765b897c5b78f9f53d41439d4 560w, https://mintcdn.com/mcp/4ZXF1PrDkEaJvXpn/images/current-weather.png?w=840&fit=max&auto=format&n=4ZXF1PrDkEaJvXpn&q=85&s=dc349e75341b046d35a649762774da49 840w, https://mintcdn.com/mcp/4ZXF1PrDkEaJvXpn/images/current-weather.png?w=1100&fit=max&auto=format&n=4ZXF1PrDkEaJvXpn&q=85&s=deeb99214d9383ee4a0c8aaacb120049 1100w, https://mintcdn.com/mcp/4ZXF1PrDkEaJvXpn/images/current-weather.png?w=1650&fit=max&auto=format&n=4ZXF1PrDkEaJvXpn&q=85&s=5c6f948059635e376deeadce3893e9b9 1650w, https://mintcdn.com/mcp/4ZXF1PrDkEaJvXpn/images/current-weather.png?w=2500&fit=max&auto=format&n=4ZXF1PrDkEaJvXpn&q=85&s=3922160478785cc88d5e98d418e8f7dd 2500w" />
</Frame>

<Note>
  Servers can connect to any client. We've chosen Claude for Desktop here for simplicity, but we also have guides on [building your own client](/docs/develop/build-client) as well as a [list of other clients here](/clients).
</Note>

### Core MCP Concepts

MCP servers can provide three main types of capabilities:

1. **[Resources](/docs/learn/server-concepts#resources)**: File-like data that can be read by clients (like API responses or file contents)
2. **[Tools](/docs/learn/server-concepts#tools)**: Functions that can be called by the LLM (with user approval)
3. **[Prompts](/docs/learn/server-concepts#prompts)**: Pre-written templates that help users accomplish specific tasks

This tutorial will primarily focus on tools.

  Let's get started with building our weather server! [You can find the complete code for what we'll be building here.](https://github.com/modelcontextprotocol/csharp-sdk/tree/main/samples/QuickstartWeatherServer)

  ### Prerequisite knowledge

  This quickstart assumes you have familiarity with:

  * C#
  * LLMs like Claude
  * .NET 8 or higher

  ### Logging in MCP Servers

  When implementing MCP servers, be careful about how you handle logging:

  **For STDIO-based servers:** Never write to standard output (stdout). This includes:

  * `print()` statements in Python
  * `console.log()` in JavaScript
  * `fmt.Println()` in Go
  * Similar stdout functions in other languages

  Writing to stdout will corrupt the JSON-RPC messages and break your server.

  **For HTTP-based servers:** Standard output logging is fine since it doesn't interfere with HTTP responses.

  ### Best Practices

  1. Use a logging library that writes to stderr or files

  ### System requirements

  * [.NET 8 SDK](https://dotnet.microsoft.com/download/dotnet/8.0) or higher installed.

  ### Set up your environment

  First, let's install `dotnet` if you haven't already. You can download `dotnet` from [official Microsoft .NET website](https://dotnet.microsoft.com/download/). Verify your `dotnet` installation:

  ```bash  theme={null}
  dotnet --version
  ```

  Now, let's create and set up your project:

  ```bash
  # Create a new directory for our project
  mkdir weather
  cd weather
  # Initialize a new C# project
  dotnet new console
  ```

  After running `dotnet new console`, you will be presented with a new C# project.
  You can open the project in your favorite IDE, such as [Visual Studio](https://visualstudio.microsoft.com/) or [Rider](https://www.jetbrains.com/rider/).
  Alternatively, you can create a C# application using the [Visual Studio project wizard](https://learn.microsoft.com/en-us/visualstudio/get-started/csharp/tutorial-console?view=vs-2022).
  After creating the project, add NuGet package for the Model Context Protocol SDK and hosting:

  ```bash
  # Add the Model Context Protocol SDK NuGet package
  dotnet add package ModelContextProtocol --prerelease
  # Add the .NET Hosting NuGet package
  dotnet add package Microsoft.Extensions.Hosting
  ```

  Now let’s dive into building your server.

  ## Building your server

  Open the `Program.cs` file in your project and replace its contents with the following code:

  ```csharp
  using Microsoft.Extensions.DependencyInjection;
  using Microsoft.Extensions.Hosting;
  using ModelContextProtocol;
  using System.Net.Http.Headers;

  var builder = Host.CreateEmptyApplicationBuilder(settings: null);

  builder.Services.AddMcpServer()
      .WithStdioServerTransport()
      .WithToolsFromAssembly();

  builder.Services.AddSingleton(_ =>
  {
      var client = new HttpClient() { BaseAddress = new Uri("https://api.weather.gov") };
      client.DefaultRequestHeaders.UserAgent.Add(new ProductInfoHeaderValue("weather-tool", "1.0"));
      return client;
  });

  var app = builder.Build();

  await app.RunAsync();
  ```

  <Note>
    When creating the `ApplicationHostBuilder`, ensure you use `CreateEmptyApplicationBuilder` instead of `CreateDefaultBuilder`. This ensures that the server does not write any additional messages to the console. This is only necessary for servers using STDIO transport.
  </Note>

  This code sets up a basic console application that uses the Model Context Protocol SDK to create an MCP server with standard I/O transport.

  ### Weather API helper functions

  Create an extension class for `HttpClient` which helps simplify JSON request handling:

  ```csharp
  using System.Text.Json;

  internal static class HttpClientExt
  {
      public static async Task<JsonDocument> ReadJsonDocumentAsync(this HttpClient client, string requestUri)
      {
          using var response = await client.GetAsync(requestUri);
          response.EnsureSuccessStatusCode();
          return await JsonDocument.ParseAsync(await response.Content.ReadAsStreamAsync());
      }
  }
  ```

  Next, define a class with the tool execution handlers for querying and converting responses from the National Weather Service API:

  ```csharp
  using ModelContextProtocol.Server;
  using System.ComponentModel;
  using System.Globalization;
  using System.Text.Json;

  namespace QuickstartWeatherServer.Tools;

  [McpServerToolType]
  public static class WeatherTools
  {
      [McpServerTool, Description("Get weather alerts for a US state code.")]
      public static async Task<string> GetAlerts(
          HttpClient client,
          [Description("The US state code to get alerts for.")] string state)
      {
          using var jsonDocument = await client.ReadJsonDocumentAsync($"/alerts/active/area/{state}");
          var jsonElement = jsonDocument.RootElement;
          var alerts = jsonElement.GetProperty("features").EnumerateArray();

          if (!alerts.Any())
          {
              return "No active alerts for this state.";
          }

          return string.Join("\n--\n", alerts.Select(alert =>
          {
              JsonElement properties = alert.GetProperty("properties");
              return $"""
                      Event: {properties.GetProperty("event").GetString()}
                      Area: {properties.GetProperty("areaDesc").GetString()}
                      Severity: {properties.GetProperty("severity").GetString()}
                      Description: {properties.GetProperty("description").GetString()}
                      Instruction: {properties.GetProperty("instruction").GetString()}
                      """;
          }));
      }

      [McpServerTool, Description("Get weather forecast for a location.")]
      public static async Task<string> GetForecast(
          HttpClient client,
          [Description("Latitude of the location.")] double latitude,
          [Description("Longitude of the location.")] double longitude)
      {
          var pointUrl = string.Create(CultureInfo.InvariantCulture, $"/points/{latitude},{longitude}");
          using var jsonDocument = await client.ReadJsonDocumentAsync(pointUrl);
          var forecastUrl = jsonDocument.RootElement.GetProperty("properties").GetProperty("forecast").GetString()
              ?? throw new Exception($"No forecast URL provided by {client.BaseAddress}points/{latitude},{longitude}");

          using var forecastDocument = await client.ReadJsonDocumentAsync(forecastUrl);
          var periods = forecastDocument.RootElement.GetProperty("properties").GetProperty("periods").EnumerateArray();

          return string.Join("\n---\n", periods.Select(period => $"""
                  {period.GetProperty("name").GetString()}
                  Temperature: {period.GetProperty("temperature").GetInt32()}°F
                  Wind: {period.GetProperty("windSpeed").GetString()} {period.GetProperty("windDirection").GetString()}
                  Forecast: {period.GetProperty("detailedForecast").GetString()}
                  """));
      }
  }
  ```

  ### Running the server

  Finally, run the server using the following command:

  ```bash
  dotnet run
  ```

  This will start the server and listen for incoming requests on standard input/output.

  ## Testing your server with Claude for Desktop

  <Note>
    Claude for Desktop is not yet available on Linux. Linux users can proceed to the [Building a client](/docs/develop/build-client) tutorial to build an MCP client that connects to the server we just built.
  </Note>

  First, make sure you have Claude for Desktop installed. [You can install the latest version here.](https://claude.ai/download) If you already have Claude for Desktop, **make sure it's updated to the latest version.**
  We'll need to configure Claude for Desktop for whichever MCP servers you want to use. To do this, open your Claude for Desktop App configuration at `~/Library/Application Support/Claude/claude_desktop_config.json` in a text editor. Make sure to create the file if it doesn't exist.
  For example, if you have [VS Code](https://code.visualstudio.com/) installed:

  ```bash macOS/Linux theme={null}
  code ~/Library/Application\ Support/Claude/claude_desktop_config.json
  ```

  You'll then add your servers in the `mcpServers` key. The MCP UI elements will only show up in Claude for Desktop if at least one server is properly configured.
  In this case, we'll add our single weather server like so:

  ```json
  {
    "mcpServers": {
      "weather": {
        "command": "dotnet",
        "args": ["run", "--project", "/ABSOLUTE/PATH/TO/PROJECT", "--no-build"]
      }
    }
  }
  ```

  This tells Claude for Desktop:

  1. There's an MCP server named "weather"
  2. Launch it by running `dotnet run /ABSOLUTE/PATH/TO/PROJECT`
     Save the file, and restart **Claude for Desktop**.

### Test with commands

Let's make sure Claude for Desktop is picking up the two tools we've exposed in our `weather` server. You can do this by looking for the "Search and tools"  icon:

<Frame>
  <img src="https://mintcdn.com/mcp/4ZXF1PrDkEaJvXpn/images/visual-indicator-mcp-tools.png?fit=max&auto=format&n=4ZXF1PrDkEaJvXpn&q=85&s=dee15f1044586f26f7c8f489d1b1bea1" data-og-width="2250" width="2250" data-og-height="462" height="462" data-path="images/visual-indicator-mcp-tools.png" data-optimize="true" data-opv="3" srcset="https://mintcdn.com/mcp/4ZXF1PrDkEaJvXpn/images/visual-indicator-mcp-tools.png?w=280&fit=max&auto=format&n=4ZXF1PrDkEaJvXpn&q=85&s=7d5a49e662150224cacc011f27310bd0 280w, https://mintcdn.com/mcp/4ZXF1PrDkEaJvXpn/images/visual-indicator-mcp-tools.png?w=560&fit=max&auto=format&n=4ZXF1PrDkEaJvXpn&q=85&s=8f4d36114ab38529374b8ed651bec92d 560w, https://mintcdn.com/mcp/4ZXF1PrDkEaJvXpn/images/visual-indicator-mcp-tools.png?w=840&fit=max&auto=format&n=4ZXF1PrDkEaJvXpn&q=85&s=6e0145760586e28c8d45e11424653912 840w, https://mintcdn.com/mcp/4ZXF1PrDkEaJvXpn/images/visual-indicator-mcp-tools.png?w=1100&fit=max&auto=format&n=4ZXF1PrDkEaJvXpn&q=85&s=17d958bc6c8ac8260c2b18d5e6e033bb 1100w, https://mintcdn.com/mcp/4ZXF1PrDkEaJvXpn/images/visual-indicator-mcp-tools.png?w=1650&fit=max&auto=format&n=4ZXF1PrDkEaJvXpn&q=85&s=0bdb3a92e0e7721921d77288c5594fd4 1650w, https://mintcdn.com/mcp/4ZXF1PrDkEaJvXpn/images/visual-indicator-mcp-tools.png?w=2500&fit=max&auto=format&n=4ZXF1PrDkEaJvXpn&q=85&s=5eabd994c742cc7379e99b908e9dea3e 2500w" />
</Frame>

After clicking on the slider icon, you should see two tools listed:

<Frame>
  <img src="https://mintcdn.com/mcp/4ZXF1PrDkEaJvXpn/images/available-mcp-tools.png?fit=max&auto=format&n=4ZXF1PrDkEaJvXpn&q=85&s=040f7b6ec62f24b8fa0aaf4c5cab2d29" data-og-width="638" width="638" data-og-height="240" height="240" data-path="images/available-mcp-tools.png" data-optimize="true" data-opv="3" srcset="https://mintcdn.com/mcp/4ZXF1PrDkEaJvXpn/images/available-mcp-tools.png?w=280&fit=max&auto=format&n=4ZXF1PrDkEaJvXpn&q=85&s=588fae5b4ba6c310a527e91eb20d83bf 280w, https://mintcdn.com/mcp/4ZXF1PrDkEaJvXpn/images/available-mcp-tools.png?w=560&fit=max&auto=format&n=4ZXF1PrDkEaJvXpn&q=85&s=d277a424f541ff9a901072a4103ff05e 560w, https://mintcdn.com/mcp/4ZXF1PrDkEaJvXpn/images/available-mcp-tools.png?w=840&fit=max&auto=format&n=4ZXF1PrDkEaJvXpn&q=85&s=d12c286d06565cd45e175ea08c59fcf4 840w, https://mintcdn.com/mcp/4ZXF1PrDkEaJvXpn/images/available-mcp-tools.png?w=1100&fit=max&auto=format&n=4ZXF1PrDkEaJvXpn&q=85&s=e30982d8fea169d8559777facd1a07e3 1100w, https://mintcdn.com/mcp/4ZXF1PrDkEaJvXpn/images/available-mcp-tools.png?w=1650&fit=max&auto=format&n=4ZXF1PrDkEaJvXpn&q=85&s=d90eaa83029d8ae66dec21d632c118b5 1650w, https://mintcdn.com/mcp/4ZXF1PrDkEaJvXpn/images/available-mcp-tools.png?w=2500&fit=max&auto=format&n=4ZXF1PrDkEaJvXpn&q=85&s=f89e9bb7244323d208cf367bcd45f4ef 2500w" />
</Frame>

If your server isn't being picked up by Claude for Desktop, proceed to the [Troubleshooting](#troubleshooting) section for debugging tips.

If the tool settings icon has shown up, you can now test your server by running the following commands in Claude for Desktop:

* What's the weather in Sacramento?
* What are the active weather alerts in Texas?

<Frame>
  <img src="https://mintcdn.com/mcp/4ZXF1PrDkEaJvXpn/images/current-weather.png?fit=max&auto=format&n=4ZXF1PrDkEaJvXpn&q=85&s=dce7b2f8a06c20ba358e4bd2e75fa4c7" data-og-width="2780" width="2780" data-og-height="1849" height="1849" data-path="images/current-weather.png" data-optimize="true" data-opv="3" srcset="https://mintcdn.com/mcp/4ZXF1PrDkEaJvXpn/images/current-weather.png?w=280&fit=max&auto=format&n=4ZXF1PrDkEaJvXpn&q=85&s=bbb19f34c5df59f66bc6bbb75d2bc5ed 280w, https://mintcdn.com/mcp/4ZXF1PrDkEaJvXpn/images/current-weather.png?w=560&fit=max&auto=format&n=4ZXF1PrDkEaJvXpn&q=85&s=2392d7e765b897c5b78f9f53d41439d4 560w, https://mintcdn.com/mcp/4ZXF1PrDkEaJvXpn/images/current-weather.png?w=840&fit=max&auto=format&n=4ZXF1PrDkEaJvXpn&q=85&s=dc349e75341b046d35a649762774da49 840w, https://mintcdn.com/mcp/4ZXF1PrDkEaJvXpn/images/current-weather.png?w=1100&fit=max&auto=format&n=4ZXF1PrDkEaJvXpn&q=85&s=deeb99214d9383ee4a0c8aaacb120049 1100w, https://mintcdn.com/mcp/4ZXF1PrDkEaJvXpn/images/current-weather.png?w=1650&fit=max&auto=format&n=4ZXF1PrDkEaJvXpn&q=85&s=5c6f948059635e376deeadce3893e9b9 1650w, https://mintcdn.com/mcp/4ZXF1PrDkEaJvXpn/images/current-weather.png?w=2500&fit=max&auto=format&n=4ZXF1PrDkEaJvXpn&q=85&s=3922160478785cc88d5e98d418e8f7dd 2500w" />
</Frame>

<Frame>
  <img src="https://mintcdn.com/mcp/4ZXF1PrDkEaJvXpn/images/weather-alerts.png?fit=max&auto=format&n=4ZXF1PrDkEaJvXpn&q=85&s=c4762bf2bd84a8781846d2965af3e4a4" data-og-width="2809" width="2809" data-og-height="1850" height="1850" data-path="images/weather-alerts.png" data-optimize="true" data-opv="3" srcset="https://mintcdn.com/mcp/4ZXF1PrDkEaJvXpn/images/weather-alerts.png?w=280&fit=max&auto=format&n=4ZXF1PrDkEaJvXpn&q=85&s=e25afdd84f6ae9c612b898c6eb9c518d 280w, https://mintcdn.com/mcp/4ZXF1PrDkEaJvXpn/images/weather-alerts.png?w=560&fit=max&auto=format&n=4ZXF1PrDkEaJvXpn&q=85&s=1e7ef678cbc93c0966789e61d5209092 560w, https://mintcdn.com/mcp/4ZXF1PrDkEaJvXpn/images/weather-alerts.png?w=840&fit=max&auto=format&n=4ZXF1PrDkEaJvXpn&q=85&s=4dbaeb8840a7b1aeb73b188804877d71 840w, https://mintcdn.com/mcp/4ZXF1PrDkEaJvXpn/images/weather-alerts.png?w=1100&fit=max&auto=format&n=4ZXF1PrDkEaJvXpn&q=85&s=68f5e0cb428c8b9cb53d28ec1108073b 1100w, https://mintcdn.com/mcp/4ZXF1PrDkEaJvXpn/images/weather-alerts.png?w=1650&fit=max&auto=format&n=4ZXF1PrDkEaJvXpn&q=85&s=56025243c2b8c6413f8da087122e848d 1650w, https://mintcdn.com/mcp/4ZXF1PrDkEaJvXpn/images/weather-alerts.png?w=2500&fit=max&auto=format&n=4ZXF1PrDkEaJvXpn&q=85&s=12f50039e4a1c9544a22a9bdae46f719 2500w" />
</Frame>

<Note>
  Since this is the US National Weather service, the queries will only work for US locations.
</Note>

## What's happening under the hood

When you ask a question:

1. The client sends your question to Claude
2. Claude analyzes the available tools and decides which one(s) to use
3. The client executes the chosen tool(s) through the MCP server
4. The results are sent back to Claude
5. Claude formulates a natural language response
6. The response is displayed to you!

## Troubleshooting

### Claude for Desktop Integration Issues

  **Getting logs from Claude for Desktop**

  Claude.app logging related to MCP is written to log files in `~/Library/Logs/Claude`:

  * `mcp.log` will contain general logging about MCP connections and connection failures.
  * Files named `mcp-server-SERVERNAME.log` will contain error (stderr) logging from the named server.

  You can run the following command to list recent logs and follow along with any new ones:

  ```bash  theme={null}
  # Check Claude's logs for errors
  tail -n 20 -f ~/Library/Logs/Claude/mcp*.log
  ```

  **Server not showing up in Claude**

  1. Check your `claude_desktop_config.json` file syntax
  2. Make sure the path to your project is absolute and not relative
  3. Restart Claude for Desktop completely

  <Warning>
    To properly restart Claude for Desktop, you must fully quit the application:

    * **Windows**: Right-click the Claude icon in the system tray (which may be hidden in the "hidden icons" menu) and select "Quit" or "Exit".
    * **macOS**: Use Cmd+Q or select "Quit Claude" from the menu bar.

    Simply closing the window does not fully quit the application, and your MCP server configuration changes will not take effect.
  </Warning>

  **Tool calls failing silently**

  If Claude attempts to use the tools but they fail:

  1. Check Claude's logs for errors
  2. Verify your server builds and runs without errors
  3. Try restarting Claude for Desktop

  **None of this is working. What do I do?**

  Please refer to our [debugging guide](/legacy/tools/debugging) for better debugging tools and more detailed guidance.

### Weather API Issues
  **Error: Failed to retrieve grid point data**

  This usually means either:

  1. The coordinates are outside the US
  2. The NWS API is having issues
  3. You're being rate limited

  Fix:

  * Verify you're using US coordinates
  * Add a small delay between requests
  * Check the NWS API status page

  **Error: No active alerts for \[STATE]**

  This isn't an error - it just means there are no current weather alerts for that state. Try a different state or check during severe weather.

<Note>
  For more advanced troubleshooting, check out our guide on [Debugging MCP](/legacy/tools/debugging)
</Note>

## Next steps

<CardGroup cols={2}>
  <Card title="Building a client" icon="outlet" href="/docs/develop/build-client">
    Learn how to build your own MCP client that can connect to your server
  </Card>

  <Card title="Example servers" icon="grid" href="/examples">
    Check out our gallery of official MCP servers and implementations
  </Card>

  <Card title="Debugging Guide" icon="bug" href="/legacy/tools/debugging">
    Learn how to effectively debug MCP servers and integrations
  </Card>

  <Card title="Building MCP with LLMs" icon="comments" href="/tutorials/building-mcp-with-llms">
    Learn how to use LLMs like Claude to speed up your MCP development
  </Card>
</CardGroup>


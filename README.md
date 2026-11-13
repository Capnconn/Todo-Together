# Todo-Together

A shared todo app for a two-person household. Each person can see what is on the other's plate, take tasks off it, and see what the other has done.

The full description of what the app does and why is in the [product brief](docs/product-brief.md).

## Status

Early development. The repository currently contains the project skeleton only.

## Stack

- .NET 10
- ASP.NET Core server with EF Core and SQLite
- Blazor WebAssembly client, installed on phones as a PWA
- SignalR for live updates and Web Push for notifications

## Layout

| Path | Contents |
|---|---|
| `src/TodoTogether.Server` | ASP.NET Core server. Hosts the API and serves the client. |
| `src/TodoTogether.Client` | Blazor WebAssembly client (PWA). |
| `src/TodoTogether.Shared` | Types shared by the server and the client. |
| `docs` | Product brief and other documentation. |

## Running locally

Requires the [.NET 10 SDK](https://dotnet.microsoft.com/download).

```
dotnet run --project src/TodoTogether.Server
```

Then open the URL printed in the console. The server serves the client, so this is the only project you need to run.

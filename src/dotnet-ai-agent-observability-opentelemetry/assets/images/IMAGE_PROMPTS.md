# Image prompt ideas — dotnet-ai-agent-observability-opentelemetry

## 1. Hero / social card
A clean, modern flat-illustration banner: a C#/.NET logo on the left connected by a glowing "OTLP" pipeline to three telemetry backends on the right (Aspire Dashboard, Jaeger, Azure Monitor). Dark developer-tool background, purple/teal accent palette. Overlay text space for the title "AI Agent Observability in .NET with OpenTelemetry". No stock-photo people.

## 2. Diagram — the .NET-native OTel bridge
Architecture diagram: boxes for "Microsoft.Extensions.AI", "Semantic Kernel", "OpenAI .NET SDK", "Agent Framework" all feeding into a shared "System.Diagnostics.ActivitySource / Meter" layer, which flows through "OpenTelemetry .NET SDK (AddSource/AddMeter + OTLP exporter)" out to a backend. Emphasize "no Python sidecar" with a crossed-out sidecar icon.

## 3. Diagram — multi-step agent trace tree
A waterfall/trace-tree visualization matching the article's ASCII example: root `invoke_agent TravelPlanner`, nested `chat`, `execute_tool search_flights`, `execute_tool get_weather`, and a child `invoke_agent BudgetChecker`. Annotate spans with `gen_ai.usage.input_tokens` / `output_tokens` and duration bars. Style like a Jaeger/Aspire timeline view.

## 4. Concept — "experimental / Development" warning callout
A small badge-style graphic: a blue "Development" stability badge next to a `gen_ai.*` code snippet, with a subtle "pin your versions" motif. Useful inline near the semantic-conventions status section.

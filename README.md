# GraphTools — Enhanced Call Graph Explorer

An offline, Roslyn-based code knowledge graph for .NET solutions. Build, query, compare, and visually explore call graphs and project dependencies across small and large codebases.

This fork extends the original GraphTools viewer with **method-level drill-down, a textual call tree, namespace visualization, and project summaries** to make navigating large, multi-project .NET solutions easier.

## Why GraphTools?

Understanding a large codebase often requires navigating multiple layers of method calls and project dependencies. GraphTools builds a structural map of your solution locally, allowing developers and AI coding agents to explore relationships between types, methods, and projects without repeatedly scanning source files.

- **Local analysis:** source code analysis runs locally through Roslyn.
- **Solution-wide visibility:** explore relationships across multiple projects.
- **Method-level navigation:** follow outgoing calls recursively.
- **Architecture discovery:** identify which namespaces and projects participate in a call chain.
- **AI-friendly graph data:** query specific symbols instead of loading the entire graph into an AI context.
- **Change analysis:** compare graph snapshots to identify structural changes.

## What's Included

| Component | Type | Purpose |
|---|---|---|
| `GraphTools.Core` | Class library | Shared graph models and Roslyn workspace loading |
| `GraphTools.Builder` | Console application | Builds, updates, and compares graph data |
| `GraphTools.Query` | Console application | Queries individual symbols and their relationships |
| `graph-viewer.html` | Standalone HTML file | Interactive visual and textual graph explorer |

## Features Inherited from the Original Project

- Project-to-project dependency visualization.
- Type-level dependency exploration.
- Relationships including calls, implementations, inheritance, usages, and overrides.
- Search, zoom, pan, and fullscreen support.
- Light and dark themes.
- Method and member information in the details panel.
- Optional node-size scaling.
- Graph loading through drag-and-drop or file selection.
- Full and incremental graph generation.
- JSON-based graph comparison.

## Enhancements in This Fork

### 1. Method-Level Drill-Down

Click a class to inspect its members. Click a method to open its outgoing call graph.

Continue clicking method nodes to explore deeper levels of the call hierarchy without manually searching for every referenced method.

The viewer uses the existing graph data and its `calls` relationships.

### 2. Call Depth Selector

The **Call depth** selector controls how many levels of outgoing calls are displayed.

For example, starting from `OrderService.InsertOrder()`:

- **Depth 1:** the selected method and its direct callees.
- **Depth 2:** direct callees and their callees.
- **Depth 3:** three levels of outgoing calls.
- **Depth 5 or higher:** explore deeper call chains.

Changing the depth refreshes the graph and the accompanying textual summaries.

### 3. Back and Forward Navigation

Navigate through previously selected methods using Back and Forward controls.

This makes it possible to explore a call chain and return to previously visited methods without losing your navigation history.

### 4. Textual Call Tree

The viewer includes a tree-style textual representation of the methods reachable from the selected method.

Example:

```text
OrderService.InsertOrder()
├── OrderValidator.ValidateOrder()
│   ├── CustomerService.GetCustomer()
│   │   └── CustomerRepository.GetById()
│   └── ProductService.ValidateProducts()
├── OrderCalculator.CalculateTotals()
│   ├── DiscountService.CalculateDiscount()
│   └── TaxService.CalculateTax()
└── OrderRepository.Save()
    └── AppDbContext.SaveChanges()
```

Features:

- Hierarchical representation of outgoing method calls.
- Expandable and collapsible branches.
- Navigation to a method's call graph by clicking its name.
- Integration with the Call depth selector.
- Detection of recursive call paths to prevent infinite expansion.

### 5. Namespace Identification and Color Coding

Methods in the textual tree display their namespace alongside their method name.

Namespaces are assigned consistent colors to make it easier to identify which parts of the application a call chain traverses.

This is particularly useful in solutions where application logic, domain models, persistence, and infrastructure are distributed across different namespaces.

### 6. Namespace Summary

The **Namespace Summary** section lists the distinct namespaces reached from the selected method.

Features:

- Duplicate namespaces are removed.
- Entries are sorted alphabetically.
- Namespace colors match those used in the textual call tree.
- The list follows the selected call depth.

Use this section to identify the logical areas of the codebase involved in a particular operation.

### 7. Projects List

The **Projects List** section displays the distinct projects represented in the explored call tree.

Example:

```text
Projects List

Orders.API
Orders.Application
Orders.Domain
Orders.Infrastructure
Shared.Messaging
```

Features:

- Unique project names.
- Alphabetically sorted entries.
- Automatic refresh when the selected method changes.
- Integration with the Call depth selector.

This provides a quick overview of the projects involved in a method's outgoing call chain.

## Prerequisites

- .NET 8 SDK.
- Visual Studio 2022 or a compatible MSBuild environment.
- NuGet access to restore the required Roslyn and MSBuild packages.

## Building

From the repository root:

```bash
dotnet build GraphTools.slnx
```

## Usage

### 1. Build a Graph

```bash
GraphTools.Builder.exe --solution "C:\src\MySolution.sln" --output "C:\graphs\full-graph.json" --mode full
```

The builder also produces `project-dependencies.json` in the output directory.

### 2. Update the Graph Incrementally

```bash
GraphTools.Builder.exe --solution "C:\src\MySolution.sln" --output "C:\graphs\full-graph.json" --mode incremental --graph "C:\graphs\full-graph.json"
```

Incremental mode reuses the existing graph and re-analyzes changed files according to the builder's change-detection logic.

### 3. Query a Symbol

```bash
GraphTools.Query.exe --graph "C:\graphs\full-graph.json" --symbol "MyApp.Orders.OrderService.InsertOrder()"
```

Query callers:

```bash
GraphTools.Query.exe --graph "C:\graphs\full-graph.json" --symbol "MyApp.Orders.OrderService.InsertOrder()" --direction callers
```

Query callees:

```bash
GraphTools.Query.exe --graph "C:\graphs\full-graph.json" --symbol "MyApp.Orders.OrderService.InsertOrder()" --direction callees
```

List symbols in a project:

```bash
GraphTools.Query.exe --graph "C:\graphs\full-graph.json" --list-symbols --project "Orders.Application"
```

Use the symbol IDs emitted by the graph or query tool when identifying methods.

### 4. Open the Enhanced Viewer

1. Generate `full-graph.json` and `project-dependencies.json`.
2. Open `docs/graph-viewer.html` in a modern browser, or use the viewer location appropriate to your checkout.
3. Load the JSON files using drag-and-drop or the file picker if prompted.
4. Click a project, then a type, and select a method.
5. Use the call graph to navigate through outgoing calls.
6. Use the textual call tree, namespace summary, and Projects List to understand the structure of the selected call chain.
7. Adjust **Call depth** to control how far the exploration extends.

The viewer is a standalone HTML file. The graph visualization may load D3.js through a CDN, in which case an internet connection is needed unless the library is available locally.

## Important Notes and Limitations

### Static Analysis vs. Runtime Execution

The call graph represents relationships detected through static analysis. It does not necessarily describe the exact sequence of calls executed at runtime.

Conditional branches, loops, dependency injection, interface dispatch, delegates, reflection, asynchronous execution, and dynamically loaded code can affect the actual runtime behavior.

### Call Tree and Summaries

The textual call tree, namespace summary, and project list are derived from the available graph nodes and `calls` edges.

Their contents depend on the graph generated by the builder and the selected depth. A missing relationship in the generated graph cannot be recovered by the viewer alone.

### Namespace and Project Metadata

Namespace and project labels depend on the metadata available in the graph. Some symbols may not have all the information required for a fully qualified display.

### Large Solutions

Deep call graphs can become large. Use the depth selector to keep the visualization and textual tree manageable.

## Recommended `.gitignore`

Generated graph files can be large and are reproducible. Consider excluding them from repositories that consume GraphTools:

```gitignore
full-graph.json
project-dependencies.json
```

## Credits

This project is based on the original [GraphTools repository](https://github.com/citsuresh/GraphTools).

This fork retains the original project's core graph-generation and query capabilities while extending the viewer for recursive method exploration and codebase architecture discovery.

Please refer to the upstream repository for the original implementation, documentation, and licensing information.

## License

See the repository's `LICENSE` file for the applicable license terms.
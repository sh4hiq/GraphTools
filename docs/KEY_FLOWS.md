# Key Flows

- **Full build**: `Builder.Program` -> `WorkspaceLoader.LoadSolutionAsync` -> `GraphExtractor.ExtractProjectAsync` (per project) -> `JsonHelper.WriteToFile` (full-graph.json) -> `ProjectDependencyExtractor.Extract` -> `JsonHelper.WriteToFile` (project-dependencies.json).
- **Incremental build**: `Builder.Program` -> load existing graph -> detect changed files (mtime > `GeneratedAt`, excluding `PathUtils.IsInBuildOutputFolder`) -> re-extract only affected projects -> merge nodes/edges by file path -> write merged graph.
- **Diff**: `Builder.Program --diff` -> load two `FullGraph` files -> compare `Nodes`/`Edges` by id/key -> produce `DiffReport` (added/removed/modified).
- **Query**: `Query.Program` -> `JsonHelper.ReadFromFile<FullGraph>` -> filter `Nodes`/`Edges` by symbol id, direction, or project -> print JSON result.

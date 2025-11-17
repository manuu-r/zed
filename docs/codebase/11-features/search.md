# Project Search

**Last Updated:** 2025-11-16

---

## Overview

Project-wide search and replace.

**Path:** `/home/user/zed/crates/search/`

## Search UI

```rust
pub struct ProjectSearch {
    query: String,
    results: Vec<SearchResult>,
    project: Entity<Project>,
}
```

## Search Execution

```rust
let results = project.search(SearchQuery {
    query: "TODO".to_string(),
    case_sensitive: false,
    whole_word: false,
    regex: false,
    files: None, // Search all files
}, cx).await?;
```

## Results Display

```rust
impl Render for ProjectSearchView {
    fn render(&mut self, window: &mut Window, cx: &mut Context<Self>) -> impl IntoElement {
        List::new(self.results.clone(), |result, cx| {
            SearchResultItem::new(result)
                .on_click(cx.listener(|this, result, cx| {
                    this.navigate_to_result(result, cx);
                }))
        })
    }
}
```

## Further Reading

- [Project Search](../03-project/search.md)
- [Workspace](../04-workspace/README.md)

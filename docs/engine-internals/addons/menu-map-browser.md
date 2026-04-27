---
title: Menu Map Browser
icon: "🗺️"
sources:
  - game/addons/menu/Code/MenuUI/MapsPage.razor
created: 2026-04-27
updated: 2026-04-27
---

# Menu Map Browser

The Menu Map Browser handles querying and displaying the list of maps available to the player within the Main Menu.

## Quick Working Example

The map browser uses the engine's built-in `PackageList` UI component to fetch and display maps asynchronously based on search filters.

```html
<!-- Inside MapsPage.razor -->
<PackageList 
    @ref=PackageList 
    ShowFilters="@false" 
    Query=@($"type:map sort:{filterOrder} {GetQuery()}") 
    OnSelected="@OnPackageSelected" 
    Take=@(200)>
</PackageList>

@code {
    string filterOrder = "trending";
    TextEntry SearchEntry = default;
    List<string> SelectedTags = new();
    
    // Builds the dynamic query string based on user input
    string GetQuery()
    {
        string tagString = SearchEntry?.Text ?? "";
        foreach (var tag in SelectedTags)
        {
            tagString += $" +{tag}";
        }
        return tagString;
    }
}
```

## Querying and Filtering

The `MapsPage` dynamically builds a query string based on user input:
-   **Text Search:** Added directly from the `TextEntry` component.
-   **Tags & Categories:** Filter selections (e.g., specific game modes or visual styles) are appended to the query using the `+{tag}` syntax.
-   **Sort Order:** Controls whether the maps are sorted by "trending", "newest", etc., by prepending `sort:{filterOrder}`.

This combined query is passed to the `PackageList`, which automatically handles the asynchronous fetching and updating of the UI when the query string changes.


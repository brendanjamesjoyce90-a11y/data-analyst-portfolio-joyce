## Exemplary Code: UAV Gradient Map

**What this chunk does:**

This code transforms raw COW panel data into a global gradient map showing when each country adopted armed UAVs. It filters for UAV adopters, finds each state's first adoption year, joins that to world map polygons via a custom country mapping table, then renders a choropleth with a blue-to-red gradient (early to recent adopters). The result is a publication-ready map that shows diffusion patterns at a glance — no intermediate files, just one pipeline from CSV to visualization.

**Why it's worth highlighting:**

The key challenge is joining COW state names (e.g., "United States of America") to map region names (e.g., "USA"). The `country_mapping` table handles this explicitly rather than relying on fuzzy matching. Once joined, `scale_fill_gradientn()` applies a colorblind-friendly diverging palette, and `coord_fixed()` excludes Antarctica while preserving aspect ratio. The whole thing is self-contained and reproducible.

```{r}

# Extract UAV adoption years
uav_adoption <- cow_long |>
  filter(techtype == "Armed UAVs", !is.na(use), use %in% c(1, 9)) |>
  group_by(statename) |>
  summarise(adoption_year = min(year), .groups = "drop")

# Map COW state names to map region names
country_mapping <- tribble(
  ~statename, ~region,
  "United States of America", "USA",
  "United Kingdom", "UK",
  "Russia", "Russia",
  "China", "China",
  # ... (full mapping table in actual code)
)

# Join adoption data to world map
uav_map_data <- uav_adoption_all |>
  left_join(country_mapping, by = "statename") |>
  filter(!is.na(region))

world_uav <- world_map |>
  left_join(uav_map_data, by = "region")

# Render gradient map
ggplot(world_uav, aes(x = long, y = lat, group = group)) +
  geom_polygon(aes(fill = adoption_year), color = "grey30", linewidth = 0.1) +
  scale_fill_gradientn(
    colors = c("#2166ac", "#4393c3", "#92c5de", "#d1e5f0", "#fddbc7", "#f4a582", "#d6604d", "#b2182b"),
    na.value = "grey20",
    name = "Adoption Year",
    breaks = seq(1990, 2023, by = 5),
    labels = seq(1990, 2023, by = 5),
    limits = c(1990, 2023)
  ) +
  coord_fixed(1.3, ylim = c(-55, 83), expand = FALSE) +
  labs(
    title = "UAV Diffusion Eminates from the Middle East (Iran) and Asia (China)",
    caption = "Source: COW Arms Technology Dataset v1.0 (2025). Blue = early adopters, Red = recent adopters."
  )… (full ggplot in code)

```
![Gradient output](gradient.PNG)

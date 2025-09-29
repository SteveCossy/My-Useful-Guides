In a Google Sheet I have a graph with people's names on the horizontal axis. How can I add a photo of each person to the left of their name?

Unfortunately, directly embedding images **into the axis labels** of a Google Sheet chart is not a built-in feature and isn't straightforward or practically achievable using standard charting tools within Google Sheets.

Google Sheets charts are designed to display data visually, and while they offer extensive customization for text labels, colors, and series, they don't support rich media like images within the axis text itself.

**However, there are several workarounds, depending on your goal:**

**Option 1: Create a Dashboard/Report-style View (Recommended for visual impact)**

This is the most effective way to combine photos with chart data in a visually appealing manner. Instead of trying to put photos *on* the chart axis, you create a separate section next to the chart.

1.  **Prepare your data:**
    *   Have a column for names.
    *   Have a column for the URL of each person's photo (if photos are online) or leave space if you'll manually insert.

2.  **Insert the photos:**
    *   **From a URL:** Use the `IMAGE()` function in an empty cell next to the name.
        ```excel
        =IMAGE("https://example.com/person1.jpg")
        ```
    *   **Manually:** Go to `Insert > Image > Image in cell` (or `Image over cells` if you prefer). Adjust cell sizes to fit.

3.  **Arrange on your sheet:**
    *   Place your chart prominently.
    *   Create a section to the left (or right, or below) of the chart where you list the names.
    *   In a cell next to each name, insert the corresponding person's photo.
    *   You can use conditional formatting to highlight the row corresponding to the chart's current selection, if applicable.

    *Example Layout:*
    ```
    +--------------------------------+
    |  Photo  | Name  |  [Chart Here] |
    |---------|-------|---------------|
    |  [Img1] | Alice |               |
    |  [Img2] | Bob   |               |
    |  [Img3] | Carol |               |
    +--------------------------------+
    ```
    

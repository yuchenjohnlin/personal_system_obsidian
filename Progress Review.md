```dataviewjs
// Adjust "daily" if your folder has a different name
const pages = dv.pages('"daily"')
    .sort(p => p.file.name, 'desc')
    .limit(10);

function extractSection(text, headerName) {
    const lines = text.split("\n");
    let capture = false;
    let result = [];

    for (let line of lines) {
        // Start at: ### TO DO
        if (line.trim().startsWith(`### ${headerName}`)) {
            capture = true;
            continue;
        }
        // Stop when the next ### heading appears
        if (capture && line.trim().startsWith("### ")) {
            break;
        }
        if (capture) {
            result.push(line);
        }
    }
    return result.join("\n").trim();
}

for (let page of pages) {
    const content = await dv.io.load(page.file.path);
    const todo = extractSection(content, "TO DO");

    if (!todo) continue; // skip days with no TO DO section

    // Clickable date title
    dv.header(3, `[${page.file.name}](${page.file.path})`);

    // Print the TO DO section as markdown
    dv.paragraph(todo);
}
```




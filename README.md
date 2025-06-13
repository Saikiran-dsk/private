<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>🧩 Dynamic Category Manager</title>
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet" />
  <link href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.10.5/font/bootstrap-icons.css" rel="stylesheet" />
  <style>
    :root {
      --gap: 0.5rem;
      --bg: #f8f9fa;
      --bg-card: #fff;
      --br: 0.6rem;
    }

    body {
      background-color: var(--bg);
      font-family: 'Segoe UI', sans-serif;
      padding: 2rem;
    }

    .card {
      border-radius: var(--br);
      background-color: var(--bg-card);
      padding: 1rem;
      margin-bottom: 2rem;
      box-shadow: 0 0.5rem 1rem rgba(0,0,0,0.05);
    }

    .category-grid {
      display: grid;
      grid-auto-flow: row;
      row-gap: var(--gap);
    }

    .category-grid .header, .category-grid .row {
      display: grid;
      grid-auto-columns: 1fr;
      align-items: center;
      column-gap: var(--gap);
    }

    .category-grid .header {
      font-weight: bold;
    }

    .category-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 1rem;
    }

    .input-wrap {
      display: flex;
      align-items: center;
    }

    .input-wrap input {
      flex-grow: 1;
    }

    .input-wrap .btn {
      margin-left: 4px;
    }

    .row-action {
      display: flex;
      align-items: center;
      justify-content: flex-start;
    }

    .small-btn {
      padding: 0.25rem 0.5rem;
      font-size: 0.85rem;
    }

    @media (max-width: 768px) {
      .category-grid .header,
      .category-grid .row {
        display: block;
      }
    }
	.add-cat-btn{
	 min-width:160px;
	}
	
  </style>
</head>
<body>

<div class="container">
  <h3 class="mb-4 text-primary text-center">🧩 Dynamic Category Manager</h3>

  <div class="d-flex gap-2 mb-3">
    <input type="text" id="newCategoryInput" class="form-control" placeholder="Category Name (e.g. QA)" />
    <input type="text" id="newFieldsInput" class="form-control" placeholder="Fields (comma-separated)" />
    <button class="btn btn-outline-primary add-cat-btn" onclick="addCategory()">➕ Add Category</button>
  </div>

  <div id="categoriesContainer"></div>

  <h6 class="text-muted mt-4">🧾 Local Storage:</h6>
  <pre id="jsonOutput"></pre>
</div>

<script>
  let projectData = JSON.parse(localStorage.getItem("projectData")) || {};
  const container = document.getElementById("categoriesContainer");
  const jsonOutput = document.getElementById("jsonOutput");

  function saveData() {
    localStorage.setItem("projectData", JSON.stringify(projectData, null, 2));
    jsonOutput.textContent = JSON.stringify(projectData, null, 2);
  }

  function createRow(category, fields, data = {}) {
    const row = document.createElement("div");
    row.className = "row";
    row.style.gridTemplateColumns = `repeat(${fields.length + 1}, 1fr)`;

    fields.forEach(field => {
      const inputWrap = document.createElement("div");
      inputWrap.className = "input-wrap";

      const input = document.createElement("input");
      input.className = "form-control";
      input.placeholder = field;
      input.value = data[field] || "";
      input.setAttribute("data-field", field);
      input.addEventListener("input", () => updateCategoryData(category));

      inputWrap.appendChild(input);

      if (field.toLowerCase() === "url") {
        const openBtn = document.createElement("button");
        openBtn.className = "btn btn-outline-secondary small-btn";
        openBtn.title = "Open URL";
        openBtn.innerHTML = '<i class="bi bi-box-arrow-up-right"></i>';
        openBtn.onclick = () => {
          let val = input.value;
          if (!val.startsWith("http")) val = "https://" + val;
          window.open(val, "_blank");
        };
        inputWrap.appendChild(openBtn);
      }

      const copyBtn = document.createElement("button");
      copyBtn.className = "btn btn-outline-secondary small-btn";
      copyBtn.title = "Copy";
      copyBtn.innerHTML = '<i class="bi bi-clipboard"></i>';
      copyBtn.onclick = () => {
        navigator.clipboard.writeText(input.value).then(() => {
          copyBtn.innerHTML = '<i class="bi bi-clipboard-check text-success"></i>';
          setTimeout(() => copyBtn.innerHTML = '<i class="bi bi-clipboard"></i>', 1200);
        });
      };
      inputWrap.appendChild(copyBtn);

      row.appendChild(inputWrap);
    });

    const actionCol = document.createElement("div");
    actionCol.className = "row-action";
    const delBtn = document.createElement("button");
    delBtn.className = "btn btn-outline-danger small-btn";
    delBtn.innerHTML = '<i class="bi bi-trash"></i>';
    delBtn.title = "Delete Row";
    delBtn.onclick = () => {
      row.remove();
      updateCategoryData(category);
    };
    actionCol.appendChild(delBtn);
    row.appendChild(actionCol);

    return row;
  }

  function updateCategoryData(category) {
    const section = document.querySelector(`[data-category="${category}"]`);
    const fields = section.getAttribute("data-fields").split(",");
    const rows = section.querySelectorAll(".row:not(.header)");
    const result = [];

    rows.forEach(row => {
      const inputs = row.querySelectorAll("input");
      const obj = {};
      inputs.forEach(input => {
        const field = input.getAttribute("data-field");
        obj[field] = input.value;
      });
      result.push(obj);
    });

    projectData[category].data = result;
    saveData();
  }

  function renderCategory(category, fields, data = []) {
    const wrapper = document.createElement("div");
    wrapper.className = "card category-grid";
    wrapper.setAttribute("data-category", category);
    wrapper.setAttribute("data-fields", fields.join(","));

    const head = document.createElement("div");
    head.className = "category-header";
    head.innerHTML = `<h5>📁 ${category}</h5><button class="btn btn-sm btn-outline-danger" onclick="deleteCategory('${category}')">❌ Delete</button>`;
    wrapper.appendChild(head);

    const headerRow = document.createElement("div");
    headerRow.className = "header row";
    headerRow.style.gridTemplateColumns = `repeat(${fields.length + 1}, 1fr)`;

    fields.forEach(field => {
      const col = document.createElement("div");
      col.textContent = field.toUpperCase();
      headerRow.appendChild(col);
    });
    const actionLabel = document.createElement("div");
    actionLabel.textContent = "Actions";
    headerRow.appendChild(actionLabel);

    wrapper.appendChild(headerRow);

    data.forEach(entry => {
      wrapper.appendChild(createRow(category, fields, entry));
    });

    const addBtn = document.createElement("button");
    addBtn.className = "btn btn-outline-success w-100 mt-3";
    addBtn.textContent = "➕ Add Row";
    addBtn.onclick = () => {
      wrapper.insertBefore(createRow(category, fields), addBtn);
      updateCategoryData(category);
    };

    wrapper.appendChild(addBtn);
    container.appendChild(wrapper);
  }

  function deleteCategory(category) {
    if (confirm(`Delete category "${category}"?`)) {
      delete projectData[category];
      document.querySelector(`[data-category="${category}"]`).remove();
      saveData();
    }
  }

  function addCategory() {
    const name = document.getElementById("newCategoryInput").value.trim();
    const fields = document.getElementById("newFieldsInput").value.trim().split(',').map(f => f.trim()).filter(Boolean);

    if (!name || fields.length === 0) {
      alert("Please enter both category name and fields.");
      return;
    }

    if (projectData[name]) {
      alert("Category already exists.");
      return;
    }

    projectData[name] = { fields, data: [] };
    renderCategory(name, fields);
    saveData();

    document.getElementById("newCategoryInput").value = '';
    document.getElementById("newFieldsInput").value = '';
  }

  function init() {
    for (const category in projectData) {
      const { fields, data } = projectData[category];
      renderCategory(category, fields, data);
    }
    saveData();
  }

  init();
</script>

</body>
</html>

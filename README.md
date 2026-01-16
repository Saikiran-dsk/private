Get-MgOrganization | Select-Object DisplayName, VerifiedDomains



$ClientId     = "<APP-ID>"
$TenantDomain = "<tenant>.onmicrosoft.com"
$PfxPath      = "C:\certs\exo-app.pfx"
$PfxPassword  = ConvertTo-SecureString "PfxPasswordHere" -AsPlainText -Force

Connect-ExchangeOnline `
  -AppId $ClientId `
  -Organization $TenantDomain `
  -CertificateFilePath $PfxPath `
  -CertificatePassword $PfxPassword `
  -CommandName Get-OrganizationConfig `
  -ShowBanner:$false



;;;;;;;;;;

$ClientId     = "<APP-ID>"
$TenantDomain = "<tenant>.onmicrosoft.com"

$PfxPath = "C:\certs\exo-app.pfx"
$PfxPassword = ConvertTo-SecureString "PfxPasswordHere" -AsPlainText -Force

Connect-ExchangeOnline `
  -AppId $ClientId `
  -Organization $TenantDomain `
  -CertificateFilePath $PfxPath `
  -CertificatePassword $PfxPassword `
  -CommandName Get-DistributionGroup `
  -ShowBanner:$false


Import-PfxCertificate `
  -FilePath "E:\exo-app.pfx" `
  -CertStoreLocation Cert:\CurrentUser\My `
  -Password (ConvertTo-SecureString "PfxPasswordHere" -AsPlainText -Force)

Connect-ExchangeOnline `
  -AppId $ClientId `
  -Organization $TenantDomain `
  -CertificateThumbprint "<THUMBPRINT>" `
  -CommandName Get-DistributionGroup `
  -ShowBanner:$false

Import-PfxCertificate `
  -FilePath "E:\exo-app.pfx" `
  -CertStoreLocation Cert:\CurrentUser\My `
  -Password (ConvertTo-SecureString "PfxPasswordHere" -AsPlainText -Force)


function Add-SharedMailboxToMailEnabledSecurityGroup {
    [CmdletBinding()]
    param (
        # Exchange Online App-only auth
        [Parameter(Mandatory)]
        [string]$ClientId,

        [Parameter(Mandatory)]
        [string]$TenantId,

        [Parameter(Mandatory)]
        [string]$CertificateThumbprint,

        # Objects
        [Parameter(Mandatory)]
        [string]$SharedMailboxDisplayName,

        [Parameter(Mandatory)]
        [string]$MailEnabledSecurityGroupDisplayName
    )

    try {
        # -----------------------------
        # Connect to Exchange Online
        # -----------------------------
        Import-Module ExchangeOnlineManagement -ErrorAction Stop

        Connect-ExchangeOnline `
            -AppId $ClientId `
            -Organization $TenantId `
            -CertificateThumbprint $CertificateThumbprint `
            -ShowBanner:$false

        Write-Verbose "Connected to Exchange Online"

        # -----------------------------
        # Validate Group
        # -----------------------------
        $group = Get-DistributionGroup `
            -Identity $MailEnabledSecurityGroupDisplayName `
            -ErrorAction SilentlyContinue

        if (-not $group) {
            throw "Mail-enabled security group '$MailEnabledSecurityGroupDisplayName' not found."
        }

        # -----------------------------
        # Validate Shared Mailbox
        # -----------------------------
        $mailbox = Get-Mailbox `
            -Identity $SharedMailboxDisplayName `
            -ErrorAction SilentlyContinue

        if (-not $mailbox) {
            throw "Shared mailbox '$SharedMailboxDisplayName' not found."
        }

        # -----------------------------
        # Check existing membership
        # -----------------------------
        $alreadyMember = Get-DistributionGroupMember `
            -Identity $MailEnabledSecurityGroupDisplayName |
            Where-Object {
                $_.PrimarySmtpAddress -eq $mailbox.PrimarySmtpAddress
            }

        if ($alreadyMember) {
            Write-Host "Shared mailbox already exists in the group." -ForegroundColor Yellow
            return
        }

        # -----------------------------
        # Add mailbox to group
        # -----------------------------
        Add-DistributionGroupMember `
            -Identity $MailEnabledSecurityGroupDisplayName `
            -Member $SharedMailboxDisplayName

        Write-Host "Shared mailbox added successfully." -ForegroundColor Green
    }
    catch {
        Write-Error $_
        throw
    }
    finally {
        # -----------------------------
        # Disconnect
        # -----------------------------
        Disconnect-ExchangeOnline -Confirm:$false
    }
}








<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Credential Manager</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <!-- Icons -->
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.0/css/all.min.css"/>

  <style>
    :root{
      --bg1:#f4f7fb;        /* light gradient */
      --bg2:#eaf1f9;
      --card:#ffffffee;     /* soft card */
      --muted:#e1e7f0;      /* borders */
      --muted-2:#f4f8ff;    /* header band */
      --ring:#1877f2;       /* focus */
      --text:#243447;       /* body text */
      --gap:6px;            /* compact gaps */
      --radius:12px;        /* rounded */
      --btn-radius:10px;
      --shadow:0 8px 28px rgba(0,0,0,.08);
    }

    /* Page + full use with small margins 10/20px */
    html, body { height:100%; }
    body{
      margin:0;
      background:linear-gradient(135deg,var(--bg1),var(--bg2));
      font-family: "Inter", system-ui, -apple-system, "Segoe UI", Roboto, Arial, sans-serif;
      color:var(--text);
      font-size:14px;
    }
    .wrap{
      width:calc(100% - 40px);
      margin:10px 20px 20px;
      background:var(--card);
      border-radius:16px;
      box-shadow:var(--shadow);
      padding:14px;
    }
    .title{
      margin:0 0 12px;
      text-align:center;
      font-weight:700;
      font-size:20px;
      color:#0d6efd;
      letter-spacing:.2px;
    }

    /* Toolbar: keep in one row; scroll if overflow */
    .toolbar{
      display:flex;
      align-items:center;
      gap:8px;
      white-space:nowrap;
      overflow-x:auto;
      padding-bottom:4px;
      -webkit-overflow-scrolling:touch;
    }
    .toolbar input{
      flex:1 1 280px;       /* grow to share space */
      min-width:240px;
    }

    /* Inputs */
    .input{
      border:1px solid #cfd6e0;
      background:#f9fbfe;
      color:var(--text);
      border-radius:10px;
      padding:6px 10px;
      font-size:13px;
      outline:none;
      transition:border-color .15s, box-shadow .15s, background .15s;
    }
    .input:focus{
      border-color:var(--ring);
      box-shadow:0 0 0 3px rgba(24,119,242,.15);
      background:#fff;
    }

    /* Buttons (consistent) */
    .btn{
      border:none;
      border-radius:var(--btn-radius);
      padding:7px 12px;
      font-size:12.5px;
      font-weight:600;
      display:inline-flex;
      align-items:center;
      gap:6px;
      cursor:pointer;
      transition:transform .12s ease, box-shadow .12s ease, background .12s ease, opacity .12s ease;
      user-select:none;
      -webkit-tap-highlight-color:transparent;
      white-space:nowrap;
    }
    .btn:hover{ transform:translateY(-1px); box-shadow:0 3px 12px rgba(0,0,0,.12); }
    .btn:active{ transform:translateY(0); }
    .btn-green{ background:#28a745; color:#fff; }
    .btn-green:hover{ background:#218838; }
    .btn-blue{ background:#1877f2; color:#fff; }
    .btn-blue:hover{ background:#0f66d6; }
    .btn-gray{ background:#6b7280; color:#fff; }
    .btn-gray:hover{ background:#5a6472; }
    .btn-red{ background:#e11d48; color:#fff; }
    .btn-red:hover{ background:#be123c; }
    .btn-ghost{ background:#f3f6fb; color:#374151; border:1px solid #cfd6e0; }
    .btn-ghost:hover{ background:#eaf0f8; }

    /* Category card */
    .cat{
      background:#fbfdff;
      border:1px solid var(--muted);
      border-radius:var(--radius);
      padding:10px;
      margin-top:12px;
    }
    .cat-head{
      display:flex;
      align-items:center;
      justify-content:space-between;
      gap:10px;
      margin-bottom:8px;
    }
    .cat-title{
      display:flex;
      align-items:center;
      gap:8px;
      margin:0;
      font-weight:600;
      font-size:15px;
      color:#0d6efd;
    }
    .head-actions{ display:flex; gap:6px; }

    /* Grid (perfect alignment, minimal spacing) */
    .grid{ display:grid; grid-auto-flow:row; row-gap:var(--gap); }
    .grid-header, .grid-row{
      display:grid;
      align-items:center;
      column-gap:var(--gap);
    }
    .grid-header{
      background:var(--muted-2);
      border:1px solid var(--muted);
      border-radius:10px;
      padding:6px;
      font-weight:600;
      font-size:12.5px;
    }
    .grid-row{ padding:0; margin-top: 8px;
    margin-bottom: 8px;}

    .cell{
      display:flex;
      align-items:center;
      gap:4px;            /* << minimal per-column spacing */
      min-width:0;        /* allow shrinking */
    }
    .cell .input{ padding:5px 8px; font-size:12.5px; width:100%; }

    .row-actions{ display:flex; align-items:center; gap:6px; justify-content:flex-start; }

    /* Feedback flashes */
    .flash-success{ animation:flashGreen .45s; }
    .flash-danger{ animation:flashRed .45s; }
    @keyframes flashGreen{ from{ background:#d1e7dd; } to{ background:transparent; } }
    @keyframes flashRed{ from{ background:#fde2e7; } to{ background:transparent; } }

    /* Collapse */
    .collapsed .grid-header,
    .collapsed .grid-row,
    .collapsed .add-row-wrap{ display:none; }

    /* Responsive: keep columns on one row for desktops; on very small screens allow wrap by switching to block */
    @media (max-width: 840px){
      .grid-header, .grid-row { display:block; }
      .row-actions{ margin-top:6px; }
    }
    .add-row-btn{
        display: flex;
    justify-content: center;
    }
    .grid-header{
            margin-top: 10px;
    margin-bottom: 10px;
    }
  </style>
</head>
<body>
  <div class="wrap">
    <h2 class="title"><i class="fa-solid fa-vault"></i> Credential Manager</h2>

    <!-- Top bar: all controls in ONE row -->
    <div class="toolbar">
      <input id="newCategoryInput" class="input" placeholder="Category name (e.g. QA)" />
      <input id="newFieldsInput" class="input" placeholder="Column names (comma-separated: Name, URL, Username, Password)" />
      <button id="addCatBtn" class="btn btn-green"><i class="fa-solid fa-plus"></i> Add Category</button>
      <button id="exportBtn" class="btn btn-blue"><i class="fa-solid fa-file-arrow-down"></i> Export</button>
      <input type="file" id="importFile" class="d-none" accept="application/json" style="display:none" />
      <button id="importBtn" class="btn btn-gray"><i class="fa-solid fa-file-arrow-up"></i> Import</button>
    </div>

    <!-- Categories render here -->
    <div id="categoriesContainer"></div>
  </div>

<script>
/* ===================== STATE ===================== */
let projectData = JSON.parse(localStorage.getItem("projectData")) || {};
const container = document.getElementById("categoriesContainer");

/* ===================== HELPERS ===================== */
const saveData = () => localStorage.setItem("projectData", JSON.stringify(projectData, null, 2));
const toCols = s => s.split(",").map(x => x.trim()).filter(Boolean);
function flash(el, ok=true){
  el.classList.add(ok ? 'flash-success' : 'flash-danger');
  setTimeout(()=>el.classList.remove(ok ? 'flash-success' : 'flash-danger'), 450);
}

/* ===================== BUILDERS ===================== */
function buildRow(category, fields, data = {}){
  const row = document.createElement("div");
  row.className = "grid-row";
  row.style.gridTemplateColumns = `repeat(${fields.length}, minmax(0, 1fr)) 140px`; // fields + actions

  fields.forEach(field => {
    const lower = field.toLowerCase();
    const cell = document.createElement("div");
    cell.className = "cell";

    const input = document.createElement("input");
    input.className = "input";
    input.placeholder = field;
    input.value = data[field] || "";
    input.setAttribute("data-field", field);
    input.addEventListener("input", () => updateCategoryData(category));
    if (lower === "password") input.type = "password";
    cell.appendChild(input);

    // URL open button
    if (lower === "url") {
      const openBtn = document.createElement("button");
      openBtn.className = "btn btn-ghost";
      openBtn.title = "Open link";
      openBtn.innerHTML = `<i class="fa-solid fa-arrow-up-right-from-square"></i>`;
      openBtn.addEventListener("click", () => {
        let val = (input.value || "").trim();
        if (!val) return;
        if (!/^https?:\/\//i.test(val)) val = "https://" + val;
        window.open(val, "_blank");
      });
      cell.appendChild(openBtn);
    }

    // Password toggle
    if (lower === "password") {
      const toggleBtn = document.createElement("button");
      toggleBtn.className = "btn btn-ghost";
      toggleBtn.title = "Show / Hide";
      toggleBtn.innerHTML = `<i class="fa-regular fa-eye"></i>`;
      toggleBtn.addEventListener("click", () => {
        const isPwd = input.type === "password";
        input.type = isPwd ? "text" : "password";
        toggleBtn.innerHTML = isPwd
          ? `<i class="fa-regular fa-eye-slash"></i>`
          : `<i class="fa-regular fa-eye"></i>`;
      });
      cell.appendChild(toggleBtn);
    }

    // Copy button for every field
    const copyBtn = document.createElement("button");
    copyBtn.className = "btn btn-ghost";
    copyBtn.title = "Copy value";
    copyBtn.innerHTML = `<i class="fa-regular fa-copy"></i>`;
    copyBtn.addEventListener("click", async () => {
      try{
        await navigator.clipboard.writeText(input.value || "");
        copyBtn.innerHTML = `<i class="fa-solid fa-check"></i>`;
        flash(copyBtn, true);
        setTimeout(()=> copyBtn.innerHTML = `<i class="fa-regular fa-copy"></i>`, 900);
      }catch{
        flash(copyBtn, false);
        alert("Copy failed");
      }
    });
    cell.appendChild(copyBtn);

    row.appendChild(cell);
  });

  // actions column
  const actions = document.createElement("div");
  actions.className = "row-actions";

  const delBtn = document.createElement("button");
  delBtn.className = "btn btn-red";
  delBtn.innerHTML = `<i class="fa-solid fa-trash"></i> Delete`;
  delBtn.addEventListener("click", () => {
    if (!confirm("Delete this row?")) return;
    row.remove();
    updateCategoryData(category);
    flash(delBtn, false);
  });
  actions.appendChild(delBtn);

  row.appendChild(actions);
  return row;
}

function updateCategoryData(category){
  const section = document.querySelector(`[data-category="${CSS.escape(category)}"]`);
  if (!section) return;
  const fields = section.getAttribute("data-fields").split(",");
  const rows = section.querySelectorAll(".grid-row");
  const data = [];
  rows.forEach(r=>{
    const obj = {};
    fields.forEach(f=>{
      const inp = r.querySelector(`input[data-field="${CSS.escape(f)}"]`);
      if (inp) obj[f] = inp.value;
    });
    if (Object.keys(obj).length) data.push(obj);
  });
  projectData[category].data = data;
  saveData();
}

function renderCategory(category, fields, data=[]){
  const card = document.createElement("div");
  card.className = "cat";
  card.setAttribute("data-category", category);
  card.setAttribute("data-fields", fields.join(","));

  // Header
  const head = document.createElement("div");
  head.className = "cat-head";

  const title = document.createElement("h3");
  title.className = "cat-title";
  title.innerHTML = `<i class="fa-solid fa-folder"></i> ${category}`;
  head.appendChild(title);

  const headBtns = document.createElement("div");
  headBtns.className = "head-actions";

  const collapseBtn = document.createElement("button");
  collapseBtn.className = "btn btn-ghost";
  collapseBtn.title = "Collapse / Expand";
  collapseBtn.innerHTML = `<i class="fa-solid fa-chevron-up"></i>`;
  collapseBtn.addEventListener("click", ()=>{
    card.classList.toggle("collapsed");
    collapseBtn.innerHTML = card.classList.contains("collapsed")
      ? `<i class="fa-solid fa-chevron-down"></i>`
      : `<i class="fa-solid fa-chevron-up"></i>`;
  });
  headBtns.appendChild(collapseBtn);

  const renameBtn = document.createElement("button");
  renameBtn.className = "btn btn-blue";
  renameBtn.innerHTML = `<i class="fa-solid fa-pen"></i> Rename`;
  renameBtn.addEventListener("click", ()=>{
    const newName = prompt("Enter new category name:", category);
    if (!newName || newName === category) return;
    if (projectData[newName]) return alert("A category with that name already exists.");
    projectData[newName] = projectData[category];
    delete projectData[category];
    saveData();
    renderAll();
  });
  headBtns.appendChild(renameBtn);

  const editColsBtn = document.createElement("button");
  editColsBtn.className = "btn btn-gray";
  editColsBtn.innerHTML = `<i class="fa-solid fa-columns"></i> Edit Columns`;
  editColsBtn.addEventListener("click", ()=>{
    const newFieldsStr = prompt("Enter new column names (comma-separated):", fields.join(", "));
    if (!newFieldsStr) return;
    const newFields = toCols(newFieldsStr);
    if (newFields.length === 0) return alert("Please enter at least one column name.");

    // Update fields in projectData
    projectData[category].fields = newFields;

    // Create a new data array with updated field names
    const oldFields = fields;
    const newData = projectData[category].data.map(oldRow => {
      const newRow = {};
      for (let i = 0; i < newFields.length; i++) {
        const newField = newFields[i];
        const oldField = oldFields[i];
        newRow[newField] = oldRow[oldField] || "";
      }
      return newRow;
    });

    projectData[category].data = newData;
    saveData();
    renderAll();
  });
  headBtns.appendChild(editColsBtn);

  const delCatBtn = document.createElement("button");
  delCatBtn.className = "btn btn-red";
  delCatBtn.innerHTML = `<i class="fa-solid fa-trash"></i> Delete`;
  delCatBtn.addEventListener("click", ()=>{
    if (!confirm(`Delete category "${category}"?`)) return;
    delete projectData[category];
    saveData();
    renderAll();
  });
  headBtns.appendChild(delCatBtn);

  head.appendChild(headBtns);
  card.appendChild(head);

  // Grid header
  const header = document.createElement("div");
  header.className = "grid-header";
  header.style.gridTemplateColumns = `repeat(${fields.length}, minmax(0, 1fr)) 140px`;
  fields.forEach(f=>{
    const d = document.createElement("div");
    d.textContent = f.toUpperCase();
    header.appendChild(d);
  });
  const actLab = document.createElement("div");
  actLab.textContent = "ACTIONS";
  header.appendChild(actLab);
  card.appendChild(header);

  // rows
  data.forEach(entry => {
    card.appendChild(buildRow(category, fields, entry));
  });

  // add row
  const addWrap = document.createElement("div");
  addWrap.className = "add-row-wrap";
  const addBtn = document.createElement("button");
  addBtn.className = "btn btn-green add-row-btn";
  addBtn.style.width = "100%";
  addBtn.innerHTML = `<i class="fa-solid fa-plus"></i> Add Row`;
  addBtn.addEventListener("click", ()=>{
    const r = buildRow(category, fields);
    card.insertBefore(r, addWrap);
    updateCategoryData(category);
  });
  addWrap.appendChild(addBtn);
  card.appendChild(addWrap);

  container.appendChild(card);
}

function renderAll(){
  container.innerHTML = "";
  for (const cat in projectData){
    const { fields, data } = projectData[cat];
    renderCategory(cat, fields, data);
  }
}

/* ===================== TOP BAR ===================== */
document.getElementById("addCatBtn").addEventListener("click", ()=>{
  const name = document.getElementById("newCategoryInput").value.trim();
  const fields = toCols(document.getElementById("newFieldsInput").value.trim());
  if (!name || fields.length === 0){
    alert("Please enter both category name and column names (comma-separated).");
    return;
  }
  if (projectData[name]){
    alert("Category already exists.");
    return;
  }
  projectData[name] = { fields, data: [] };
  saveData();
  renderAll();
  document.getElementById("newCategoryInput").value = "";
  document.getElementById("newFieldsInput").value = "";
});

document.getElementById("exportBtn").addEventListener("click", ()=>{
  const blob = new Blob([JSON.stringify(projectData, null, 2)], {type:"application/json"});
  const a = document.createElement("a");
  a.href = URL.createObjectURL(blob);
  a.download = "credentials.json";
  a.click();
});

document.getElementById("importBtn").addEventListener("click", ()=>{
  document.getElementById("importFile").click();
});
document.getElementById("importFile").addEventListener("change", (e)=>{
  const file = e.target.files[0];
  if (!file) return;
  const reader = new FileReader();
  reader.onload = ev => {
    try{
      const data = JSON.parse(ev.target.result);
      if (!data || typeof data !== "object" || Array.isArray(data)) throw new Error("Invalid shape");
      projectData = data;
      saveData();
      renderAll();
    }catch(err){
      alert("Invalid JSON file.");
    }
  };
  reader.readAsText(file);
});

/* ===================== INIT ===================== */
renderAll();
</script>
</body>
</html>
------------------



" Always start in Insert mode
autocmd VimEnter * startinsert
autocmd BufEnter * startinsert

" Pass system shortcuts to IntelliJ
map <C-c> <Action>(EditorCopy)
map <C-x> <Action>(EditorCut)
map <C-v> <Action>(EditorPaste)
map <C-z> <Action>(Undo)
map <C-y> <Action>(Redo)


------------

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Credential Manager</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <!-- Icons -->
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.0/css/all.min.css"/>

  <style>
    :root{
      --bg1:#f4f7fb;        /* light gradient */
      --bg2:#eaf1f9;
      --card:#ffffffee;     /* soft card */
      --muted:#e1e7f0;      /* borders */
      --muted-2:#f4f8ff;    /* header band */
      --ring:#1877f2;       /* focus */
      --text:#243447;       /* body text */
      --gap:6px;            /* compact gaps */
      --radius:12px;        /* rounded */
      --btn-radius:10px;
      --shadow:0 8px 28px rgba(0,0,0,.08);
    }

    /* Page + full use with small margins 10/20px */
    html, body { height:100%; }
    body{
      margin:0;
      background:linear-gradient(135deg,var(--bg1),var(--bg2));
      font-family: "Inter", system-ui, -apple-system, "Segoe UI", Roboto, Arial, sans-serif;
      color:var(--text);
      font-size:14px;
    }
    .wrap{
      width:calc(100% - 40px);
      margin:10px 20px 20px;
      background:var(--card);
      border-radius:16px;
      box-shadow:var(--shadow);
      padding:14px;
    }
    .title{
      margin:0 0 12px;
      text-align:center;
      font-weight:700;
      font-size:20px;
      color:#0d6efd;
      letter-spacing:.2px;
    }

    /* Toolbar: keep in one row; scroll if overflow */
    .toolbar{
      display:flex;
      align-items:center;
      gap:8px;
      white-space:nowrap;
      overflow-x:auto;
      padding-bottom:4px;
      -webkit-overflow-scrolling:touch;
    }
    .toolbar input{
      flex:1 1 280px;       /* grow to share space */
      min-width:240px;
    }

    /* Inputs */
    .input{
      border:1px solid #cfd6e0;
      background:#f9fbfe;
      color:var(--text);
      border-radius:10px;
      padding:6px 10px;
      font-size:13px;
      outline:none;
      transition:border-color .15s, box-shadow .15s, background .15s;
    }
    .input:focus{
      border-color:var(--ring);
      box-shadow:0 0 0 3px rgba(24,119,242,.15);
      background:#fff;
    }

    /* Buttons (consistent) */
    .btn{
      border:none;
      border-radius:var(--btn-radius);
      padding:7px 12px;
      font-size:12.5px;
      font-weight:600;
      display:inline-flex;
      align-items:center;
      gap:6px;
      cursor:pointer;
      transition:transform .12s ease, box-shadow .12s ease, background .12s ease, opacity .12s ease;
      user-select:none;
      -webkit-tap-highlight-color:transparent;
      white-space:nowrap;
    }
    .btn:hover{ transform:translateY(-1px); box-shadow:0 3px 12px rgba(0,0,0,.12); }
    .btn:active{ transform:translateY(0); }
    .btn-green{ background:#28a745; color:#fff; }
    .btn-green:hover{ background:#218838; }
    .btn-blue{ background:#1877f2; color:#fff; }
    .btn-blue:hover{ background:#0f66d6; }
    .btn-gray{ background:#6b7280; color:#fff; }
    .btn-gray:hover{ background:#5a6472; }
    .btn-red{ background:#e11d48; color:#fff; }
    .btn-red:hover{ background:#be123c; }
    .btn-ghost{ background:#f3f6fb; color:#374151; border:1px solid #cfd6e0; }
    .btn-ghost:hover{ background:#eaf0f8; }

    /* Category card */
    .cat{
      background:#fbfdff;
      border:1px solid var(--muted);
      border-radius:var(--radius);
      padding:10px;
      margin-top:12px;
    }
    .cat-head{
      display:flex;
      align-items:center;
      justify-content:space-between;
      gap:10px;
      margin-bottom:8px;
    }
    .cat-title{
      display:flex;
      align-items:center;
      gap:8px;
      margin:0;
      font-weight:600;
      font-size:15px;
      color:#0d6efd;
    }
    .head-actions{ display:flex; gap:6px; }

    /* Grid (perfect alignment, minimal spacing) */
    .grid{ display:grid; grid-auto-flow:row; row-gap:var(--gap); }
    .grid-header, .grid-row{
      display:grid;
      align-items:center;
      column-gap:var(--gap);
    }
    .grid-header{
      background:var(--muted-2);
      border:1px solid var(--muted);
      border-radius:10px;
      padding:6px;
      font-weight:600;
      font-size:12.5px;
    }
    .grid-row{ padding:0; margin-top: 8px;
    margin-bottom: 8px;}

    .cell{
      display:flex;
      align-items:center;
      gap:4px;            /* << minimal per-column spacing */
      min-width:0;        /* allow shrinking */
    }
    .cell .input{ padding:5px 8px; font-size:12.5px; width:100%; }

    .row-actions{ display:flex; align-items:center; gap:6px; justify-content:flex-start; }

    /* Feedback flashes */
    .flash-success{ animation:flashGreen .45s; }
    .flash-danger{ animation:flashRed .45s; }
    @keyframes flashGreen{ from{ background:#d1e7dd; } to{ background:transparent; } }
    @keyframes flashRed{ from{ background:#fde2e7; } to{ background:transparent; } }

    /* Collapse */
    .collapsed .grid-header,
    .collapsed .grid-row,
    .collapsed .add-row-wrap{ display:none; }

    /* Responsive: keep columns on one row for desktops; on very small screens allow wrap by switching to block */
    @media (max-width: 840px){
      .grid-header, .grid-row { display:block; }
      .row-actions{ margin-top:6px; }
    }
    .add-row-btn{
        display: flex;
    justify-content: center;
    }
    .grid-header{
            margin-top: 10px;
    margin-bottom: 10px;
    }
  </style>
</head>
<body>
  <div class="wrap">
    <h2 class="title"><i class="fa-solid fa-vault"></i> Credential Manager</h2>

    <!-- Top bar: all controls in ONE row -->
    <div class="toolbar">
      <input id="newCategoryInput" class="input" placeholder="Category name (e.g. QA)" />
      <input id="newFieldsInput" class="input" placeholder="Column names (comma-separated: Name, URL, Username, Password)" />
      <button id="addCatBtn" class="btn btn-green"><i class="fa-solid fa-plus"></i> Add Category</button>
      <button id="exportBtn" class="btn btn-blue"><i class="fa-solid fa-file-arrow-down"></i> Export</button>
      <input type="file" id="importFile" class="d-none" accept="application/json" style="display:none" />
      <button id="importBtn" class="btn btn-gray"><i class="fa-solid fa-file-arrow-up"></i> Import</button>
    </div>

    <!-- Categories render here -->
    <div id="categoriesContainer"></div>
  </div>

<script>
/* ===================== STATE ===================== */
let projectData = JSON.parse(localStorage.getItem("projectData")) || {};
const container = document.getElementById("categoriesContainer");

/* ===================== HELPERS ===================== */
const saveData = () => localStorage.setItem("projectData", JSON.stringify(projectData, null, 2));
const toCols = s => s.split(",").map(x => x.trim()).filter(Boolean);
function flash(el, ok=true){
  el.classList.add(ok ? 'flash-success' : 'flash-danger');
  setTimeout(()=>el.classList.remove(ok ? 'flash-success' : 'flash-danger'), 450);
}

/* ===================== BUILDERS ===================== */
function buildRow(category, fields, data = {}){
  const row = document.createElement("div");
  row.className = "grid-row";
  row.style.gridTemplateColumns = `repeat(${fields.length}, minmax(0, 1fr)) 140px`; // fields + actions

  fields.forEach(field => {
    const lower = field.toLowerCase();
    const cell = document.createElement("div");
    cell.className = "cell";

    const input = document.createElement("input");
    input.className = "input";
    input.placeholder = field;
    input.value = data[field] || "";
    input.setAttribute("data-field", field);
    input.addEventListener("input", () => updateCategoryData(category));
    if (lower === "password") input.type = "password";
    cell.appendChild(input);

    // URL open button
    if (lower === "url") {
      const openBtn = document.createElement("button");
      openBtn.className = "btn btn-ghost";
      openBtn.title = "Open link";
      openBtn.innerHTML = `<i class="fa-solid fa-arrow-up-right-from-square"></i>`;
      openBtn.addEventListener("click", () => {
        let val = (input.value || "").trim();
        if (!val) return;
        if (!/^https?:\/\//i.test(val)) val = "https://" + val;
        window.open(val, "_blank");
      });
      cell.appendChild(openBtn);
    }

    // Password toggle
    if (lower === "password") {
      const toggleBtn = document.createElement("button");
      toggleBtn.className = "btn btn-ghost";
      toggleBtn.title = "Show / Hide";
      toggleBtn.innerHTML = `<i class="fa-regular fa-eye"></i>`;
      toggleBtn.addEventListener("click", () => {
        const isPwd = input.type === "password";
        input.type = isPwd ? "text" : "password";
        toggleBtn.innerHTML = isPwd
          ? `<i class="fa-regular fa-eye-slash"></i>`
          : `<i class="fa-regular fa-eye"></i>`;
      });
      cell.appendChild(toggleBtn);
    }

    // Copy button for every field
    const copyBtn = document.createElement("button");
    copyBtn.className = "btn btn-ghost";
    copyBtn.title = "Copy value";
    copyBtn.innerHTML = `<i class="fa-regular fa-copy"></i>`;
    copyBtn.addEventListener("click", async () => {
      try{
        await navigator.clipboard.writeText(input.value || "");
        copyBtn.innerHTML = `<i class="fa-solid fa-check"></i>`;
        flash(copyBtn, true);
        setTimeout(()=> copyBtn.innerHTML = `<i class="fa-regular fa-copy"></i>`, 900);
      }catch{
        flash(copyBtn, false);
        alert("Copy failed");
      }
    });
    cell.appendChild(copyBtn);

    row.appendChild(cell);
  });

  // actions column
  const actions = document.createElement("div");
  actions.className = "row-actions";

  const delBtn = document.createElement("button");
  delBtn.className = "btn btn-red";
  delBtn.innerHTML = `<i class="fa-solid fa-trash"></i> Delete`;
  delBtn.addEventListener("click", () => {
    if (!confirm("Delete this row?")) return;
    row.remove();
    updateCategoryData(category);
    flash(delBtn, false);
  });
  actions.appendChild(delBtn);

  row.appendChild(actions);
  return row;
}

function updateCategoryData(category){
  const section = document.querySelector(`[data-category="${CSS.escape(category)}"]`);
  if (!section) return;
  const fields = section.getAttribute("data-fields").split(",");
  const rows = section.querySelectorAll(".grid-row");
  const data = [];
  rows.forEach(r=>{
    const obj = {};
    fields.forEach(f=>{
      const inp = r.querySelector(`input[data-field="${CSS.escape(f)}"]`);
      if (inp) obj[f] = inp.value;
    });
    if (Object.keys(obj).length) data.push(obj);
  });
  projectData[category].data = data;
  saveData();
}

function renderCategory(category, fields, data=[]){
  const card = document.createElement("div");
  card.className = "cat";
  card.setAttribute("data-category", category);
  card.setAttribute("data-fields", fields.join(","));

  // Header
  const head = document.createElement("div");
  head.className = "cat-head";

  const title = document.createElement("h3");
  title.className = "cat-title";
  title.innerHTML = `<i class="fa-solid fa-folder"></i> ${category}`;
  head.appendChild(title);

  const headBtns = document.createElement("div");
  headBtns.className = "head-actions";

  const collapseBtn = document.createElement("button");
  collapseBtn.className = "btn btn-ghost";
  collapseBtn.title = "Collapse / Expand";
  collapseBtn.innerHTML = `<i class="fa-solid fa-chevron-up"></i>`;
  collapseBtn.addEventListener("click", ()=>{
    card.classList.toggle("collapsed");
    collapseBtn.innerHTML = card.classList.contains("collapsed")
      ? `<i class="fa-solid fa-chevron-down"></i>`
      : `<i class="fa-solid fa-chevron-up"></i>`;
  });
  headBtns.appendChild(collapseBtn);

  const renameBtn = document.createElement("button");
  renameBtn.className = "btn btn-blue";
  renameBtn.innerHTML = `<i class="fa-solid fa-pen"></i> Rename`;
  renameBtn.addEventListener("click", ()=>{
    const newName = prompt("Enter new category name:", category);
    if (!newName || newName === category) return;
    if (projectData[newName]) return alert("A category with that name already exists.");
    projectData[newName] = projectData[category];
    delete projectData[category];
    saveData();
    renderAll();
  });
  headBtns.appendChild(renameBtn);

  const delCatBtn = document.createElement("button");
  delCatBtn.className = "btn btn-red";
  delCatBtn.innerHTML = `<i class="fa-solid fa-trash"></i> Delete`;
  delCatBtn.addEventListener("click", ()=>{
    if (!confirm(`Delete category "${category}"?`)) return;
    delete projectData[category];
    saveData();
    renderAll();
  });
  headBtns.appendChild(delCatBtn);

  head.appendChild(headBtns);
  card.appendChild(head);

  // Grid header
  const header = document.createElement("div");
  header.className = "grid-header";
  header.style.gridTemplateColumns = `repeat(${fields.length}, minmax(0, 1fr)) 140px`;
  fields.forEach(f=>{
    const d = document.createElement("div");
    d.textContent = f.toUpperCase();
    header.appendChild(d);
  });
  const actLab = document.createElement("div");
  actLab.textContent = "ACTIONS";
  header.appendChild(actLab);
  card.appendChild(header);

  // rows
  data.forEach(entry => {
    card.appendChild(buildRow(category, fields, entry));
  });

  // add row
  const addWrap = document.createElement("div");
  addWrap.className = "add-row-wrap";
  const addBtn = document.createElement("button");
  addBtn.className = "btn btn-green add-row-btn";
  addBtn.style.width = "100%";
  addBtn.innerHTML = `<i class="fa-solid fa-plus"></i> Add Row`;
  addBtn.addEventListener("click", ()=>{
    const r = buildRow(category, fields);
    card.insertBefore(r, addWrap);
    updateCategoryData(category);
  });
  addWrap.appendChild(addBtn);
  card.appendChild(addWrap);

  container.appendChild(card);
}

function renderAll(){
  container.innerHTML = "";
  for (const cat in projectData){
    const { fields, data } = projectData[cat];
    renderCategory(cat, fields, data);
  }
}

/* ===================== TOP BAR ===================== */
document.getElementById("addCatBtn").addEventListener("click", ()=>{
  const name = document.getElementById("newCategoryInput").value.trim();
  const fields = toCols(document.getElementById("newFieldsInput").value.trim());
  if (!name || fields.length === 0){
    alert("Please enter both category name and column names (comma-separated).");
    return;
  }
  if (projectData[name]){
    alert("Category already exists.");
    return;
  }
  projectData[name] = { fields, data: [] };
  saveData();
  renderAll();
  document.getElementById("newCategoryInput").value = "";
  document.getElementById("newFieldsInput").value = "";
});

document.getElementById("exportBtn").addEventListener("click", ()=>{
  const blob = new Blob([JSON.stringify(projectData, null, 2)], {type:"application/json"});
  const a = document.createElement("a");
  a.href = URL.createObjectURL(blob);
  a.download = "credentials.json";
  a.click();
});

document.getElementById("importBtn").addEventListener("click", ()=>{
  document.getElementById("importFile").click();
});
document.getElementById("importFile").addEventListener("change", (e)=>{
  const file = e.target.files[0];
  if (!file) return;
  const reader = new FileReader();
  reader.onload = ev => {
    try{
      const data = JSON.parse(ev.target.result);
      if (!data || typeof data !== "object" || Array.isArray(data)) throw new Error("Invalid shape");
      projectData = data;
      saveData();
      renderAll();
    }catch(err){
      alert("Invalid JSON file.");
    }
  };
  reader.readAsText(file);
});

/* ===================== INIT ===================== */
renderAll();
</script>
</body>
</html>
----------------------------------------------------------------------------------------------------------------

// FedSecuritiesApi.java

import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.media.Content;
import io.swagger.v3.oas.annotations.media.Schema;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import io.swagger.v3.oas.annotations.responses.ApiResponses;
import io.swagger.v3.oas.annotations.tags.Tag;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@Tag(name = "Fed Securities Tests", description = "Operations related to Fed Securities testing")
@RequestMapping("/fedtests/fedSecurities")
public interface FedSecuritiesApi {

    @Operation(summary = "Run pre-test tasks for Fed Securities")
    @ApiResponses(value = {
        @ApiResponse(responseCode = "200", description = "Pre-test tasks executed successfully",
            content = @Content(schema = @Schema(implementation = FedTestResponse.class))),
        @ApiResponse(responseCode = "400", description = "Invalid input", content = @Content),
        @ApiResponse(responseCode = "500", description = "Server error", content = @Content)
    })
    @PostMapping("/runPreTestTasks")
    ResponseEntity<FedTestResponse> runPreTestTasks(@RequestBody FedTestRequest request);


    @Operation(summary = "Run post-test tasks for Fed Securities")
    @ApiResponses(value = {
        @ApiResponse(responseCode = "200", description = "Post-test tasks executed successfully",
            content = @Content(schema = @Schema(implementation = FedTestResponse.class))),
        @ApiResponse(responseCode = "400", description = "Invalid input", content = @Content),
        @ApiResponse(responseCode = "500", description = "Server error", content = @Content)
    })
    @PostMapping("/runPostTestTasks")
    ResponseEntity<FedTestResponse> runPostTestTasks(@RequestBody FedTestRequest request);
}

// FedSecuritiesController.java

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class FedSecuritiesController implements FedSecuritiesApi {

    @Override
    public ResponseEntity<FedTestResponse> runPreTestTasks(FedTestRequest request) {
        FedTestResponse response = new FedTestResponse();
        response.setStatus("SUCCESS");
        response.setMessage("Pre-test tasks executed for test ID: " + request.getTestId());
        return new ResponseEntity<>(response, HttpStatus.OK);
    }

    @Override
    public ResponseEntity<FedTestResponse> runPostTestTasks(FedTestRequest request) {
        FedTestResponse response = new FedTestResponse();
        response.setStatus("SUCCESS");
        response.setMessage("Post-test tasks executed for test ID: " + request.getTestId());
        return new ResponseEntity<>(response, HttpStatus.OK);
    }
}

// FedTestRequest.java
public class FedTestRequest {
    private String testId;
    private String description;

    // Getters and Setters
}
// FedTestResponse.java
public class FedTestResponse {
    private String status;
    private String message;

    // Getters and Setters
}

--------------------------------------------------------------
# --- CONFIGURATION ---
$sharedMailbox = "sharedmailbox@yourdomain.com"            # <-- Replace with your shared mailbox
$mailEnabledSecurityGroupName = "Your MESG Display Name"   # <-- Replace with your MESG name

# Connect and grant access
try {
    Write-Host "`n🌐 Connecting to Exchange Online..." -ForegroundColor Cyan
    Connect-ExchangeOnline -ErrorAction Stop

    # Resolve MESG
    Write-Host "🔍 Resolving group '$mailEnabledSecurityGroupName'..." -ForegroundColor Yellow
    $group = Get-Recipient -RecipientTypeDetails MailUniversalSecurityGroup -ErrorAction Stop |
             Where-Object { $_.Name -eq $mailEnabledSecurityGroupName }

    if (-not $group) {
        throw "Mail-Enabled Security Group '$mailEnabledSecurityGroupName' not found."
    }

    $groupAddress = $group.PrimarySmtpAddress
    Write-Host "✅ MESG resolved: $groupAddress" -ForegroundColor Green

    # Grant FullAccess
    Write-Host "🔐 Granting FullAccess to '$sharedMailbox' for '$groupAddress'..." -ForegroundColor Yellow
    Add-MailboxPermission -Identity $sharedMailbox `
                          -User $groupAddress `
                          -AccessRights FullAccess `
                          -InheritanceType All `
                          -AutoMapping $false -ErrorAction Stop

    Write-Host "✅ FullAccess permission granted successfully." -ForegroundColor Green
}
catch {
    Write-Host "`n❌ ERROR: $($_.Exception.Message)" -ForegroundColor Red
}
finally {
    Write-Host "`n🔌 Disconnecting from Exchange Online..." -ForegroundColor DarkCyan
    Disconnect-ExchangeOnline -Confirm:$false
    Write-Host "✅ Disconnected." -ForegroundColor Green
}

--------------------------------------------------------------
function Manage-SharedMailboxAccess {
    param (
        [Parameter(Mandatory = $true)]
        [PSCredential]$exchangeCreds,

        [Parameter(Mandatory = $true)]
        [hashtable]$taskParams
    )

    # Extract parameters
    $sharedMailbox = $taskParams["SharedMailbox"]
    $groupName     = $taskParams["SecurityGroup"]  # Now just the group name

    if (-not $sharedMailbox -or -not $groupName) {
        Write-Error "Missing required parameters: SharedMailbox, SecurityGroup (group name)"
        return
    }

    # Check if already connected to Exchange Online
    $isConnected = $false
    try {
        $sessionInfo = Get-ConnectionInformation -ErrorAction Stop
        if ($sessionInfo.ConnectedEndpoints -contains "ExchangeOnline") {
            $isConnected = $true
        }
    } catch {
        # Not connected
    }

    if (-not $isConnected) {
        try {
            Write-Host "Connecting to Exchange Online..." -ForegroundColor Cyan
            Connect-ExchangeOnline -Credential $exchangeCreds `
                                   -WarningAction SilentlyContinue `
                                   -ErrorAction Stop `
                                   -ShowBanner:$false
        } catch {
            Write-Error "Failed to connect to Exchange Online: $_"
            return
        }
    }

    # Resolve group name to identity
    try {
        $group = Get-Recipient -RecipientTypeDetails MailUniversalSecurityGroup -Identity $groupName -ErrorAction Stop
        $resolvedGroupIdentity = $group.Name
    } catch {
        Write-Error "Failed to resolve group name '$groupName' in Exchange Online."
        return
    }

    # Add mailbox permission (Read-only or FullAccess)
    try {
        Write-Host "Adding 'ReadPermission' to '$resolvedGroupIdentity' on mailbox '$sharedMailbox'" -ForegroundColor Green

        # 🔹 Grant read-only access (this is rarely used and more symbolic here)
        Add-MailboxPermission -Identity $sharedMailbox `
                              -User $resolvedGroupIdentity `
                              -AccessRights ReadPermission `
                              -InheritanceType All `
                              -AutoMapping:$false

        # 🔒 COMMENTED: To give full access instead of read-only, use below line:
        # Add-MailboxPermission -Identity $sharedMailbox `
        #                       -User $resolvedGroupIdentity `
        #                       -AccessRights FullAccess `
        #                       -InheritanceType All `
        #                       -AutoMapping:$false

        Write-Host "Access granted successfully." -ForegroundColor Green
    } catch {
        Write-Error "Error granting access to mailbox '$sharedMailbox': $_"
    }
}

----------------------------------------------------------------
function Manage-SharedMailboxAccess {
    param (
        [Parameter(Mandatory = $true)]
        [PSCredential]$exchangeCreds,

        [Parameter(Mandatory = $true)]
        [hashtable]$taskParams
    )

    # Extract parameters
    $sharedMailbox = $taskParams["SharedMailbox"]
    $securityGroup = $taskParams["SecurityGroup"]
    $accessRights  = $taskParams["AccessRights"]
    $action        = $taskParams["Action"]

    if (-not $sharedMailbox -or -not $securityGroup -or -not $accessRights -or -not $action) {
        Write-Error "Missing required parameters: SharedMailbox, SecurityGroup, AccessRights, Action"
        return
    }

    # Check if already connected to Exchange Online
    $isConnected = $false
    try {
        $sessionInfo = Get-ConnectionInformation -ErrorAction Stop
        if ($sessionInfo.ConnectedEndpoints -contains "ExchangeOnline") {
            $isConnected = $true
        }
    } catch {
        # Not connected
    }

    if (-not $isConnected) {
        try {
            Write-Host "Connecting to Exchange Online..." -ForegroundColor Cyan
            Connect-ExchangeOnline -Credential $exchangeCreds `
                                   -WarningAction SilentlyContinue `
                                   -ErrorAction Stop `
                                   -ShowBanner:$false
        } catch {
            Write-Error "Failed to connect to Exchange Online: $_"
            return
        }
    }

    try {
        switch ($action.ToLower()) {
            "add" {
                Write-Host "Adding '$accessRights' permission to '$securityGroup' on mailbox '$sharedMailbox'" -ForegroundColor Green
                Add-MailboxPermission -Identity $sharedMailbox `
                                      -User $securityGroup `
                                      -AccessRights $accessRights `
                                      -InheritanceType All `
                                      -AutoMapping:$false
                Write-Host "Access granted." -ForegroundColor Green
            }

            "remove" {
                Write-Host "Removing '$accessRights' permission from '$securityGroup' on mailbox '$sharedMailbox'" -ForegroundColor Yellow
                Remove-MailboxPermission -Identity $sharedMailbox `
                                         -User $securityGroup `
                                         -AccessRights $accessRights `
                                         -InheritanceType All `
                                         -Confirm:$false
                Write-Host "Access revoked." -ForegroundColor Yellow
            }

            "update" {
                Write-Host "Updating permission: removing then adding '$accessRights' for '$securityGroup' on '$sharedMailbox'" -ForegroundColor Cyan
                Remove-MailboxPermission -Identity $sharedMailbox `
                                         -User $securityGroup `
                                         -AccessRights $accessRights `
                                         -InheritanceType All `
                                         -Confirm:$false -ErrorAction SilentlyContinue

                Add-MailboxPermission -Identity $sharedMailbox `
                                      -User $securityGroup `
                                      -AccessRights $accessRights `
                                      -InheritanceType All `
                                      -AutoMapping:$false
                Write-Host "Access updated." -ForegroundColor Cyan
            }

            default {
                Write-Error "Invalid action: '$action'. Use 'Add', 'Remove', or 'Update'."
            }
        }
    } catch {
        Write-Error "Error performing '$action' on mailbox '$sharedMailbox': $_"
    }
}



----------------------------------------
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Credential Manager</title>
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
  <h3 class="mb-4 text-primary text-center">Credential Manager</h3>

  <div class="d-flex gap-2 mb-3">
    <input type="text" id="newCategoryInput" class="form-control" placeholder="Category Name (e.g. QA)" />
    <input type="text" id="newFieldsInput" class="form-control" placeholder="Fields (comma-separated)" />
    <button class="btn btn-outline-success add-cat-btn" onclick="addCategory()">➕ Add Category</button>
  </div>

  <div id="categoriesContainer"></div>

  <!--<h6 class="text-muted mt-4">🧾 Local Storage:</h6>
  <pre id="jsonOutput"></pre>-->
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
      if (field.toLowerCase() === "password") {
	   input.type = "password";
	  }
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

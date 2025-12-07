<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Sistema de Almoxarifado</title>

<!-- BIBLIOTECA DE ASSINATURA PROFISSIONAL -->
<script src="https://cdn.jsdelivr.net/npm/signature_pad@4.0.0/dist/signature_pad.umd.min.js"></script>

<style>
    body { font-family: Arial; background:#111; color:#fff; margin:0; }
    header { background:#ff6600; padding:15px; text-align:center; font-size:24px; font-weight:bold; }
    nav { display:flex; background:#222; }
    nav button {
        flex:1; padding:12px; background:#222; color:#fff;
        border:1px solid #333; cursor:pointer; font-size:16px;
    }
    nav button:hover { background:#ff6600; }
    .page { display:none; padding:20px; }
    input, select {
        padding:10px; width:100%; margin:8px 0;
        background:#222; color:#fff; border:1px solid #555;
    }
    button.action {
        padding:10px; background:#ff6600; color:#000;
        border:none; font-size:16px; cursor:pointer; width:100%;
        margin-top:10px;
    }
    table { width:100%; margin-top:20px; border-collapse:collapse; }
    th, td {
        padding:10px; border:1px solid #444; text-align:center;
    }

    /* POP-UP */
    #popupFundo {
        display:none; position:fixed; top:0; left:0; width:100%; height:100%;
        background:rgba(0,0,0,0.8); justify-content:center; align-items:center;
        z-index:9999;
    }
    #popup {
        background:#fff; padding:20px; width:90%; max-width:500px;
        border-radius:10px; color:#000;
        text-align:center;
    }
    #signaturePad {
        border:2px solid #ff6600;
        border-radius:10px;
        background:white;
        width:100%;
        height:260px;
    }
</style>
</head>
<body>

<header>Sistema de Almoxarifado</header>

<nav>
    <button onclick="showPage('emprestimo')">Empréstimo</button>
    <button onclick="showPage('devolucao')">Devolução</button>
    <button onclick="showPage('almox')">Almoxarifado</button>
    <button onclick="showPage('registros')">Registros</button>
</nav>

<!-- -------------------- EMPRÉSTIMO -------------------- -->
<div id="emprestimo" class="page" style="display:block;">
    <h2>Empréstimo de Ferramenta</h2>

    <label>Nome da Pessoa</label>
    <input id="nomePessoa">

    <label>Nome da Ferramenta</label>
    <input id="nomeFerramenta">

    <label>Data</label>
    <input id="dataEmp" type="date">

    <button class="action" onclick="abrirPopup()">Assinar Digitalmente</button>

    <button class="action" onclick="salvarEmprestimo()">Salvar Empréstimo</button>
</div>

<!-- POP-UP DE ASSINATURA DIGITAL -->
<div id="popupFundo">
    <div id="popup">
        <h3>Assinatura Digital</h3>

        <canvas id="signaturePad"></canvas>

        <button class="action" onclick="limparAssinatura()">Limpar</button>
        <button class="action" onclick="confirmarAssinatura()">Confirmar Assinatura</button>
    </div>
</div>

<!-- -------------------- DEVOLUÇÃO -------------------- -->
<div id="devolucao" class="page">
    <h2>Devolução</h2>

    <label>Nome da Pessoa</label>
    <input id="devolucaoPessoa">

    <label>Nome da Ferramenta</label>
    <input id="devolucaoFerramenta">

    <button class="action" onclick="registrarDevolucao()">Registrar Devolução</button>
</div>

<!-- -------------------- ALMOXARIFADO -------------------- -->
<div id="almox" class="page">
    <h2>Almoxarifado</h2>

    <label>Nome da ferramenta</label>
    <input id="almoxNome">

    <label>Quantidade</label>
    <input id="almoxQtd" type="number">

    <label>Categoria</label>
    <select id="almoxCat">
        <option value="manual">Manual</option>
        <option value="eletrica">Elétrica</option>
        <option value="hidraulica">Hidráulica</option>
    </select>

    <button class="action" onclick="adicionarFerramenta()">Adicionar</button>

    <h3>Filtros</h3>

    <label>Quantidade mínima</label>
    <input type="number" id="filtroQtd">

    <label>Categoria</label>
    <select id="filtroCat">
        <option value="todas">Todas</option>
        <option value="manual">Manual</option>
        <option value="eletrica">Elétrica</option>
        <option value="hidraulica">Hidráulica</option>
    </select>

    <button class="action" onclick="ordenarAZ()">Ordenar A–Z</button>
    <button class="action" onclick="filtrar()">Aplicar Filtro</button>
    <button class="action" onclick="mostrarTudo()">Mostrar Tudo</button>
    <button class="action" onclick="filtrarBaixa()">Baixa Quantidade</button>

    <table>
        <thead>
            <tr><th>Ferramenta</th><th>Qtd</th><th>Categoria</th></tr>
        </thead>
        <tbody id="tabelaAlmox"></tbody>
    </table>
</div>

<!-- -------------------- REGISTROS -------------------- -->
<div id="registros" class="page">
    <h2>Registros</h2>
    <table>
        <thead>
            <tr><th>Pessoa</th><th>Ferramenta</th><th>Data</th><th>Assinatura</th></tr>
        </thead>
        <tbody id="tabelaRegistros"></tbody>
    </table>
</div>

<script>
let ferramentas = [];
let registros = [];
let assinaturaOK = false;
let assinaturaImg = "";

/* PÁGINAS */
function showPage(id) {
    document.querySelectorAll('.page').forEach(p => p.style.display = 'none');
    document.getElementById(id).style.display = 'block';
}

/* POP-UP */
function abrirPopup() { document.getElementById("popupFundo").style.display = "flex"; }
function fecharPopup() { document.getElementById("popupFundo").style.display = "none"; }

/* ASSINATURA DIGITAL PROFISSIONAL */
let canvas = document.getElementById("signaturePad");
let signaturePad = new SignaturePad(canvas, {
    backgroundColor: "white",
    penColor: "black"
});

function limparAssinatura() {
    signaturePad.clear();
}

function confirmarAssinatura() {
    if (signaturePad.isEmpty()) {
        alert("Faça a assinatura primeiro!");
        return;
    }

    assinaturaImg = signaturePad.toDataURL();
    assinaturaOK = true;
    fecharPopup();
    alert("Assinatura salva!");
}

/* SALVAR EMPRÉSTIMO */
function salvarEmprestimo() {
    let pessoa = document.getElementById("nomePessoa").value;
    let ferramenta = document.getElementById("nomeFerramenta").value;
    let data = document.getElementById("dataEmp").value;

    if (!pessoa || !ferramenta || !data) {
        alert("Preencha nome, ferramenta e data!");
        return;
    }

    if (!assinaturaOK) {
        alert("Você precisa assinar antes!");
        return;
    }

    registros.push({ pessoa, ferramenta, data, assinatura: assinaturaImg });
    atualizarRegistros();

    assinaturaOK = false;
    signaturePad.clear();

    alert("Empréstimo salvo!");
}

/* DEVOLUÇÃO */
function registrarDevolucao() {
    let nome = document.getElementById("devolucaoFerramenta").value;
    alert("Ferramenta devolvida: " + nome);
}

/* ALMOXARIFADO */
function adicionarFerramenta() {
    let nome = document.getElementById("almoxNome").value;
    let qtd = parseInt(document.getElementById("almoxQtd").value);
    let cat = document.getElementById("almoxCat").value;

    ferramentas.push({ nome, qtd, cat });
    mostrarTudo();
}

function mostrarTudo() { atualizarTabela(ferramentas); }

function filtrar() {
    let qtd = parseInt(document.getElementById("filtroQtd").value);
    let cat = document.getElementById("filtroCat").value;

    atualizarTabela(
        ferramentas.filter(f =>
            (isNaN(qtd) || f.qtd >= qtd) &&
            (cat === "todas" || f.cat === cat)
        )
    );
}

function ordenarAZ() {
    ferramentas.sort((a,b)=>a.nome.localeCompare(b.nome));
    mostrarTudo();
}

function filtrarBaixa() {
    atualizarTabela(ferramentas.filter(f => f.qtd <= 2));
}

/* TABELAS */
function atualizarTabela(lista) {
    let tabela = document.getElementById("tabelaAlmox");
    tabela.innerHTML = "";
    lista.forEach(f => tabela.innerHTML += `<tr><td>${f.nome}</td><td>${f.qtd}</td><td>${f.cat}</td></tr>`);
}

function atualizarRegistros() {
    let tabela = document.getElementById("tabelaRegistros");
    tabela.innerHTML = "";
    registros.forEach(r => {
        tabela.innerHTML += `
        <tr>
            <td>${r.pessoa}</td>
            <td>${r.ferramenta}</td>
            <td>${r.data}</td>
            <td><img src="${r.assinatura}" width="100"></td>
        </tr>`;
    });
}
</script>

</body>
</html>

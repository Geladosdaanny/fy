<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Gelados Da Anny 🍧</title>

<!-- jsPDF -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
<!-- QR Code -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>

<style>
body{
    margin:0;
    font-family:Segoe UI,Arial,sans-serif;
    background:radial-gradient(circle at top,#ffe1ec,#ffc1d9);
}
header{
    text-align:center;
    padding:30px;
    color:#fff;
    background:linear-gradient(135deg,#ff3f8e,#ff78b0);
    box-shadow:0 15px 30px rgba(0,0,0,.3);
}
section{
    max-width:900px;
    margin:auto;
    padding:20px;
}
.card{
    background:#fff;
    border-radius:18px;
    padding:20px;
    margin-bottom:20px;
    box-shadow:0 20px 40px rgba(0,0,0,.15);
}
h2{color:#ff3f8e}
input,select,button{
    width:100%;
    padding:12px;
    margin-top:10px;
    border-radius:10px;
    border:1px solid #ccc;
    font-size:15px;
}
button{
    background:linear-gradient(135deg,#ff3f8e,#ff78b0);
    color:#fff;
    border:none;
    font-weight:bold;
    cursor:pointer;
}
.status{
    text-align:center;
    font-weight:bold;
    margin-bottom:15px;
}
.aberto{color:#0a8f3c}
.fechado{color:#b30000}
ul{padding-left:15px}
li{margin-bottom:8px}
#qrcodePix{text-align:center;margin-top:10px}
footer{text-align:center;padding:15px;color:#555}
</style>
</head>

<body>

<header>
<h1>🍧 Gelados Da Anny</h1>
<p>Dindin & Mousses Artesanais</p>
</header>

<section>

<div id="statusLoja" class="status"></div>

<div class="card">
<h2>👤 Cliente</h2>
<input id="nome" placeholder="Nome do cliente">
<input id="endereco" placeholder="Endereço completo">
</div>

<div class="card">
<h2>🍦 Produto</h2>
<select id="produto">
<option>Dindin Tradicional</option>
<option>Dindin com Recheio de Chocolate 🍫</option>
<option>Mousse Tradicional</option>
<option>Mousse com Recheio de Chocolate 🍫</option>
</select>

<select id="sabor">
<option>Morango 🍓</option>
<option>Uva 🍇</option>
<option>Abacaxi 🍍</option>
<option>Maracujá</option>
<option>Chocolate 🍫</option>
</select>

<input type="number" id="quantidade" placeholder="Quantidade" min="1">
<input type="number" id="valor" placeholder="Valor unitário (R$)">
<button onclick="adicionarCarrinho()">➕ Adicionar ao Carrinho</button>
</div>

<div class="card">
<h2>🛒 Carrinho</h2>
<ul id="listaCarrinho"></ul>
<p><b>Total:</b> R$ <span id="totalCarrinho">0.00</span></p>
<button onclick="limparCarrinho()">🧹 Limpar Carrinho</button>
</div>

<div class="card">
<h2>💳 Pagamento</h2>
<select id="pagamento">
<option>Pix</option>
<option>Dinheiro</option>
<option>Cartão</option>
</select>

<p><b>🔑 Chave Pix:</b><br>
cf0299ff-c43f-41ee-9a26-d35b07463e3b</p>

<div id="qrcodePix"></div>
<p id="pixValor"></p>
</div>

<button onclick="finalizarPedido()">📲 Finalizar Pedido</button>

</section>

<footer>
💖 Obrigado por escolher a Gelados Da Anny
</footer>

<script>
const chavePix = "cf0299ff-c43f-41ee-9a26-d35b07463e3b";
let carrinho = [];

// HORÁRIO DA LOJA
const statusDiv = document.getElementById("statusLoja");
const agora = new Date();
const dia = agora.getDay();
const hora = agora.getHours();
const lojaAberta = !(dia===0 || dia===6 || hora<15 || hora>=22);

statusDiv.textContent = lojaAberta
 ? "🟢 Loja aberta — Segunda a Sexta, 15h às 22h"
 : "🔴 Loja fechada — Atendimento apenas em horário comercial";
statusDiv.className += lojaAberta ? " aberto" : " fechado";

// ELEMENTOS
const nomeEl = document.getElementById("nome");
const enderecoEl = document.getElementById("endereco");
const produtoEl = document.getElementById("produto");
const saborEl = document.getElementById("sabor");
const qtdEl = document.getElementById("quantidade");
const valorEl = document.getElementById("valor");
const listaEl = document.getElementById("listaCarrinho");
const totalEl = document.getElementById("totalCarrinho");

// CARRINHO
function adicionarCarrinho(){
    const qtd = parseInt(qtdEl.value);
    const valor = parseFloat(valorEl.value);
    if(!qtd || !valor){ alert("Informe quantidade e valor."); return; }

    carrinho.push({
        produto: produtoEl.value,
        sabor: saborEl.value,
        qtd, valor
    });
    atualizarCarrinho();
}

function removerItem(i){
    carrinho.splice(i,1);
    atualizarCarrinho();
}

function limparCarrinho(){
    carrinho = [];
    atualizarCarrinho();
}

function atualizarCarrinho(){
    listaEl.innerHTML="";
    let total=0;

    carrinho.forEach((item,i)=>{
        const sub = item.qtd * item.valor;
        total += sub;
        listaEl.innerHTML += `
        <li>🍦 ${item.produto} (${item.sabor}) —
        ${item.qtd}x R$ ${item.valor.toFixed(2)}
        = <b>R$ ${sub.toFixed(2)}</b>
        <button onclick="removerItem(${i})">❌</button></li>`;
    });

    totalEl.textContent = total.toFixed(2);
    gerarQRCodePix(total);
}

// QR CODE PIX
function gerarQRCodePix(valor){
    const box = document.getElementById("qrcodePix");
    const txt = document.getElementById("pixValor");
    box.innerHTML=""; txt.innerHTML="";
    if(valor<=0) return;

    const payload =
`00020126360014BR.GOV.BCB.PIX0114${chavePix}520400005303986540${valor.toFixed(2)}5802BR5920Gelados Da Anny6009Brasil62130509PedidoPix6304`;

    new QRCode(box,{text:payload,width:200,height:200});
    txt.innerHTML = `💰 Valor do Pix: <b>R$ ${valor.toFixed(2)}</b>`;
}

// FINALIZAR PEDIDO
function finalizarPedido(){
    if(!lojaAberta){ alert("Loja fechada."); return; }
    if(carrinho.length===0){ alert("Carrinho vazio."); return; }
    if(!nomeEl.value || !enderecoEl.value){ alert("Preencha seus dados."); return; }

    const pedido = Math.floor(Math.random()*9000)+1000;
    let resumo="", total=0;

    carrinho.forEach(i=>{
        const t=i.qtd*i.valor;
        total+=t;
        resumo+=`• ${i.produto} (${i.sabor}) - ${i.qtd}x R$ ${i.valor.toFixed(2)} = R$ ${t.toFixed(2)}\n`;
    });

    // PDF
    const { jsPDF } = window.jspdf;
    const pdf = new jsPDF();
    pdf.text("Comprovante - Gelados Da Anny",10,10);
    pdf.text(`Pedido Nº ${pedido}`,10,20);
    pdf.text(`Cliente: ${nomeEl.value}`,10,30);
    pdf.text(`Endereço: ${enderecoEl.value}`,10,40);
    pdf.text("Itens:",10,50);
    pdf.text(resumo,10,60);
    pdf.text(`Total: R$ ${total.toFixed(2)}`,10,120);
    pdf.save(`Pedido_${pedido}.pdf`);

    const msg =
`🍧*Pedido Nº ${pedido} - Gelados Da Anny* 🍧
👤${nomeEl.value}
📍${enderecoEl.value}

🛒Itens:
${resumo}
💰Total: R$ ${total.toFixed(2)}

🔑 Pix:
${chavePix}`;

    window.open(
      "https://wa.me/5586998120194?text="+encodeURIComponent(msg),
      "_blank"
    );
}
</script>

</body>
</html>

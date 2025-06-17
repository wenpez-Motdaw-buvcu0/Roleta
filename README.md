<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8"/>
  <meta name="viewport" content="width=device-width, initial-scale=1"/>
  <title>Plataforma de Roleta</title>
  <style>
    body { font-family: Arial, sans-serif; margin:0; padding:20px; }
    .hidden { display: none; }
    #loginForm, #roletaPage { max-width: 320px; margin: auto; }
    input, button { width: 100%; padding: 10px; margin: 6px 0; font-size: 16px; }
    button { cursor: pointer; background: #4CAF50; color: #fff; border: none; border-radius:4px; }
    canvas { display: block; margin: auto; border:4px solid #444; border-radius:50%; }
    #seta { text-align: center; font-size: 40px; margin-top: -10px; }
    #resultado, #saldo { text-align: center; margin-top: 12px; font-weight: bold;}
  </style>
</head>
<body>

<div id="loginForm">
  <h2>Login / Cadastro</h2>
  <input type="email" id="email" placeholder="Seu e‑mail" required>
  <input type="password" id="senha" placeholder="Sua senha" required>
  <button id="btnLogin">Entrar ou Cadastrar</button>
  <p id="msgLogin" style="color:red;"></p>
</div>

<div id="roletaPage" class="hidden">
  <h2>Bem-vindo, <span id="userEmail"></span></h2>
  <p id="saldo">Você tem <span id="giros">3</span> giros.</p>
  <div id="seta">🔻</div>
  <canvas id="roleta" width="300" height="300"></canvas>
  <button id="btnGirar">Girar Roleta 🎯</button>
  <div id="resultado"></div>
  <button id="btnLogout" style="background:#e74c3c;">Sair</button>
</div>

<script>
  const premios = ['R$100', 'Tente de novo', 'R$50', 'Cupom', 'R$10', 'Nada'];
  const cores = ['#FF5733','#33C1FF','#75FF33','#FFD433','#B833FF','#FF33A6'];
  const canvas = document.getElementById('roleta'), ctx = canvas.getContext('2d');
  const tamanho = 360/premios.length;
  let anguloAtual = 0, girando = false;

  function desenhar() {
    premios.forEach((p, i) => {
      const start = (i*tamanho)*Math.PI/180, end = ((i+1)*tamanho)*Math.PI/180;
      ctx.beginPath();
      ctx.moveTo(150,150);
      ctx.arc(150,150,150,start,end);
      ctx.fillStyle = cores[i];
      ctx.fill(); ctx.stroke();
      ctx.save();
      ctx.translate(150,150);
      ctx.rotate((start+end)/2);
      ctx.textAlign = 'right'; ctx.fillStyle='#000';
      ctx.font='bold 14px Arial'; ctx.fillText(p,140,10);
      ctx.restore();
    });
  }
  desenhar();

  // Login/logout
  const loginForm = document.getElementById('loginForm'), roletaPage = document.getElementById('roletaPage');
  const btnLogin = document.getElementById('btnLogin'), msgLogin = document.getElementById('msgLogin');
  const emailIn = document.getElementById('email'), senhaIn = document.getElementById('senha');
  const userEmail = document.getElementById('userEmail'), girosEl = document.getElementById('giros');
  const saldoGiros = 3;

  btnLogin.onclick = () => {
    const email = emailIn.value.trim(), senha = senhaIn.value;
    if(!email || !senha) { msgLogin.textContent = 'Preencha e‑mail e senha.'; return; }
    let user = JSON.parse(localStorage.getItem(email));
    if(user) {
      if(user.senha !== senha) { msgLogin.textContent = 'Senha incorreta.'; return;}
    } else {
      user = { senha, giros: saldoGiros, historico: [] };
      localStorage.setItem(email, JSON.stringify(user));
    }
    localStorage.setItem('logado', email);
    showRoleta(email);
  };

  // Logout
  document.getElementById('btnLogout').onclick = () => {
    localStorage.removeItem('logado');
    roletaPage.classList.add('hidden');
    loginForm.classList.remove('hidden');
    emailIn.value=''; senhaIn.value='';
  };

  // Mostrar roleta
  function showRoleta(email) {
    loginForm.classList.add('hidden');
    roletaPage.classList.remove('hidden');
    userEmail.textContent = email;
    const user = JSON.parse(localStorage.getItem(email));
    girosEl.textContent = user.giros;
    document.getElementById('resultado').textContent = '';
  }

  // Se já estiver logado
  window.onload = () => {
    const log = localStorage.getItem('logado');
    if(log) showRoleta(log);
  };

  // Girar roleta
  document.getElementById('btnGirar').onclick = () => {
    const email = localStorage.getItem('logado');
    let user = JSON.parse(localStorage.getItem(email));
    if(user.giros < 1) { alert('Sem giros!'); return; }
    if(girando) return;
    girando = true;
    const idx = Math.floor(Math.random()*premios.length);
    const extra = 360*5 + (360 - idx*tamanho - tamanho/2);
    anguloAtual += extra;
    canvas.style.transition = 'transform 4s ease-out';
    canvas.style.transform = `rotate(${anguloAtual}deg)`;
    setTimeout(() => {
      const premio = premios[idx];
      document.getElementById('resultado').textContent = `🎉 Você ganhou: ${premio}`;
      user.giros--;
      user.historico.push({ data: new Date(), premio });
      localStorage.setItem(email, JSON.stringify(user));
      girosEl.textContent = user.giros;
      girando = false;
      canvas.style.transition = ''; // reset
    }, 4000);
  };
</script>

</body>
</html>

// ----- Template Config (edit only this part) -----
const template = {
  title: "happy birthday my girlfriend!!",
  subtitle: "there’s more waiting… click to find out ✨",
  slides: [
    { img: "assets/images/photo1.jpg", text: "kamu makin indah setiap tahunnya. selamat ulang tahun, sayang." },
    { img: "assets/images/photo2.jpg", text: "kamu pantas dapat semua cinta hari ini dan selalu." },
    { img: "assets/images/photo3.jpg", text: "setiap tahun kamu makin bijak, baik, dan lebih menakjubkan." },
    { img: "assets/images/photo4.jpg", text: "setiap foto kita selalu bikin aku makin suka sama kamu." }
  ],
  letter: "Dear kamu, hari ini adalah hari istimewa — terima kasih sudah jadi kamu. Aku berharap yang terbaik untukmu selalu. — With love",
  audio: "assets/music/bgm.wav",
  quiz: [
    {q:"Kapan pertama kali kita bertemu?", options:["Januari","Februari","Maret","April"], answer:2},
    {q:"Makanan favorit pasanganmu adalah?", options:["Pizza","Sushi","Nasi Goreng","Pasta"], answer:0},
    {q:"Tempat liburan impian kalian?", options:["Bali","Paris","Tokyo","Maldives"], answer:1}
  ],
  memoryImages: ["assets/images/photo1.jpg","assets/images/photo2.jpg","assets/images/photo3.jpg","assets/images/photo4.jpg"]
};
// ----- End config -----

document.addEventListener('DOMContentLoaded', ()=>{
  // scenes
  const sceneOpen = document.getElementById('scene-open');
  const sceneMessages = document.getElementById('scene-messages');
  const sceneGames = document.getElementById('scene-games');
  const sceneLetter = document.getElementById('scene-letter');
  const openBtn = document.getElementById('openBtn');
  const bgm = document.getElementById('bgm');
  const playBtn = document.getElementById('playBtn');
  const pauseBtn = document.getElementById('pauseBtn');
  const slidesContainer = document.getElementById('slidesContainer');
  const toGames = document.getElementById('to-games');
  const toLetter = document.getElementById('to-letter');
  const finalReplay = document.getElementById('finalReplay');

  // populate header and slides
  document.getElementById('mainTitle').innerText = template.title;
  document.getElementById('subtitle').innerText = template.subtitle;
  document.getElementById('letterText').innerText = template.letter;
  bgm.src = template.audio;

  template.slides.forEach((s, i)=>{
    const div = document.createElement('div');
    div.className = 'slide hidden';
    div.id = 'slide-'+i;
    div.innerHTML = `<img src="${s.img}" alt="slide${i}"><p class="text">${s.text}</p><button class="btn slide-next">next</button>`;
    slidesContainer.appendChild(div);
  });

  // open flow
  openBtn.addEventListener('click', ()=>{
    sceneOpen.classList.remove('active');
    sceneOpen.classList.add('hidden');
    sceneMessages.classList.remove('hidden');
    sceneMessages.classList.add('active');
    document.getElementById('slide-0').classList.remove('hidden');
    try{ bgm.play(); }catch(e){}
  });

  // slide next
  slidesContainer.addEventListener('click', (ev)=>{
    if(ev.target.classList.contains('slide-next')){
      const curr = ev.target.closest('.slide');
      const next = curr.nextElementSibling;
      curr.classList.add('hidden');
      if(next) next.classList.remove('hidden');
    }
  });

  toGames.addEventListener('click', ()=>{
    sceneMessages.classList.add('hidden');
    sceneGames.classList.remove('hidden');
    sceneGames.classList.add('active');
  });

  toLetter.addEventListener('click', ()=>{
    sceneGames.classList.add('hidden');
    sceneLetter.classList.remove('hidden');
    sceneLetter.classList.add('active');
    // confetti on letter
    launchConfetti(120);
  });

  finalReplay.addEventListener('click', ()=> location.reload());

  playBtn.addEventListener('click', ()=> bgm.play());
  pauseBtn.addEventListener('click', ()=> bgm.pause());

  // ----- Puzzle (15-puzzle style) -----
  const puzzleCanvas = document.getElementById('puzzleCanvas');
  const ctx = puzzleCanvas.getContext('2d');
  const puzzleImg = new Image();
  puzzleImg.src = 'assets/images/puzzle.jpg';
  const gridSize = 3; // 3x3 puzzle for simplicity
  let tiles = [], emptyIndex;
  puzzleImg.onload = ()=> initPuzzle();

  function initPuzzle(){
    const tileW = puzzleCanvas.width / gridSize;
    const tileH = puzzleCanvas.height / gridSize;
    tiles = [];
    for(let y=0;y<gridSize;y++){
      for(let x=0;x<gridSize;x++){
        tiles.push({sx:x*tileW, sy:y*tileH, ix:x, iy:y});
      }
    }
    // remove last tile to create empty
    emptyIndex = tiles.length-1;
    // shuffle
    shuffleTiles();
    drawPuzzle();
  }

  function shuffleTiles(){
    for(let i=0;i<200;i++){
      const r = Math.floor(Math.random()*tiles.length);
      swapTiles(r, emptyIndex);
    }
  }

  function swapTiles(a,b){
    const tmp = tiles[a];
    tiles[a] = tiles[b];
    tiles[b] = tmp;
  }

  function drawPuzzle(){
    const tw = puzzleCanvas.width / gridSize;
    const th = puzzleCanvas.height / gridSize;
    ctx.clearRect(0,0,puzzleCanvas.width,puzzleCanvas.height);
    tiles.forEach((t,i)=>{
      if(i===emptyIndex)return; // skip empty
      ctx.drawImage(puzzleImg, t.sx, t.sy, tw, th, (i%gridSize)*tw, Math.floor(i/gridSize)*th, tw, th);
      ctx.strokeStyle = 'rgba(255,255,255,0.6)';
      ctx.strokeRect((i%gridSize)*tw, Math.floor(i/gridSize)*th, tw, th);
    });
  }

  puzzleCanvas.addEventListener('click', (ev)=>{
    const rect = puzzleCanvas.getBoundingClientRect();
    const x = Math.floor((ev.clientX - rect.left) / (puzzleCanvas.width / gridSize));
    const y = Math.floor((ev.clientY - rect.top) / (puzzleCanvas.height / gridSize));
    const idx = y*gridSize + x;
    // if clicked tile adjacent to empty, swap
    const adj = [idx-1, idx+1, idx-gridSize, idx+gridSize];
    if(adj.includes(emptyIndex)){
      swapTiles(idx, emptyIndex);
      emptyIndex = idx;
      drawPuzzle();
      if(checkPuzzleSolved()) setTimeout(()=> alert('Puzzle lengkap! 🎉'), 200);
    }
  });

  document.getElementById('shufflePuzzle').addEventListener('click', ()=>{ shuffleTiles(); drawPuzzle(); });

  function checkPuzzleSolved(){
    // naive check using original ordering
    for(let i=0;i<tiles.length-1;i++){
      const t = tiles[i];
      const correct = (t.ix === (i % gridSize)) && (t.iy === Math.floor(i/gridSize));
      if(!correct) return false;
    }
    return true;
  }

  // ----- Memory game -----
  const memoryBoard = document.getElementById('memoryBoard');
  let memoryDeck = [];
  function initMemory(){
    memoryBoard.innerHTML = '';
    const imgs = template.memoryImages.slice(0,8); // ensure <=8
    let pairs = imgs.concat(imgs);
    pairs = pairs.slice(0,16); // max 16 cards
    shuffleArray(pairs);
    pairs.forEach((src, idx)=>{
      const card = document.createElement('div');
      card.className = 'memory-card';
      card.dataset.src = src;
      card.dataset.idx = idx;
      card.innerText = '?';
      memoryBoard.appendChild(card);
    });
    memoryDeck = [];
  }
  memoryBoard.addEventListener('click', (ev)=>{
    const c = ev.target.closest('.memory-card');
    if(!c) return;
    if(c.classList.contains('revealed')) return;
    c.classList.add('revealed');
    c.style.background = '#fff';
    c.innerText = '❤';
    memoryDeck.push(c);
    if(memoryDeck.length===2){
      const a = memoryDeck[0], b = memoryDeck[1];
      if(a.dataset.src===b.dataset.src){
        // matched
        memoryDeck = [];
        if(document.querySelectorAll('.memory-card.revealed').length === document.querySelectorAll('.memory-card').length){
          setTimeout(()=> alert('Kamu menang memory! 🎉'),200);
        }
      } else {
        setTimeout(()=>{
          a.classList.remove('revealed');
          b.classList.remove('revealed');
          a.innerText = '?'; b.innerText = '?';
          memoryDeck = [];
        },700);
      }
    }
  });
  document.getElementById('resetMemory').addEventListener('click', initMemory);

  // ----- Quiz -----
  const quizArea = document.getElementById('quizArea');
  let quizIndex = 0, score = 0;
  function renderQuiz(){
    quizArea.innerHTML = '';
    if(quizIndex >= template.quiz.length){
      quizArea.innerHTML = `<p>Kamu selesai! Skor: ${score}/${template.quiz.length}</p><button class="small-btn" onclick="location.reload()">Ulangi Quiz</button>`;
      return;
    }
    const q = template.quiz[quizIndex];
    const qdiv = document.createElement('div');
    qdiv.innerHTML = `<p>${q.q}</p>`;
    q.options.forEach((opt, i)=>{
      const btn = document.createElement('button');
      btn.className='small-btn';
      btn.style.margin='6px';
      btn.innerText = opt;
      btn.onclick = ()=>{
        if(i===q.answer) score++;
        quizIndex++;
        renderQuiz();
      };
      qdiv.appendChild(btn);
    });
    quizArea.appendChild(qdiv);
  }

  // ----- Hearts reveal -----
  const heartField = document.getElementById('heartField');
  function spawnHearts(){
    heartField.innerHTML='';
    for(let i=0;i<8;i++){
      const h = document.createElement('div');
      h.className='heart';
      h.style.left = Math.random()*80 + '%';
      h.style.top = Math.random()*70 + '%';
      h.innerText = '❤';
      h.dataset.msg = ['Love you','You are my world','Always you','Miss you','Forever','My heart'][i%6];
      h.onclick = ()=>{
        alert(h.dataset.msg);
        h.style.transform='scale(0.6)';
        h.style.opacity='0.6';
      };
      heartField.appendChild(h);
    }
  }

  // ----- Confetti -----
  const confettiCanvas = document.getElementById('confettiCanvas');
  confettiCanvas.width = window.innerWidth;
  confettiCanvas.height = window.innerHeight;
  const cctx = confettiCanvas.getContext('2d');
  let confettiParticles = [];
  function launchConfetti(n=80){
    confettiParticles = [];
    for(let i=0;i<n;i++){
      confettiParticles.push({
        x: Math.random()*confettiCanvas.width,
        y: Math.random()*-confettiCanvas.height,
        r: Math.random()*6+4,
        d: Math.random()*10+2,
        color: ['#ff79b0','#ffd6e8','#fff46b','#a0e3a9'][Math.floor(Math.random()*4)],
        tilt: Math.random()*10
      });
    }
    requestAnimationFrame(updateConfetti);
    setTimeout(()=> confettiParticles = [], 4000);
  }
  function updateConfetti(){
    cctx.clearRect(0,0,confettiCanvas.width,confettiCanvas.height);
    confettiParticles.forEach((p, i)=>{
      p.y += Math.cos(p.d) + 2 + p.r/2;
      p.x += Math.sin(p.d);
      p.tilt += 0.1;
      cctx.fillStyle = p.color;
      cctx.beginPath();
      cctx.ellipse(p.x, p.y, p.r, p.r/2, p.tilt, 0, Math.PI*2);
      cctx.fill();
    });
    if(confettiParticles.length) requestAnimationFrame(updateConfetti);
  }

  document.getElementById('boomConfetti').addEventListener('click', ()=> launchConfetti(120));

  // ----- Helpers -----
  function shuffleArray(a){ for(let i=a.length-1;i>0;i--){ const j=Math.floor(Math.random()*(i+1)); [a[i],a[j]]=[a[j],a[i]]; } }
  function shuffleArrayInplace(a){ shuffleArray(a); return a; }

  // init games
  initMemory();
  renderQuiz();
  spawnHearts();
});

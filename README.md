<!DOCTYPE html>
<html lang="zh-HK">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>K3 幼兒認讀字詞小幫手 - 趣味探索遊戲</title>
  <!-- Tailwind CSS CDN -->
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- Google Fonts: 圓潤易讀中文字型 -->
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=M+PLUS+Rounded+1c:wght@500;700;800;900&family=Zen+Maru+Gothic:wght@500;700;900&display=swap" rel="stylesheet">
  
  <style>
    body {
      font-family: 'Zen Maru Gothic', 'M PLUS Rounded 1c', "PingFang HK", "Microsoft JhengHei", sans-serif;
      user-select: none;
      -webkit-user-select: none;
      -webkit-touch-callout: none;
      touch-action: manipulation;
    }

    /* 搖晃動畫（答錯/干擾物品晃動） */
    @keyframes shake {
      0%, 100% { transform: translateX(0); }
      20%, 60% { transform: translateX(-8px) rotate(-3deg); }
      40%, 80% { transform: translateX(8px) rotate(3deg); }
    }
    .shake-animation {
      animation: shake 0.4s ease-in-out;
    }

    /* 答對彈出慶祝 */
    @keyframes popSuccess {
      0% { transform: scale(0.8); opacity: 0.5; }
      50% { transform: scale(1.15); }
      100% { transform: scale(1); opacity: 1; }
    }
    .pop-success {
      animation: popSuccess 0.45s cubic-bezier(0.34, 1.56, 0.64, 1);
    }

    /* 柔和印章蓋章動畫 */
    @keyframes stampDrop {
      0% { transform: scale(3) rotate(-20deg); opacity: 0; }
      70% { transform: scale(0.9) rotate(5deg); opacity: 0.9; }
      100% { transform: scale(1) rotate(-5deg); opacity: 1; }
    }
    .stamp-animation {
      animation: stampDrop 0.5s cubic-bezier(0.175, 0.885, 0.32, 1.275) forwards;
    }

    /* 懸浮微動效果 */
    @keyframes gentleFloat {
      0%, 100% { transform: translateY(0); }
      50% { transform: translateY(-4px); }
    }
    .gentle-bounce {
      animation: gentleFloat 3s ease-in-out infinite;
    }

    /* 拖曳中樣式 */
    .dragging-active {
      opacity: 0.4;
      transform: scale(0.95);
    }
  </style>
</head>

<body class="bg-amber-50 text-slate-800 min-h-screen flex flex-col overflow-x-hidden antialiased">
  
  <!-- 頂部操作列：主題切換與重設 -->
  <header class="w-full bg-white/90 backdrop-blur shadow-md px-4 py-3 sticky top-0 z-30">
    <div class="max-w-7xl mx-auto flex flex-wrap items-center justify-between gap-3">
      <!-- 標題與語音控制 -->
      <div class="flex items-center gap-3">
        <span class="text-3xl filter drop-shadow">🌟</span>
        <div>
          <h1 class="text-xl sm:text-2xl font-black text-amber-900 tracking-wide">K3 認讀小幫手</h1>
          <p class="text-xs text-amber-700 font-bold hidden sm:block">看字詞 ‧ 找物品 ‧ 認真學好中文</p>
        </div>
        <!-- 語言切換 (粵語/普通話) -->
        <button id="lang-toggle-btn" class="ml-2 text-xs font-bold px-2.5 py-1.5 rounded-full border-2 border-amber-300 bg-amber-100/80 text-amber-900 hover:bg-amber-200 transition">
          🗣️ 發音: <span id="current-lang-text">廣東話</span>
        </button>
      </div>

      <!-- 主題切換導航鍵 -->
      <nav class="flex items-center gap-1.5 sm:gap-2 flex-wrap" id="theme-nav-container">
        <!-- 由 JavaScript 動態生成 4 個主題按鈕 -->
      </nav>

      <!-- 工具按鈕：重新開始關卡 -->
      <div class="flex items-center gap-2">
        <button id="reset-level-btn" class="flex items-center gap-1 text-sm font-bold bg-amber-200 hover:bg-amber-300 text-amber-900 px-3 py-2 rounded-2xl shadow-sm transition active:scale-95">
          <span>🔄</span> 重新挑戰
        </button>
      </div>
    </div>
  </header>

  <!-- 遊戲核心主區域 (平板橫向優化版型) -->
  <main class="flex-1 max-w-7xl w-full mx-auto p-3 sm:p-5 flex flex-col gap-4">

    <!-- 當前任務情境提示橫額 -->
    <div id="mission-banner" class="bg-gradient-to-r from-amber-400 via-orange-400 to-amber-500 rounded-3xl p-3 sm:p-4 text-white shadow-lg flex items-center justify-between transition-all">
      <div class="flex items-center gap-3">
        <span id="mission-icon" class="text-3xl sm:text-4xl bg-white/30 rounded-2xl p-2 shadow-inner">🍏</span>
        <div>
          <h2 id="mission-title" class="text-lg sm:text-2xl font-extrabold tracking-wide">食物篇：幫媽媽買食物</h2>
          <p id="mission-desc" class="text-xs sm:text-sm font-medium text-amber-100">請看左邊清單上的詞語，在貨架上找出對應的物品拖進購物籃吧！</p>
        </div>
      </div>
      <!-- 進度指示器 -->
      <div class="bg-white/20 backdrop-blur rounded-2xl px-3 py-1.5 sm:px-4 sm:py-2 text-right">
        <div class="text-xs font-bold text-amber-100">已收集</div>
        <div class="text-xl sm:text-2xl font-black text-white" id="progress-indicator">0 / 5</div>
      </div>
    </div>

    <!-- 雙欄遊戲互動區域 -->
    <div class="flex-1 grid grid-cols-1 lg:grid-cols-12 gap-4 items-start">
      
      <!-- 左欄：任務清單 (30% 佔比) -->
      <section class="lg:col-span-4 bg-white/95 rounded-3xl p-4 sm:p-5 shadow-xl border-4 border-amber-200 flex flex-col gap-3">
        <div class="flex items-center justify-between border-b-2 border-amber-100 pb-2">
          <div class="flex items-center gap-2">
            <span class="text-2xl">📝</span>
            <h3 id="list-title" class="font-black text-lg sm:text-xl text-amber-950">購物清單</h3>
          </div>
          <span class="text-xs bg-amber-100 text-amber-800 font-bold px-2 py-1 rounded-full">點小喇叭可聽音</span>
        </div>

        <!-- 5個中文詞彙清單卡 -->
        <ul id="task-checklist" class="flex flex-col gap-2.5 sm:gap-3 py-1">
          <!-- 由 JS 動態注入 5 個目標單詞列 -->
        </ul>
        
        <!-- 幼兒溫馨提示 -->
        <div class="bg-amber-50 rounded-2xl p-2.5 text-center text-xs text-amber-800 border border-amber-200/80">
          💡 <span class="font-bold">小提示：</span>點擊物品或拖動到下方框框都可以喔！
        </div>
      </section>

      <!-- 右欄：貨架探索區與放置目標籃 (70% 佔比) -->
      <section class="lg:col-span-8 flex flex-col gap-4">
        
        <!-- 貨架/場景探索區 -->
        <div class="bg-amber-100/70 border-4 border-amber-300 rounded-3xl p-4 sm:p-6 shadow-inner min-h-[320px] flex flex-col justify-between relative">
          <div class="flex items-center justify-between mb-3 text-amber-900 font-black text-sm sm:text-base">
            <span class="flex items-center gap-1.5"><span>🔍</span> <span id="shelf-label">超市貨架：請仔細找找看</span></span>
            <span class="text-xs font-semibold text-amber-700 bg-amber-200/60 px-2 py-0.5 rounded-lg">共有 8-9 件物品</span>
          </div>

          <!-- 物品網格 (含目標物品 + 干擾生活雜物) -->
          <div id="items-grid" class="grid grid-cols-3 sm:grid-cols-3 md:grid-cols-4 gap-3 sm:gap-4 my-auto">
            <!-- 由 JS 動態渲染物件方塊 -->
          </div>
        </div>

        <!-- 下方目標放置區 (可拖入或點擊自動收集) -->
        <div id="drop-zone" class="border-4 border-dashed border-amber-400 bg-white/90 hover:bg-amber-50/90 rounded-3xl p-4 sm:p-5 shadow-lg transition-all flex flex-col items-center justify-center min-h-[140px] text-center cursor-pointer select-none">
          <div class="flex items-center gap-3">
            <span id="target-zone-icon" class="text-4xl sm:text-5xl transform hover:scale-110 transition-transform">🧺</span>
            <div class="text-left">
              <h4 id="target-zone-title" class="font-black text-lg sm:text-xl text-amber-950">媽媽的購物籃</h4>
              <p id="target-zone-hint" class="text-xs sm:text-sm font-bold text-amber-600">把清單上的食物拖進來，或直接點擊正確物品！</p>
            </div>
          </div>

          <!-- 已放入籃內的收集品迷你圖示列 -->
          <div id="collected-tray" class="flex flex-wrap items-center justify-center gap-2 mt-3 w-full min-h-[36px]">
            <span class="text-xs text-gray-400 font-medium">（籃子仲係空空如也，快啲去搵啦～）</span>
          </div>
        </div>

      </section>
    </div>
  </main>

  <!-- 勝利彈窗 (通關蓋章與表揚) -->
  <div id="victory-modal" class="fixed inset-0 bg-black/60 backdrop-blur-sm z-50 flex items-center justify-center p-4 hidden">
    <div class="bg-white rounded-3xl p-6 sm:p-8 max-w-md w-full shadow-2xl border-4 border-amber-300 text-center relative flex flex-col items-center gap-4">
      
      <!-- 獎章動畫印章 -->
      <div class="w-32 h-32 rounded-full border-4 border-dashed border-red-500 bg-red-50 flex flex-col items-center justify-center shadow-lg stamp-animation text-red-600">
        <span class="text-3xl">⭐</span>
        <span class="font-black text-lg tracking-wider">優秀小幫手</span>
        <span class="text-[10px] font-bold border-t border-red-300 pt-0.5">K3 認讀達人</span>
      </div>

      <div>
        <h3 class="text-2xl sm:text-3xl font-black text-amber-950">太棒了！全部找對了！</h3>
        <p id="victory-detail-text" class="text-sm font-bold text-amber-800 mt-1">你成功認讀了所有詞語，幫媽媽買齊了食物！</p>
      </div>

      <!-- 本關字詞大複習卡 -->
      <div class="w-full bg-amber-50 rounded-2xl p-3 border border-amber-200">
        <div class="text-xs font-bold text-amber-700 mb-2">一齊再讀一次記住佢哋：</div>
        <div id="victory-word-chips" class="flex flex-wrap justify-center gap-2">
          <!-- 動態注入字詞回顧 -->
        </div>
      </div>

      <!-- 操作按鈕：下一關 / 再玩一次 -->
      <div class="flex items-center gap-3 w-full pt-2">
        <button id="modal-replay-btn" class="flex-1 py-3 rounded-2xl bg-gray-100 hover:bg-gray-200 text-gray-700 font-bold text-base transition">
          🔄 再玩一次
        </button>
        <button id="modal-next-btn" class="flex-1 py-3 rounded-2xl bg-gradient-to-r from-amber-500 to-orange-500 hover:from-amber-600 hover:to-orange-600 text-white font-extrabold text-base shadow-md transition active:scale-95">
          下一關 🚀
        </button>
      </div>
    </div>
  </div>

  <!-- 彩帶紙花特效畫布 -->
  <canvas id="confetti-canvas" class="fixed inset-0 pointer-events-none z-40"></canvas>

  <script>
    /* ====================================================================
     * 1. 遊戲資料庫設定 (4大主題、5大認讀詞彙、豐富干擾項、SVG/Emoji圖示)
     * ==================================================================== */
    const THEMES = [
      {
        id: 'food',
        name: '食物篇',
        badge: '🍏',
        title: '幫媽媽買食物',
        desc: '請看左邊清單上的詞語，在貨架上找出對應的食物放進購物籃！',
        shelfLabel: '超市食品貨架：請仔細找找看',
        targetIcon: '🧺',
        targetTitle: '媽媽的購物籃',
        targetHint: '把清單上的食物拖進來，或直接點擊正確食物！',
        victoryMsg: '你成功幫媽媽買齊了 5 樣食物，真是一個乖孩子！',
        // 5 個學習目標詞彙
        targets: [
          { id: 't_bread', name: '麵包', icon: '🍞', hint: '香噴噴的麵包' },
          { id: 't_egg', name: '雞蛋', icon: '🥚', hint: '圓滾滾的雞蛋' },
          { id: 't_apple', name: '蘋果', icon: '🍎', hint: '紅彤彤的蘋果' },
          { id: 't_milk', name: '牛奶', icon: '🥛', hint: '營養豐富的牛奶' },
          { id: 't_strawberry', name: '士多啤梨', icon: '🍓', hint: '酸酸甜甜的士多啤梨' }
        ],
        // 干擾物品（生活雜物或非清單食物）
        distractors: [
          { id: 'd_car', name: '玩具車', icon: '🚗', hint: '這是玩具車，不是食物喔' },
          { id: 'd_umbrella', name: '雨傘', icon: '☂️', hint: '這是雨傘，落雨先用' },
          { id: 'd_shoe', name: '波鞋', icon: '👟', hint: '這是鞋子，唔食得㗎' },
          { id: 'd_fish', name: '小魚', icon: '🐟', hint: '這是一條魚，清單無寫呀' }
        ]
      },
      {
        id: 'transport',
        name: '交通篇',
        badge: '🚌',
        title: '小小調度員',
        desc: '車站好忙碌！請根據調度單的詞語，找出正確的交通工具派往車站！',
        shelfLabel: '城市交通基地：找出正確的交通工具',
        targetIcon: '🚉',
        targetTitle: '出發總站 / 月台',
        targetHint: '把清單上的車船飛機調度進站，或點擊物品！',
        victoryMsg: '市民都順利出發了！你真是一個出色的交通調度員！',
        targets: [
          { id: 't_bus', name: '巴士', icon: '🚌', hint: '雙層公共巴士' },
          { id: 't_taxi', name: '的士', icon: '🚕', hint: '紅色的士' },
          { id: 't_airplane', name: '飛機', icon: '✈️', hint: '在天空中飛翔的飛機' },
          { id: 't_ship', name: '輪船', icon: '🚢', hint: '在大海裡航行的輪船' },
          { id: 't_bike', name: '單車', icon: '🚲', hint: '兩個輪子的單車' }
        ],
        distractors: [
          { id: 'd_book', name: '故事書', icon: '📚', hint: '這是書本，不是交通工具' },
          { id: 'd_pencil', name: '鉛筆', icon: '✏️', hint: '這是畫畫用的鉛筆' },
          { id: 'd_flower', name: '花朵', icon: '🌸', hint: '這是一朵花' },
          { id: 'd_chair', name: '椅子', icon: '🪑', hint: '這是坐的椅子' }
        ]
      },
      {
        id: 'animal',
        name: '動物篇',
        badge: '🦁',
        title: '動物園飼養員',
        desc: '小動物迷路了！請看名牌上的詞彙，把牠們安全護送回動物之家！',
        shelfLabel: '神秘大森林：尋找迷路的小動物',
        targetIcon: '🏡',
        targetTitle: '動物溫馨小窩',
        targetHint: '把名牌上的動物帶回家，或直接點擊小動物！',
        victoryMsg: '所有小動物都安全回家了！你真是個有愛心的飼養員！',
        targets: [
          { id: 't_tiger', name: '老虎', icon: '🐯', hint: '森林之王大老虎' },
          { id: 't_elephant', name: '大象', icon: '🐘', hint: '長長鼻子的長鼻子大象' },
          { id: 't_dog', name: '小狗', icon: '🐶', hint: '忠心可愛的小狗' },
          { id: 't_rabbit', name: '兔子', icon: '🐰', hint: '長耳朵蹦蹦跳的小兔子' },
          { id: 't_giraffe', name: '長頸鹿', icon: '🦒', hint: '高高個子的長頸鹿' }
        ],
        distractors: [
          { id: 'd_clock', name: '鬧鐘', icon: '⏰', hint: '滴答滴答的鬧鐘' },
          { id: 'd_scissors', name: '剪刀', icon: '✂️', hint: '危險的剪刀，要小心' },
          { id: 'd_cup', name: '水杯', icon: '🥛', hint: '這是飲水的水杯' },
          { id: 'd_balloon', name: '氣球', icon: '🎈', hint: '飄在空中的氣球' }
        ]
      },
      {
        id: 'clothes',
        name: '衣服篇',
        badge: '👕',
        title: '衣櫃小管家',
        desc: '要準備出門了！請根據清單上的衣服詞語，從房間中找齊放進衣櫃！',
        shelfLabel: '溫馨臥室與衣帽間：挑選正確衣物',
        targetIcon: '🚪',
        targetTitle: '小熊的整潔大衣櫃',
        targetHint: '把清單上的衣物收進衣櫃，或直接點擊衣物！',
        victoryMsg: '衣物整理得井井有條！你真是一個懂事的小管家！',
        targets: [
          { id: 't_coat', name: '外套', icon: '🧥', hint: '暖笠笠的外套' },
          { id: 't_shorts', name: '短褲', icon: '🩳', hint: '清爽舒適的短褲' },
          { id: 't_raincoat', name: '雨衣', icon: '🌧️', hint: '落雨穿的防水雨衣' },
          { id: 't_socks', name: '襪子', icon: '🧦', hint: '保護雙腳的襪子' },
          { id: 't_gloves', name: '手套', icon: '🧤', hint: '冬天保暖的手套' }
        ],
        distractors: [
          { id: 'd_spoon', name: '湯匙', icon: '🥄', hint: '吃飯用的湯匙' },
          { id: 'd_camera', name: '相機', icon: '📷', hint: '用來拍照的相機' },
          { id: 'd_football', name: '足球', icon: '⚽', hint: '運動踢的足球' },
          { id: 'd_toothbrush', name: '牙刷', icon: '🪥', hint: '刷牙用的牙刷' }
        ]
      }
    ];

    // 遊戲運行狀態
    let currentThemeIndex = 0;
    let collectedSet = new Set(); // 儲存已找到的 target id
    let isCantonese = true; // 預設廣東話 (zh-HK)，亦可切換普通話 (zh-CN)
    let draggedItemId = null; // 支援觸控與拖曳

    // 簡易 Web Audio API 音效生成器 (無需依賴外部 mp3)
    let audioCtx = null;
    function getAudioContext() {
      if (!audioCtx) {
        audioCtx = new (window.AudioContext || window.webkitAudioContext)();
      }
      if (audioCtx.state === 'suspended') {
        audioCtx.resume();
      }
      return audioCtx;
    }

    // 播放叮咚成功音 (清脆雙音階)
    function playSuccessSound() {
      try {
        const ctx = getAudioContext();
        const now = ctx.currentTime;
        const osc = ctx.createOscillator();
        const gain = ctx.createGain();
        osc.connect(gain);
        gain.connect(ctx.destination);
        
        osc.type = 'triangle';
        osc.frequency.setValueAtTime(523.25, now); // C5
        osc.frequency.setValueAtTime(659.25, now + 0.1); // E5
        osc.frequency.setValueAtTime(783.99, now + 0.2); // G5

        gain.gain.setValueAtTime(0.3, now);
        gain.gain.exponentialRampToValueAtTime(0.001, now + 0.45);

        osc.start(now);
        osc.stop(now + 0.45);
      } catch (e) {
        console.warn('Audio play restricted:', e);
      }
    }

    // 播放答錯/晃動柔和提示音 (低音鈍音，保護幼兒自尊心)
    function playErrorSound() {
      try {
        const ctx = getAudioContext();
        const now = ctx.currentTime;
        const osc = ctx.createOscillator();
        const gain = ctx.createGain();
        osc.connect(gain);
        gain.connect(ctx.destination);

        osc.type = 'sine';
        osc.frequency.setValueAtTime(260, now);
        osc.frequency.exponentialRampToValueAtTime(180, now + 0.25);

        gain.gain.setValueAtTime(0.25, now);
        gain.gain.exponentialRampToValueAtTime(0.001, now + 0.25);

        osc.start(now);
        osc.stop(now + 0.25);
      } catch (e) {
        console.warn(e);
      }
    }

    // 播放勝利通關號角
    function playVictoryFanfare() {
      try {
        const ctx = getAudioContext();
        const now = ctx.currentTime;
        const notes = [523.25, 659.25, 783.99, 1046.50]; // C5, E5, G5, C6
        notes.forEach((freq, idx) => {
          const osc = ctx.createOscillator();
          const gain = ctx.createGain();
          osc.connect(gain);
          gain.connect(ctx.destination);
          osc.type = 'triangle';
          osc.frequency.setValueAtTime(freq, now + idx * 0.12);
          gain.gain.setValueAtTime(0.3, now + idx * 0.12);
          gain.gain.exponentialRampToValueAtTime(0.001, now + idx * 0.12 + 0.35);
          osc.start(now + idx * 0.12);
          osc.stop(now + idx * 0.12 + 0.35);
        });
      } catch(e) {}
    }

    // Web Speech API 朗讀文字
    function speakText(text) {
      if (!('speechSynthesis' in window)) return;
      window.speechSynthesis.cancel(); // 停止上一段話

      const utterance = new SpeechSynthesisUtterance(text);
      utterance.lang = isCantonese ? 'zh-HK' : 'zh-CN';
      utterance.rate = 0.85; // 稍微放慢語速，適合幼兒認讀
      utterance.pitch = 1.1; // 稍微調高音調更顯親切

      // 尋找對應語系配音員
      const voices = window.speechSynthesis.getVoices();
      const targetLang = isCantonese ? 'zh-HK' : 'zh-CN';
      const matchedVoice = voices.find(v => v.lang.replace('_', '-') === targetLang || v.lang.startsWith(targetLang));
      if (matchedVoice) {
        utterance.voice = matchedVoice;
      }

      window.speechSynthesis.speak(utterance);
    }

    // 語音按鈕切換語言
    document.getElementById('lang-toggle-btn').addEventListener('click', () => {
      isCantonese = !isCantonese;
      const langText = document.getElementById('current-lang-text');
      langText.textContent = isCantonese ? '廣東話' : '普通話';
      speakText(isCantonese ? '已切換為廣東話' : '已切换为普通话');
    });

    // 初始化頂部主題選單
    function renderThemeNav() {
      const container = document.getElementById('theme-nav-container');
      container.innerHTML = '';
      
      THEMES.forEach((theme, idx) => {
        const btn = document.createElement('button');
        const isActive = idx === currentThemeIndex;
        btn.className = `px-3 py-1.5 rounded-2xl text-xs sm:text-sm font-black transition-all flex items-center gap-1.5 shadow-sm active:scale-95 ${
          isActive 
            ? 'bg-amber-500 text-white shadow-amber-300/50 scale-105 ring-2 ring-amber-400' 
            : 'bg-white hover:bg-amber-100 text-amber-900 border border-amber-200'
        }`;
        btn.innerHTML = `<span>${theme.badge}</span> <span>${theme.name}</span>`;
        btn.addEventListener('click', () => {
          if (currentThemeIndex !== idx) {
            switchTheme(idx);
          }
        });
        container.appendChild(btn);
      });
    }

    // 切換主題關卡
    function switchTheme(idx) {
      currentThemeIndex = idx;
      collectedSet.clear();
      renderThemeNav();
      loadCurrentTheme();
    }

    // 載入當前主題資料與視圖
    function loadCurrentTheme() {
      const theme = THEMES[currentThemeIndex];
      
      // 更新橫額
      document.getElementById('mission-icon').textContent = theme.badge;
      document.getElementById('mission-title').textContent = `${theme.name}：${theme.title}`;
      document.getElementById('mission-desc').textContent = theme.desc;
      document.getElementById('shelf-label').textContent = theme.shelfLabel;

      // 更新目標放置區
      document.getElementById('target-zone-icon').textContent = theme.targetIcon;
      document.getElementById('target-zone-title').textContent = theme.targetTitle;
      document.getElementById('target-zone-hint').textContent = theme.targetHint;

      // 清空托盤
      updateProgress();
      renderTaskList();
      renderShelfItems();

      // 關閉勝利視窗
      document.getElementById('victory-modal').classList.add('hidden');
    }

    // 渲染左側 5 個目標詞彙清單
    function renderTaskList() {
      const theme = THEMES[currentThemeIndex];
      const listEl = document.getElementById('task-checklist');
      listEl.innerHTML = '';

      theme.targets.forEach(target => {
        const isDone = collectedSet.has(target.id);
        const li = document.createElement('li');
        li.id = `task-row-${target.id}`;
        li.className = `flex items-center justify-between p-2.5 sm:p-3 rounded-2xl border-2 transition-all ${
          isDone 
            ? 'bg-emerald-50 border-emerald-400 text-emerald-900 shadow-sm' 
            : 'bg-white hover:bg-amber-50/70 border-amber-200 text-slate-800'
        }`;

        li.innerHTML = `
          <div class="flex items-center gap-3">
            <!-- 核對勾選框 -->
            <div class="w-7 h-7 sm:w-8 sm:h-8 rounded-xl flex items-center justify-center font-bold text-lg border-2 transition-all ${
              isDone ? 'bg-emerald-500 border-emerald-600 text-white' : 'border-amber-300 bg-amber-50 text-transparent'
            }">
              ✓
            </div>
            <!-- 中文大字詞彙 (標楷/圓體超清晰展示) -->
            <span class="text-xl sm:text-2xl font-black tracking-widest ${isDone ? 'line-through opacity-70' : ''}">
              ${target.name}
            </span>
          </div>

          <!-- 語音朗讀小喇叭 -->
          <button type="button" class="speaker-btn p-2 rounded-xl bg-amber-100 hover:bg-amber-200 active:scale-90 text-amber-900 transition" title="點擊聽讀音">
            <span class="text-xl">🔊</span>
          </button>
        `;

        // 綁定喇叭點擊語音事件
        const speakerBtn = li.querySelector('.speaker-btn');
        speakerBtn.addEventListener('click', (e) => {
          e.stopPropagation();
          speakText(target.name);
        });

        // 點擊整列亦可播放讀音加強學習
        li.addEventListener('click', () => {
          speakText(target.name);
        });

        listEl.appendChild(li);
      });
    }

    // 渲染貨架區物品（包含目標與干擾項目打亂排布）
    function renderShelfItems() {
      const theme = THEMES[currentThemeIndex];
      const gridEl = document.getElementById('items-grid');
      gridEl.innerHTML = '';

      // 組合 5 個目標物 + 干擾物
      let allItems = [];
      theme.targets.forEach(t => {
        allItems.push({ ...t, isTarget: true });
      });
      theme.distractors.forEach(d => {
        allItems.push({ ...d, isTarget: false });
      });

      // 洗牌隨機排列
      allItems = shuffleArray(allItems);

      allItems.forEach(item => {
        const isAlreadyCollected = collectedSet.has(item.id);

        const card = document.createElement('div');
        card.id = `item-card-${item.id}`;
        card.dataset.itemId = item.id;
        card.dataset.isTarget = item.isTarget;
        card.dataset.name = item.name;
        card.draggable = !isAlreadyCollected;

        card.className = `flex flex-col items-center justify-center p-3 sm:p-4 rounded-3xl bg-white border-3 border-amber-200 shadow-md transition-all cursor-grab active:cursor-grabbing select-none hover:shadow-xl hover:-translate-y-1 ${
          isAlreadyCollected ? 'opacity-25 pointer-events-none scale-90' : ''
        }`;

        card.innerHTML = `
          <div class="text-4xl sm:text-5xl mb-1 filter drop-shadow transform transition-transform hover:scale-110">
            ${item.icon}
          </div>
          <span class="text-xs sm:text-sm font-bold text-gray-500 mt-1">${item.name}</span>
        `;

        // 綁定拖曳事件 (電腦端與觸控端)
        bindDragAndClickEvents(card, item);

        gridEl.appendChild(card);
      });
    }

    // 隨機打亂陣列輔助函數 (Fisher-Yates)
    function shuffleArray(arr) {
      const newArr = [...arr];
      for (let i = newArr.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [newArr[i], newArr[j]] = [newArr[j], newArr[i]];
      }
      return newArr;
    }

    function bindDragAndClickEvents(card, item) {
      // 1. 滑鼠原生拖曳
      card.addEventListener('dragstart', (e) => {
        draggedItemId = item.id;
        card.classList.add('dragging-active');
        e.dataTransfer.setData('text/plain', item.id);
      });

      card.addEventListener('dragend', () => {
        card.classList.remove('dragging-active');
      });

      // 2. 幼兒友好點擊（直接點擊物件亦可送入籃子檢查，方便手部肌肉未熟練的小朋友）
      card.addEventListener('click', (e) => {
        e.preventDefault();
        handleItemSelection(item, card);
      });

      // 3. 觸控平板支援 (Touch Drag 視覺模擬)
      let touchMoved = false;
      card.addEventListener('touchstart', (e) => {
        touchMoved = false;
        draggedItemId = item.id;
      }, { passive: true });

      card.addEventListener('touchmove', (e) => {
        touchMoved = true;
      }, { passive: true });

      card.addEventListener('touchend', (e) => {
        // 若在目標區域上方釋放
        const touch = e.changedTouches[0];
        const dropZone = document.getElementById('drop-zone');
        const rect = dropZone.getBoundingClientRect();

        if (
          touch.clientX >= rect.left &&
          touch.clientX <= rect.right &&
          touch.clientY >= rect.top &&
          touch.clientY <= rect.bottom
        ) {
          handleItemSelection(item, card);
        } else if (!touchMoved) {
          // 純點擊
          handleItemSelection(item, card);
        }
        draggedItemId = null;
      });
    }

    // 目標區域放下事件 (Drop Zone)
    const dropZone = document.getElementById('drop-zone');
    dropZone.addEventListener('dragover', (e) => {
      e.preventDefault();
      dropZone.classList.add('bg-amber-100', 'border-amber-600', 'scale-[1.02]');
    });

    dropZone.addEventListener('dragleave', () => {
      dropZone.classList.remove('bg-amber-100', 'border-amber-600', 'scale-[1.02]');
    });

    dropZone.addEventListener('drop', (e) => {
      e.preventDefault();
      dropZone.classList.remove('bg-amber-100', 'border-amber-600', 'scale-[1.02]');
      const itemId = e.dataTransfer.getData('text/plain') || draggedItemId;
      if (!itemId) return;

      const theme = THEMES[currentThemeIndex];
      const item = [...theme.targets, ...theme.distractors].find(i => i.id === itemId);
      const card = document.getElementById(`item-card-${itemId}`);
      if (item && card) {
        handleItemSelection(item, card);
      }
    });

    // 統一處理判定選取的物品
    function handleItemSelection(item, card) {
      if (collectedSet.has(item.id)) return; // 已選過

      const theme = THEMES[currentThemeIndex];

      if (item.isTarget) {
        // --- 答對了！---
        collectedSet.add(item.id);
        playSuccessSound();
        speakText(`搵啱喇！係「${item.name}」！`);

        // 貨架卡片淡出標記
        card.classList.add('opacity-25', 'pointer-events-none', 'scale-90');
        card.draggable = false;

        // 目標放置籃加入可愛縮圖
        addCollectedBadge(item);

        // 刷新清單狀態
        renderTaskList();
        updateProgress();

        // 檢查是否通關
        if (collectedSet.size === theme.targets.length) {
          setTimeout(() => {
            triggerVictory();
          }, 600);
        }

      } else {
        // --- 揀錯干擾項目 ---
        playErrorSound();
        speakText(`呢個係「${item.name}」，清單上面無呀，再試下啦！`);
        
        // 執行晃動動畫 (Shake)
        card.classList.remove('shake-animation');
        void card.offsetWidth; // 強制重繪重啟 CSS 動畫
        card.classList.add('shake-animation');
      }
    }

    // 在目標籃內展示已收集的小徽章
    function addCollectedBadge(item) {
      const tray = document.getElementById('collected-tray');
      if (collectedSet.size === 1) {
        tray.innerHTML = ''; // 清空原本的「空空如也」提示
      }

      const badge = document.createElement('div');
      badge.className = 'flex items-center gap-1.5 bg-amber-100 text-amber-950 font-bold px-3 py-1.5 rounded-2xl border-2 border-amber-300 pop-success text-sm shadow-sm';
      badge.innerHTML = `<span>${item.icon}</span> <span>${item.name}</span>`;
      tray.appendChild(badge);
    }

    // 更新進度條
    function updateProgress() {
      const theme = THEMES[currentThemeIndex];
      const indicator = document.getElementById('progress-indicator');
      indicator.textContent = `${collectedSet.size} / ${theme.targets.length}`;

      if (collectedSet.size === 0) {
        const tray = document.getElementById('collected-tray');
        tray.innerHTML = `<span class="text-xs text-gray-400 font-medium">（籃子仲係空空如也，快啲去搵啦～）</span>`;
      }
    }

    // 勝利表揚邏輯
    function triggerVictory() {
      const theme = THEMES[currentThemeIndex];
      playVictoryFanfare();
      speakText(`太叻仔喇！你幫手搵齊晒 5 樣嘢！全部答啱！`);

      // 啟動紙花特效
      launchConfetti();

      // 填充彈窗字詞總結
      const detailText = document.getElementById('victory-detail-text');
      detailText.textContent = theme.victoryMsg;

      const chipsContainer = document.getElementById('victory-word-chips');
      chipsContainer.innerHTML = '';
      theme.targets.forEach(t => {
        const chip = document.createElement('button');
        chip.className = 'px-3 py-1.5 bg-white border-2 border-amber-300 text-amber-900 rounded-2xl text-base font-black shadow-sm flex items-center gap-1 hover:bg-amber-100 transition active:scale-95';
        chip.innerHTML = `<span>${t.icon}</span> <span>${t.name}</span> <span>🔊</span>`;
        chip.addEventListener('click', () => speakText(t.name));
        chipsContainer.appendChild(chip);
      });

      // 顯示彈窗
      const modal = document.getElementById('victory-modal');
      modal.classList.remove('hidden');
    }

    // 紙花粒子動畫引擎 (Canvas Confetti)
    function launchConfetti() {
      const canvas = document.getElementById('confetti-canvas');
      const ctx = canvas.getContext('2d');
      canvas.width = window.innerWidth;
      canvas.height = window.innerHeight;

      const particles = [];
      const colors = ['#f59e0b', '#ef4444', '#10b981', '#3b82f6', '#ec4899', '#8b5cf6'];

      for (let i = 0; i < 100; i++) {
        particles.push({
          x: canvas.width / 2,
          y: canvas.height / 2,
          r: Math.random() * 6 + 4,
          dx: (Math.random() - 0.5) * 16,
          dy: (Math.random() - 0.7) * 16,
          color: colors[Math.floor(Math.random() * colors.length)],
          tilt: Math.random() * 10,
          tiltAngle: 0,
          tiltAngleInc: (Math.random() * 0.08) + 0.05
        });
      }

      let animationFrame;
      let frameCount = 0;

      function renderParticles() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        frameCount++;

        particles.forEach(p => {
          p.x += p.dx;
          p.y += p.dy;
          p.dy += 0.35; // 重力
          p.tiltAngle += p.tiltAngleInc;
          p.tilt = Math.sin(p.tiltAngle) * 15;

          ctx.beginPath();
          ctx.lineWidth = p.r;
          ctx.strokeStyle = p.color;
          ctx.moveTo(p.x + p.tilt + (p.r / 2), p.y);
          ctx.lineTo(p.x + p.tilt, p.y + p.tilt + (p.r / 2));
          ctx.stroke();
        });

        if (frameCount < 160) {
          animationFrame = requestAnimationFrame(renderParticles);
        } else {
          ctx.clearRect(0, 0, canvas.width, canvas.height);
          cancelAnimationFrame(animationFrame);
        }
      }
      renderParticles();
    }

    // 視窗大小改變自適應畫布
    window.addEventListener('resize', () => {
      const canvas = document.getElementById('confetti-canvas');
      canvas.width = window.innerWidth;
      canvas.height = window.innerHeight;
    });

    // 重設本關按鈕
    document.getElementById('reset-level-btn').addEventListener('click', () => {
      collectedSet.clear();
      loadCurrentTheme();
      speakText('重新開始，一齊加油！');
    });

    // 彈窗再玩一次
    document.getElementById('modal-replay-btn').addEventListener('click', () => {
      collectedSet.clear();
      loadCurrentTheme();
    });

    // 彈窗下一關
    document.getElementById('modal-next-btn').addEventListener('click', () => {
      const nextIdx = (currentThemeIndex + 1) % THEMES.length;
      switchTheme(nextIdx);
    });

    // 預加載語音包
    if ('speechSynthesis' in window) {
      window.speechSynthesis.onvoiceschanged = () => {
        window.speechSynthesis.getVoices();
      };
    }

    // 頁面初次載入
    window.addEventListener('DOMContentLoaded', () => {
      renderThemeNav();
      loadCurrentTheme();
    });
  </script>
</body>
</html>

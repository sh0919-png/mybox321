<!doctype html>
<html lang="ko" class="h-full">
 <head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <script src="https://cdn.tailwindcss.com/3.4.17"></script>
  <script src="https://cdn.jsdelivr.net/npm/lucide@0.263.0/dist/umd/lucide.min.js"></script>
  <script src="/_sdk/element_sdk.js"></script>
  <script src="/_sdk/data_sdk.js"></script>
  <link href="https://fonts.googleapis.com/css2?family=Outfit:wght@300;400;500;600;700&amp;display=swap" rel="stylesheet">
  <style>
    * { font-family: 'Outfit', sans-serif; }
    body { box-sizing: border-box; margin: 0; }
    .icon-circle { transition: transform 0.3s ease; }
    .home-icon-link:hover .icon-circle { transform: scale(1.1); }
    .home-icon-link:hover i { animation: wiggle 0.4s ease; }
    @keyframes wiggle { 0%,100%{transform:rotate(0)} 25%{transform:rotate(-8deg)} 75%{transform:rotate(8deg)} }
    @keyframes spin { from{transform:rotate(0deg)} to{transform:rotate(360deg)} }
    .toast-animation { animation: slideUp 0.3s ease forwards; }
    @keyframes slideUp { from{opacity:0;transform:translateY(10px)} to{opacity:1;transform:translateY(0)} }
  </style>
 </head>
 <body class="h-full">
  <div id="app" class="h-full w-full overflow-auto" style="background-color: #0f172a;">
   <div class="flex flex-col items-center justify-center min-h-full px-4 py-12">
    <h1 id="page-title" class="text-3xl font-bold mb-12 tracking-tight" style="color: #f8fafc;">My Homepage</h1><div class="flex gap-10 mb-16 flex-wrap justify-center">
     <div class="home-icon-link flex flex-col items-center gap-3 cursor-pointer">
      <div class="icon-circle w-20 h-20 rounded-2xl flex items-center justify-center" style="background-color: #1e293b;">
       <i data-lucide="home" style="width:36px;height:36px;color:#38bdf8;"></i>
      </div><span class="text-sm font-medium" style="color: #94a3b8;">Home</span>
     </div>
     <div class="home-icon-link flex flex-col items-center gap-3 cursor-pointer">
      <div class="icon-circle w-20 h-20 rounded-2xl flex items-center justify-center" style="background-color: #1e293b;">
       <i data-lucide="user" style="width:36px;height:36px;color:#38bdf8;"></i>
      </div><span class="text-sm font-medium" style="color: #94a3b8;">Profile</span>
     </div>
     <div class="home-icon-link flex flex-col items-center gap-3 cursor-pointer">
      <div class="icon-circle w-20 h-20 rounded-2xl flex items-center justify-center" style="background-color: #1e293b;">
       <i data-lucide="settings" style="width:36px;height:36px;color:#38bdf8;"></i>
      </div><span class="text-sm font-medium" style="color: #94a3b8;">Setting</span>
     </div>
    </div><div class="w-full max-w-md rounded-2xl p-6 mb-8" style="background-color: #1e293b;">
     <h2 id="msg-title" class="text-lg font-semibold mb-5" style="color: #f8fafc;">메시지 보내기</h2>
     <form id="msg-form" class="flex flex-col gap-4">
      <div>
       <label for="name-input" class="block text-sm mb-1.5 font-medium" style="color: #94a3b8;">이름</label> <input id="name-input" type="text" placeholder="이름을 입력하세요" class="w-full px-4 py-2.5 rounded-lg text-sm outline-none focus:ring-2 focus:ring-sky-400" style="background-color: #0f172a; color: #f8fafc; border: 1px solid #334155;">
      </div>
      <div>
       <label for="msg-input" class="block text-sm mb-1.5 font-medium" style="color: #94a3b8;">내용</label> <textarea id="msg-input" rows="4" placeholder="내용을 입력하세요" class="w-full px-4 py-2.5 rounded-lg text-sm outline-none resize-none focus:ring-2 focus:ring-sky-400" style="background-color: #0f172a; color: #f8fafc; border: 1px solid #334155;"></textarea>
      </div><button id="send-btn" type="submit" class="w-full py-2.5 rounded-lg text-sm font-semibold transition-colors" style="background-color: #38bdf8; color: #0f172a;">보내기</button>
     </form>
     <div id="toast" class="hidden mt-4 px-4 py-2.5 rounded-lg text-sm font-medium text-center"></div>
    </div><div class="w-full max-w-md rounded-2xl p-6" style="background-color: #1e293b;">
     <div class="flex justify-between items-center mb-5">
      <h2 id="replies-title" class="text-lg font-semibold" style="color: #f8fafc;">답변 목록</h2><button id="refresh-btn" type="button" class="p-2 rounded-lg transition-colors" style="background-color: #334155; color: #38bdf8;" title="새로고침"> <i data-lucide="rotate-cw" style="width:18px;height:18px;"></i> </button>
     </div>
     <div id="replies-list" class="flex flex-col gap-4">
      <p class="text-sm text-slate-500 italic text-center py-4">데이터 로딩 중...</p>
     </div>
    </div>
   </div>
  </div>
  <script>
    lucide.createIcons();

    const defaultConfig = {
      background_color: '#0f172a',
      surface_color: '#1e293b',
      text_color: '#f8fafc',
      primary_action_color: '#38bdf8',
      secondary_text_color: '#94a3b8',
      font_family: 'Outfit',
      font_size: 16,
      page_title: 'My Homepage',
      message_section_title: '메시지 보내기'
    };

    let currentData = [];

    // Element SDK 초기화
    window.elementSdk.init({
      defaultConfig,
      onConfigChange: (config) => {
        const bg = config.background_color || defaultConfig.background_color;
        const surface = config.surface_color || defaultConfig.surface_color;
        const text = config.text_color || defaultConfig.text_color;
        const accent = config.primary_action_color || defaultConfig.primary_action_color;
        const secText = config.secondary_text_color || defaultConfig.secondary_text_color;
        const font = config.font_family || defaultConfig.font_family;
        const size = config.font_size || defaultConfig.font_size;

        document.getElementById('app').style.backgroundColor = bg;
        document.querySelectorAll('.icon-circle').forEach(el => { el.style.backgroundColor = surface; });
        document.querySelectorAll('[data-lucide]').forEach(el => { el.style.color = accent; });
        document.querySelectorAll('.home-icon-link span').forEach(el => { el.style.color = secText; });
        
        const titleEl = document.getElementById('page-title');
        titleEl.textContent = config.page_title || defaultConfig.page_title;
        titleEl.style.color = text;
        titleEl.style.fontSize = `${size * 1.875}px`;

        const msgTitle = document.getElementById('msg-title');
        msgTitle.textContent = config.message_section_title || defaultConfig.message_section_title;
        msgTitle.style.color = text;

        document.getElementById('replies-title').style.color = text;

        document.querySelectorAll('label').forEach(l => { l.style.color = secText; });
        document.querySelectorAll('input, textarea').forEach(el => {
          el.style.backgroundColor = bg;
          el.style.color = text;
        });

        const btn = document.getElementById('send-btn');
        btn.style.backgroundColor = accent;
        btn.style.color = bg;

        updateRepliesStyles(bg, surface, text, accent, secText, font, size);
      }
    });

    function updateRepliesStyles(bg, surface, text, accent, secText, font, size) {
      document.querySelectorAll('[data-reply-item]').forEach(el => {
        el.style.backgroundColor = bg;
        el.style.borderLeftColor = accent;
      });
      document.querySelectorAll('[data-reply-name]').forEach(el => { el.style.color = accent; });
      document.querySelectorAll('[data-reply-date]').forEach(el => { el.style.color = secText; });
      document.querySelectorAll('[data-reply-user-msg]').forEach(el => { el.style.color = secText; });
      document.querySelectorAll('[data-reply-admin-text]').forEach(el => { el.style.color = text; });
    }

    // 렌더링 함수: 구글 시트의 데이터를 화면에 송출
    function renderReplies(data) {
      const repliesList = document.getElementById('replies-list');
      if (!data || data.length === 0) {
        repliesList.innerHTML = '<p class="text-sm italic text-center py-4" style="color: #64748b;">메시지가 없습니다.</p>';
        return;
      }

      // 최신순 정렬
      const sortedData = [...data].sort((a, b) => new Date(b.created_at) - new Date(a.created_at));
      repliesList.innerHTML = '';

      sortedData.forEach((record) => {
        const container = document.createElement('div');
        container.setAttribute('data-reply-item', 'true');
        container.className = 'p-4 rounded-lg flex flex-col gap-2 border-l-4';
        container.style.backgroundColor = '#0f172a';
        container.style.borderLeftColor = '#38bdf8';

        // 헤더: 이름 및 날짜
        const header = document.createElement('div');
        header.className = 'flex justify-between items-center';
        
        const name = document.createElement('span');
        name.setAttribute('data-reply-name', 'true');
        name.className = 'text-xs font-bold uppercase';
        name.style.color = '#38bdf8';
        name.textContent = record.name || '익명';

        const date = document.createElement('span');
        date.setAttribute('data-reply-date', 'true');
        date.className = 'text-[10px]';
        date.style.color = '#94a3b8';
        date.textContent = record.created_at ? new Date(record.created_at).toLocaleDateString() : '';

        header.appendChild(name);
        header.appendChild(date);

        // 사용자가 보낸 원래 메시지 (작게 표시)
        const userMsg = document.createElement('p');
        userMsg.setAttribute('data-reply-user-msg', 'true');
        userMsg.className = 'text-xs italic opacity-70';
        userMsg.style.color = '#94a3b8';
        userMsg.textContent = `Q: ${record.message}`;

        // 관리자의 답변 (구글 시트의 reply 항목)
        const adminReply = document.createElement('div');
        adminReply.className = 'mt-1 p-2 rounded bg-slate-800/50';
        
        const replyLabel = document.createElement('span');
        replyLabel.className = 'text-[10px] font-bold block mb-1';
        replyLabel.style.color = '#38bdf8';
        replyLabel.textContent = 'A. ANSWER';

        const replyText = document.createElement('p');
        replyText.setAttribute('data-reply-admin-text', 'true');
        replyText.className = 'text-sm leading-relaxed whitespace-pre-wrap break-words';
        replyText.style.color = '#f8fafc';
        // 구글 시트의 'reply' 탭(열)의 내용을 가져옴
        replyText.textContent = record.reply && record.reply.trim() !== "" ? record.reply : '답변을 기다리는 중입니다...';

        adminReply.appendChild(replyLabel);
        adminReply.appendChild(replyText);

        container.appendChild(header);
        container.appendChild(userMsg);
        container.appendChild(adminReply);
        repliesList.appendChild(container);
      });
    }

    // Data SDK 연동
    const dataHandler = {
      onDataChanged(newData) {
        currentData = [...newData];
        renderReplies(currentData);
      }
    };

    (async () => {
      const res = await window.dataSdk.init(dataHandler);
      if (res.isOk) {
        // 초기 로드 시 시트의 데이터를 가져와 렌더링
        const initialData = await window.dataSdk.list();
        if(initialData.isOk) {
            currentData = initialData.data;
            renderReplies(currentData);
        }
      }
    })();

    // 메시지 전송 로직
    document.getElementById('msg-form').addEventListener('submit', async (e) => {
      e.preventDefault();
      const name = document.getElementById('name-input').value.trim();
      const message = document.getElementById('msg-input').value.trim();

      if (!name || !message) {
        showToast('이름과 내용을 입력해주세요.', true);
        return;
      }

      const btn = document.getElementById('send-btn');
      btn.disabled = true;
      btn.textContent = '전송 중...';

      // 구글 시트에 데이터 추가 (reply는 비워둔 채로 생성)
      const result = await window.dataSdk.create({
        name: name,
        message: message,
        reply: '', 
        created_at: new Date().toISOString()
      });

      btn.disabled = false;
      btn.textContent = '보내기';

      if (result.isOk) {
        document.getElementById('name-input').value = '';
        document.getElementById('msg-input').value = '';
        showToast('메시지가 전송되었습니다! 답변을 기다려주세요.');
        // 시트 데이터 즉시 새로고침
        const updated = await window.dataSdk.list();
        if(updated.isOk) renderReplies(updated.data);
      } else {
        showToast('전송 실패. 시트 연결을 확인하세요.', true);
      }
    });

    // 새로고침 버튼
    document.getElementById('refresh-btn').addEventListener('click', async () => {
      const icon = document.querySelector('#refresh-btn i');
      icon.style.animation = 'spin 1s linear infinite';
      const res = await window.dataSdk.list();
      if(res.isOk) {
        currentData = res.data;
        renderReplies(currentData);
      }
      setTimeout(() => { icon.style.animation = 'none'; }, 1000);
    });

    function showToast(msg, isError) {
      const toast = document.getElementById('toast');
      toast.textContent = msg;
      toast.style.backgroundColor = isError ? '#7f1d1d' : '#065f46';
      toast.style.color = isError ? '#fca5a5' : '#6ee7b7';
      toast.classList.remove('hidden');
      setTimeout(() => { toast.classList.add('hidden'); }, 3000);
    }
  </script>
 </body>
</html>

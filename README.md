<DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Каталог документов</title>
  <style>
    /* Нежно-красная палитра */
    :root {
      --bg: #fff5f5;
      --surface: #ffffff;
      --primary: #e57373;
      --primary-dark: #c62828;
      --primary-soft: #ffcdd2;
      --text: #3e2723;
      --text-muted: #7a5c5c;
      --shadow: 0 8px 24px rgba(198, 40, 40, 0.08);
      --radius: 16px;
    }

    * {
      box-sizing: border-box;
    }

    body {
      margin: 0;
      min-height: 100vh;
      font-family: system-ui, -apple-system, "Segoe UI", Roboto, Arial, sans-serif;
      background:
        radial-gradient(circle at top left, rgba(229, 115, 115, 0.18), transparent 32%),
        radial-gradient(circle at bottom right, rgba(255, 205, 210, 0.35), transparent 38%),
        var(--bg);
      color: var(--text);
      line-height: 1.5;
    }

    .page {
      width: min(1100px, 92%);
      margin: 0 auto;
      padding: 48px 0 64px;
    }

    header {
      text-align: center;
      margin-bottom: 40px;
    }

    h1 {
      margin: 0 0 12px;
      font-size: clamp(1.8rem, 4vw, 2.8rem);
      letter-spacing: -0.03em;
      color: var(--primary-dark);
    }

    .subtitle {
      margin: 0;
      color: var(--text-muted);
      font-size: 1.05rem;
    }

    .catalog {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
      gap: 20px;
      align-items: start;
    }

    .card {
      display: flex;
      flex-direction: column;
      gap: 10px;
      padding: 22px;
      background-color: var(--surface);
      background-image: linear-gradient(
        90deg,
        rgba(229, 115, 115, 0.8),
        rgba(255, 205, 210, 0.8)
      );
      background-repeat: no-repeat;
      background-size: 100% 5px;
      background-position: top left;
      border: 1px solid rgba(229, 115, 115, 0.25);
      border-radius: var(--radius);
      box-shadow: var(--shadow);
      text-decoration: none;
      color: inherit;
      transition: transform 0.2s ease, box-shadow 0.2s ease, border-color 0.2s ease;
      position: relative;
    }

    .card:hover,
    .card:focus-visible {
      transform: translateY(-4px);
      border-color: var(--primary);
      box-shadow: 0 14px 32px rgba(198, 40, 40, 0.14);
      outline: none;
    }

    /* Карточка Вестника: при открытом виджете поднимаем z-index,
       чтобы панель легла поверх соседних карточек */
    .card-vestnik {
      position: relative;
    }
    .card-vestnik.is-open {
      z-index: 10;
    }

    .card-header {
      display: flex;
      align-items: center;
      gap: 10px;
      margin-top: 6px;
    }

    .card-favicon {
      width: 28px;
      height: 28px;
      border-radius: 8px;
      background: var(--primary-soft);
      padding: 3px;
      object-fit: contain;
      flex-shrink: 0;
    }

    .card-title {
      margin: 0;
      font-size: 1.15rem;
      font-weight: 700;
      color: var(--primary-dark);
    }

    .card-desc {
      margin: 0;
      color: var(--text-muted);
      font-size: 0.95rem;
      flex-grow: 1;
    }

    .card-link {
      display: inline-flex;
      align-items: center;
      gap: 6px;
      margin-top: 6px;
      font-size: 0.9rem;
      font-weight: 600;
      color: var(--primary-dark);
      text-decoration: none;
      width: fit-content;
    }

    .card-link::after {
      content: "→";
      transition: transform 0.2s ease;
    }

    .card:hover .card-link::after {
      transform: translateX(4px);
    }

    footer {
      margin-top: 48px;
      text-align: center;
      color: var(--text-muted);
      font-size: 0.9rem;
    }

    /* ===== Виджет «Документы по годам» внутри карточки Вестника ===== */
    .docs-widget {
      position: relative;
      margin-top: 6px;
    }

    .docs-summary {
      display: flex;
      align-items: center;
      gap: 8px;
      padding: 10px 14px;
      background: var(--primary-soft);
      color: var(--primary-dark);
      border-radius: 10px;
      font-weight: 600;
      font-size: 0.88rem;
      cursor: pointer;
      user-select: none;
      outline: none;
    }
    .docs-summary:hover,
    .docs-summary:focus-visible {
      background: #ffb8bd;
    }

    .docs-summary::after {
      content: "▾";
      margin-left: auto;
      transition: transform 0.2s ease;
      font-size: 0.9rem;
    }
    .docs-widget.open .docs-summary::after {
      transform: rotate(180deg);
    }

    /* Красная точка — есть новый документ */
    .new-badge {
      display: none;
      width: 8px;
      height: 8px;
      background: #e53935;
      border-radius: 50%;
      box-shadow: 0 0 0 2px var(--primary-soft);
    }
    .new-badge.active {
      display: inline-block;
    }

    .panel-body {
      position: absolute;
      left: 0;
      right: 0;
      top: calc(100% + 6px);
      background: var(--surface);
      border: 1px solid rgba(229, 115, 115, 0.25);
      border-radius: var(--radius);
      box-shadow: 0 14px 32px rgba(198, 40, 40, 0.18);
      padding: 16px;
      opacity: 0;
      visibility: hidden;
      transform: translateY(-4px);
      transition: opacity 0.15s ease, visibility 0.15s ease, transform 0.15s ease;
    }
    .docs-widget.open .panel-body {
      opacity: 1;
      visibility: visible;
      transform: translateY(0);
    }

    .panel-body select {
      width: 100%;
      margin-bottom: 12px;
      padding: 6px 8px;
      border-radius: 8px;
      border: 1px solid rgba(229, 115, 115, 0.4);
      background: #fff;
      font: inherit;
    }

    #document-list {
      list-style: none;
      padding: 0;
      margin: 0;
      max-height: 240px;
      overflow-y: auto;
    }
    #document-list li {
      margin-bottom: 6px;
    }
    #document-list a {
      color: var(--text);
      text-decoration: none;
      display: block;
      padding: 4px 6px;
      border-radius: 6px;
      word-break: break-word;
      font-size: 0.85rem;
    }
    #document-list a:hover {
      background: var(--primary-soft);
    }

    .new-doc-notice {
      margin-top: 12px;
      padding: 10px;
      background: #fff3cd;
      border: 1px solid #ffeeba;
      border-radius: 8px;
      color: #856404;
      font-size: 0.82rem;
      display: none;
    }
    .new-doc-notice a {
      color: #c62828;
      font-weight: 600;
    }

    @media (max-width: 520px) {
      .page {
        padding-top: 32px;
      }
      .card {
        padding: 18px;
      }
      /* На мобильных — панель раскрывает карточку, а не всплывает */
      .docs-widget .panel-body {
        position: static;
        margin-top: 8px;
        transform: none;
        display: none;
      }
      .docs-widget.open .panel-body {
        display: block;
        opacity: 1;
        visibility: visible;
      }
    }
  </style>
</head>
<body>
  <div class="page">
    <header>
      <h1>Каталог документов для работы</h1>
      <p class="subtitle"></p>
    </header>

    <main class="catalog">
      <!-- Центр 2016 -->
      <a class="card" href="https://fhmoscow.com/team/604?season=5&amp;team-inner-tab=3" target="_blank" rel="noopener noreferrer">
        <div class="card-header">
          <img class="card-favicon"
               src="https://icons.duckduckgo.com/ip3/fhmoscow.com.ico"
               alt="" loading="lazy" width="28" height="28"
               onerror="this.style.display='none'">
          <span class="card-title">Центр 2016</span>
        </div>
        <p class="card-desc">Страница команды на сайте ФХМ.</p>
        <span class="card-link">Открыть документ</span>
      </a>

      <!-- Вестник Москвы — карточка с виджетом документов по годам -->
      <div class="card card-vestnik" id="card-vestnik">
        <div class="card-header">
          <img class="card-favicon"
               src="https://icons.duckduckgo.com/ip3/vestnikmoscow.mos.ru.ico"
               alt="" loading="lazy" width="28" height="28"
               onerror="this.style.display='none'">
          <span class="card-title">Вестник Москвы</span>
        </div>
        <p class="card-desc">Переход на сайт Вестник Москвы.</p>
        <a class="card-link" href="https://vestnikmoscow.mos.ru/" target="_blank" rel="noopener noreferrer">Открыть документ</a>

        <!-- Виджет «Документы по годам» -->
        <div class="docs-widget" id="docs-widget">
          <div class="docs-summary"
               id="docs-summary"
               role="button"
               tabindex="0"
               aria-expanded="false"
               aria-controls="docs-panel-body">
            📚 Документы по годам
            <span class="new-badge" id="new-badge" title="Есть новый документ"></span>
          </div>
          <div class="panel-body" id="docs-panel-body">
            <select id="year-select"></select>
            <ul id="document-list"></ul>
            <div id="new-doc-notice" class="new-doc-notice"></div>
          </div>
        </div>
      </div>

      <!-- Документ 3 -->
      <a class="card" href="https://example.com/document-3.pdf" target="_blank" rel="noopener noreferrer">
        <div class="card-header">
          <img class="card-favicon"
               src="https://icons.duckduckgo.com/ip3/example.com.ico"
               alt="" loading="lazy" width="28" height="28"
               onerror="this.style.display='none'">
          <span class="card-title">Документ 3</span>
        </div>
        <p class="card-desc">Пояснение для третьего документа. Ссылка ведёт на внешний ресурс.</p>
        <span class="card-link">Открыть документ</span>
      </a>

      <!-- Документ 4 -->
      <a class="card" href="https://example.com/document-4.pdf" target="_blank" rel="noopener noreferrer">
        <div class="card-header">
          <img class="card-favicon"
               src="https://icons.duckduckgo.com/ip3/example.com.ico"
               alt="" loading="lazy" width="28" height="28"
               onerror="this.style.display='none'">
          <span class="card-title">Документ 4</span>
        </div>
        <p class="card-desc">Здесь может быть описание инструкции, регламента или справочного файла.</p>
        <span class="card-link">Открыть документ</span>
      </a>

      <!-- Документ 5 -->
      <a class="card" href="https://example.com/document-5.pdf" target="_blank" rel="noopener noreferrer">
        <div class="card-header">
          <img class="card-favicon"
               src="https://icons.duckduckgo.com/ip3/example.com.ico"
               alt="" loading="lazy" width="28" height="28"
               onerror="this.style.display='none'">
          <span class="card-title">Документ 5</span>
        </div>
        <p class="card-desc">Краткая аннотация. Замените текст и адрес на актуальные.</p>
        <span class="card-link">Открыть документ</span>
      </a>

      <!-- Документ 6 -->
      <a class="card" href="https://example.com/document-6.pdf" target="_blank" rel="noopener noreferrer">
        <div class="card-header">
          <img class="card-favicon"
               src="https://icons.duckduckgo.com/ip3/example.com.ico"
               alt="" loading="lazy" width="28" height="28"
               onerror="this.style.display='none'">
          <span class="card-title">Документ 6</span>
        </div>
        <p class="card-desc">Последняя карточка в примере. Добавляйте новые блоки по аналогии.</p>
        <span class="card-link">Открыть документ</span>
      </a>
    </main>

    <footer>
      Каталог размещён на GitHub Pages · Все ссылки ведут на внешний ресурс
    </footer>
  </div>

  <script>
    // ===== Загрузка документов с сайта «Вестник Москвы» =====
    const WP_API = 'https://vestnikmoscow.mos.ru/wp-json/wp/v2/posts';
    let allDocs = {};

    async function loadDocuments() {
      try {
        const res = await fetch(`${WP_API}?per_page=100&orderby=date&order=desc&_fields=date,title,content`);
        if (!res.ok) throw new Error('Ошибка запроса к API');
        const posts = await res.json();

        const docs = [];
        posts.forEach(post => {
          const year = new Date(post.date).getFullYear();
          const title = post.title.rendered;
          const parser = new DOMParser();
          const doc = parser.parseFromString(post.content.rendered, 'text/html');
          const links = doc.querySelectorAll('a[href$=".pdf"]');
          links.forEach(link => {
            docs.push({
              year,
              name: link.textContent.trim() || title,
              url: link.href,
              date: post.date
            });
          });
        });

        docs.sort((a, b) => new Date(b.date) - new Date(a.date));

        allDocs = {};
        docs.forEach(doc => {
          if (!allDocs[doc.year]) allDocs[doc.year] = [];
          if (!allDocs[doc.year].some(d => d.url === doc.url)) {
            allDocs[doc.year].push(doc);
          }
        });

        const years = Object.keys(allDocs).sort((a, b) => b - a);
        const select = document.getElementById('year-select');
        select.innerHTML = years.map(y => `<option value="${y}">${y}</option>`).join('');

        if (years.length > 0) {
          select.value = years[0];
          renderDocuments(years[0]);
        }

        select.addEventListener('change', () => renderDocuments(select.value));

        checkNewDocument(docs);
      } catch (err) {
        console.error(err);
        document.getElementById('document-list').innerHTML =
          '<li style="color:#c62828;">Не удалось загрузить документы. Возможна ошибка CORS.</li>';
      }
    }

    function renderDocuments(year) {
      const list = document.getElementById('document-list');
      const docs = allDocs[year] || [];
      if (docs.length === 0) {
        list.innerHTML = '<li>Нет документов за этот год</li>';
        return;
      }
      list.innerHTML = docs.map(d =>
        `<li><a href="${d.url}" target="_blank" rel="noopener noreferrer">${d.name}</a></li>`
      ).join('');
    }

    function checkNewDocument(docs) {
      if (docs.length === 0) return;
      const latest = docs[0];
      const lastSeen = localStorage.getItem('lastSeenDocUrl');

      if (!lastSeen) {
        localStorage.setItem('lastSeenDocUrl', latest.url);
        return;
      }

      if (latest.url !== lastSeen) {
        const badge = document.getElementById('new-badge');
        if (badge) badge.classList.add('active');

        const notice = document.getElementById('new-doc-notice');
        notice.style.display = 'block';
        notice.innerHTML =
          `Появился новый документ: <a href="${latest.url}" target="_blank" rel="noopener noreferrer">${latest.name}</a>`;

        localStorage.setItem('lastSeenDocUrl', latest.url);
      }
    }

    // ===== Управление раскрытием виджета =====
    const widget = document.getElementById('docs-widget');
    const summary = document.getElementById('docs-summary');
    const cardVestnik = document.getElementById('card-vestnik');
    const badge = document.getElementById('new-badge');
    const hasHover = window.matchMedia('(hover: hover)').matches;

    let closeTimer;

    function setOpen(open) {
      if (open) {
        clearTimeout(closeTimer);
        widget.classList.add('open');
        cardVestnik.classList.add('is-open');
        summary.setAttribute('aria-expanded', 'true');
        if (badge) badge.classList.remove('active');
      } else {
        widget.classList.remove('open');
        cardVestnik.classList.remove('is-open');
        summary.setAttribute('aria-expanded', 'false');
      }
    }

    function delayedClose() {
      clearTimeout(closeTimer);
      closeTimer = setTimeout(() => setOpen(false), 200);
    }

    if (hasHover) {
      // Десктоп: наведение раскрывает, уход курсора — сворачивает
      widget.addEventListener('mouseenter', () => setOpen(true));
      widget.addEventListener('mouseleave', delayedClose);
    } else {
      // Тач: клик переключает
      summary.addEventListener('click', () => {
        setOpen(!widget.classList.contains('open'));
      });
    }

    // Клавиатурная доступность
    summary.addEventListener('keydown', (e) => {
      if (e.key === 'Enter' || e.key === ' ') {
        e.preventDefault();
        setOpen(!widget.classList.contains('open'));
      }
      if (e.key === 'Escape') setOpen(false);
    });

    loadDocuments();
  </script>
</body>
</html>

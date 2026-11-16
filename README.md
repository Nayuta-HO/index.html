```html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>StudyLog Pro - 受験勉強・学習記録管理</title>
  <!-- Tailwind CSS -->
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- Chart.js for Interactive Charts -->
  <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.1/dist/chart.umd.min.js"></script>
  <!-- Google Fonts: Inter & Noto Sans JP -->
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=Noto+Sans+JP:wght@400;500;600;700&display=swap" rel="stylesheet" />
  <!-- Font Awesome Icons -->
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css" />

  <script>
    tailwind.config = {
      theme: {
        extend: {
          fontFamily: {
            sans: ['"Inter"', '"Noto Sans JP"', 'sans-serif'],
            mono: ['"Inter"', 'monospace']
          }
        }
      }
    };
  </script>
</head>
<body class="bg-slate-50 text-slate-800 antialiased min-h-screen flex flex-col selection:bg-indigo-100 selection:text-indigo-800">

  <!-- カスタムトースト通知コンテナ（alert()の代替） -->
  <div id="toast-container" class="fixed top-4 right-4 z-50 flex flex-col gap-2 pointer-events-none max-w-sm w-full px-4 sm:px-0"></div>

  <!-- 上部ナビゲーションバー（白背景・クリーンデザイン） -->
  <header class="bg-white border-b border-slate-200 sticky top-0 z-30 shadow-xs">
    <div class="max-w-6xl mx-auto px-4 sm:px-6">
      <div class="flex items-center justify-between h-16">
        <!-- ブランドロゴ -->
        <div class="flex items-center gap-2.5">
          <div class="w-9 h-9 rounded-xl bg-indigo-600 text-white flex items-center justify-center shadow-sm">
            <i class="fa-solid fa-graduation-cap text-base"></i>
          </div>
          <div>
            <h1 class="font-bold text-slate-900 text-base sm:text-lg leading-tight tracking-tight">StudyLog Pro</h1>
            <p class="text-[11px] text-slate-400 hidden sm:block">自動保存対応・受験学習マネジメント</p>
          </div>
        </div>

        <!-- デスクトップ用 3ページ切替タブ -->
        <nav class="hidden md:flex items-center bg-slate-100 p-1 rounded-xl border border-slate-200/70" aria-label="メインナビゲーション">
          <button type="button" data-tab="home" class="nav-tab-btn px-4 py-2 rounded-lg text-sm font-semibold transition-all flex items-center gap-2 bg-white text-indigo-600 shadow-xs">
            <i class="fa-solid fa-stopwatch"></i>
            <span>ホーム（タイマー・目標・カレンダー）</span>
          </button>
          <button type="button" data-tab="analytics" class="nav-tab-btn px-4 py-2 rounded-lg text-sm font-semibold transition-all flex items-center gap-2 text-slate-600 hover:text-slate-900">
            <i class="fa-solid fa-chart-pie"></i>
            <span>分析・過去問</span>
          </button>
          <button type="button" data-tab="materials" class="nav-tab-btn px-4 py-2 rounded-lg text-sm font-semibold transition-all flex items-center gap-2 text-slate-600 hover:text-slate-900">
            <i class="fa-solid fa-book-bookmark"></i>
            <span>教材・学習ログ</span>
          </button>
        </nav>

        <!-- データ管理 / サンプル操作メニュー -->
        <div class="flex items-center gap-2">
          <span id="autosave-badge" class="inline-flex items-center gap-1.5 text-xs font-medium text-emerald-700 bg-emerald-50 border border-emerald-200 px-2.5 py-1 rounded-full">
            <span class="w-1.5 h-1.5 rounded-full bg-emerald-500"></span>
            自動保存済
          </span>
          <button type="button" id="open-data-modal-btn" class="p-2.5 text-slate-500 hover:text-slate-800 hover:bg-slate-100 rounded-xl transition-colors" title="データ管理・サンプル読込">
            <i class="fa-solid fa-gear"></i>
          </button>
        </div>
      </div>

      <!-- モバイル用 3ページ切替タブ -->
      <nav class="flex md:hidden items-center justify-between gap-1 pb-2.5 pt-1 overflow-x-auto" aria-label="モバイルナビゲーション">
        <button type="button" data-tab="home" class="nav-tab-btn flex-1 py-2 px-2 rounded-xl text-xs font-semibold transition-all flex flex-col items-center gap-1 bg-indigo-50 text-indigo-600 border border-indigo-200">
          <i class="fa-solid fa-stopwatch text-sm"></i>
          <span>ホーム</span>
        </button>
        <button type="button" data-tab="analytics" class="nav-tab-btn flex-1 py-2 px-2 rounded-xl text-xs font-semibold transition-all flex flex-col items-center gap-1 text-slate-600 bg-slate-50 border border-transparent">
          <i class="fa-solid fa-chart-pie text-sm"></i>
          <span>分析・過去問</span>
        </button>
        <button type="button" data-tab="materials" class="nav-tab-btn flex-1 py-2 px-2 rounded-xl text-xs font-semibold transition-all flex flex-col items-center gap-1 text-slate-600 bg-slate-50 border border-transparent">
          <i class="fa-solid fa-book-bookmark text-sm"></i>
          <span>教材・ログ</span>
        </button>
      </nav>
    </div>
  </header>

  <!-- メインコンテンツ領域 -->
  <main class="flex-1 max-w-6xl w-full mx-auto px-4 sm:px-6 py-6 space-y-6">

    <!-- ===================================================================== -->
    <!-- ページ 1：ホーム（タイマー・目標・カレンダー）                        -->
    <!-- ===================================================================== -->
    <section id="page-home" class="page-section space-y-6">

      <!-- 上段：目標設定（デイリー・マイルストーン・ゴール） -->
      <div class="grid grid-cols-1 lg:grid-cols-3 gap-4">
        <!-- 1. デイリー目標カード -->
        <div class="bg-white rounded-2xl border border-slate-200 p-5 shadow-xs flex flex-col justify-between">
          <div>
            <div class="flex items-center justify-between mb-3">
              <div class="flex items-center gap-2">
                <span class="w-8 h-8 rounded-lg bg-emerald-50 text-emerald-600 flex items-center justify-center">
                  <i class="fa-solid fa-fire"></i>
                </span>
                <div>
                  <h2 class="text-sm font-bold text-slate-900">デイリー目標</h2>
                  <p class="text-xs text-slate-400">今日の学習達成状況</p>
                </div>
              </div>
              <button type="button" onclick="openGoalModal('daily')" class="text-xs font-semibold text-indigo-600 hover:text-indigo-700 bg-indigo-50 hover:bg-indigo-100 px-3 py-1.5 rounded-lg transition-colors">
                設定変更
              </button>
            </div>
            <div class="flex items-baseline justify-between mt-2">
              <div>
                <span id="daily-current-min" class="text-2xl sm:text-3xl font-bold text-slate-900">0</span>
                <span class="text-xs text-slate-500 ml-1">分 / 目標 <strong id="daily-target-min" class="text-slate-700">180</strong>分</span>
              </div>
              <span id="daily-percent-badge" class="text-xs font-bold px-2.5 py-1 rounded-full bg-slate-100 text-slate-600">0%</span>
            </div>
          </div>
          <div class="mt-4">
            <div class="w-full h-2.5 bg-slate-100 rounded-full overflow-hidden">
              <div id="daily-progress-bar" class="h-full bg-emerald-500 rounded-full transition-all duration-500" style="width: 0%"></div>
            </div>
            <p id="daily-remaining-text" class="text-xs text-slate-500 mt-2">目標達成まであと 180 分</p>
          </div>
        </div>

        <!-- 2. マイルストーン目標カード -->
        <div class="bg-white rounded-2xl border border-slate-200 p-5 shadow-xs flex flex-col justify-between">
          <div>
            <div class="flex items-center justify-between mb-3">
              <div class="flex items-center gap-2">
                <span class="w-8 h-8 rounded-lg bg-amber-50 text-amber-600 flex items-center justify-center">
                  <i class="fa-solid fa-flag"></i>
                </span>
                <div>
                  <h2 class="text-sm font-bold text-slate-900">マイルストーン目標</h2>
                  <p class="text-xs text-slate-400">直近の模試・定期試験など</p>
                </div>
              </div>
              <button type="button" onclick="openGoalModal('milestone')" class="text-xs font-semibold text-indigo-600 hover:text-indigo-700 bg-indigo-50 hover:bg-indigo-100 px-3 py-1.5 rounded-lg transition-colors">
                編集
              </button>
            </div>
            <p id="milestone-title-display" class="text-sm font-semibold text-slate-800 truncate">第2回 全統記述模試</p>
            <p id="milestone-date-display" class="text-xs text-slate-400 mt-0.5">目標日: 未設定</p>
          </div>
          <div class="mt-3 pt-3 border-t border-slate-100 flex items-baseline justify-between">
            <span class="text-xs font-medium text-slate-500">目標達成の目安まで</span>
            <div id="milestone-countdown-box" class="text-right">
              <span class="text-xs text-slate-500">あと</span>
              <span id="milestone-days-left" class="text-2xl sm:text-3xl font-bold text-amber-600 mx-1">--</span>
              <span class="text-xs font-bold text-slate-700">日</span>
            </div>
          </div>
        </div>

        <!-- 3. ゴール目標（最終目標）カード -->
        <div class="bg-white rounded-2xl border border-slate-200 p-5 shadow-xs flex flex-col justify-between">
          <div>
            <div class="flex items-center justify-between mb-3">
              <div class="flex items-center gap-2">
                <span class="w-8 h-8 rounded-lg bg-indigo-50 text-indigo-600 flex items-center justify-center">
                  <i class="fa-solid fa-trophy"></i>
                </span>
                <div>
                  <h2 class="text-sm font-bold text-slate-900">ゴール目標（最終目標）</h2>
                  <p class="text-xs text-slate-400">志望校合格・本番試験日</p>
                </div>
              </div>
              <button type="button" onclick="openGoalModal('goal')" class="text-xs font-semibold text-indigo-600 hover:text-indigo-700 bg-indigo-50 hover:bg-indigo-100 px-3 py-1.5 rounded-lg transition-colors">
                編集
              </button>
            </div>
            <p id="goal-title-display" class="text-sm font-semibold text-slate-800 truncate">第一志望大学 合格</p>
            <p id="goal-date-display" class="text-xs text-slate-400 mt-0.5">目標日: 未設定</p>
          </div>
          <div class="mt-3 pt-3 border-t border-slate-100 flex items-baseline justify-between">
            <span class="text-xs font-medium text-slate-500">最終ゴールまで</span>
            <div id="goal-countdown-box" class="text-right">
              <span class="text-xs text-slate-500">あと</span>
              <span id="goal-days-left" class="text-2xl sm:text-3xl font-bold text-indigo-600 mx-1">--</span>
              <span class="text-xs font-bold text-slate-700">日</span>
            </div>
          </div>
        </div>
      </div>

      <!-- 下段：学習タイマー（左） & 達成確認カレンダー（右） -->
      <div class="grid grid-cols-1 lg:grid-cols-12 gap-6">

        <!-- 学習タイマーカード（5カラム） -->
        <div class="lg:col-span-5 bg-white rounded-2xl border border-slate-200 p-5 sm:p-6 shadow-xs flex flex-col justify-between space-y-5">
          <div>
            <div class="flex items-center justify-between gap-2 mb-4">
              <h2 class="text-base font-bold text-slate-900 flex items-center gap-2">
                <i class="fa-regular fa-clock text-indigo-600"></i>
                <span>学習タイマー</span>
              </h2>
              <!-- アラーム設定ボタン -->
              <button type="button" id="open-alarm-modal-btn" class="inline-flex items-center gap-1.5 text-xs font-semibold text-slate-600 hover:text-indigo-600 bg-slate-100 hover:bg-indigo-50 px-3 py-1.5 rounded-xl transition-colors">
                <i class="fa-solid fa-bell"></i>
                <span>アラーム設定</span>
              </button>
            </div>

            <!-- モード切替タブ（カウントアップ / カウントダウン） -->
            <div class="grid grid-cols-2 gap-1.5 bg-slate-100 p-1 rounded-xl mb-5">
              <button type="button" id="mode-countdown-btn" class="py-2 px-3 rounded-lg text-xs sm:text-sm font-semibold bg-white text-indigo-600 shadow-xs transition-all">
                カウントダウン
              </button>
              <button type="button" id="mode-countup-btn" class="py-2 px-3 rounded-lg text-xs sm:text-sm font-semibold text-slate-600 hover:text-slate-900 transition-all">
                カウントアップ
              </button>
            </div>

            <!-- カウントダウン1分単位調整コントロール -->
            <div id="countdown-setup-panel" class="mb-4 bg-slate-50 border border-slate-200/80 rounded-xl p-3">
              <div class="flex items-center justify-between gap-2">
                <span class="text-xs font-semibold text-slate-600">セット時間（1分単位）</span>
                <div class="flex items-center gap-1.5">
                  <button type="button" id="timer-minus-5" class="px-2 py-1 text-xs font-semibold bg-white border border-slate-200 rounded-lg hover:bg-slate-100">-5分</button>
                  <button type="button" id="timer-minus-1" class="px-2.5 py-1 text-xs font-bold bg-white border border-slate-200 rounded-lg hover:bg-slate-100">-1</button>
                  <div class="flex items-center bg-white border border-slate-300 rounded-lg px-2 py-1">
                    <input type="number" id="timer-minutes-input" min="1" max="300" value="25" class="w-12 text-center text-sm font-bold text-slate-800 focus:outline-none" />
                    <span class="text-xs text-slate-500">分</span>
                  </div>
                  <button type="button" id="timer-plus-1" class="px-2.5 py-1 text-xs font-bold bg-white border border-slate-200 rounded-lg hover:bg-slate-100">+1</button>
                  <button type="button" id="timer-plus-5" class="px-2 py-1 text-xs font-semibold bg-white border border-slate-200 rounded-lg hover:bg-slate-100">+5分</button>
                </div>
              </div>
            </div>

            <!-- デジタルタイマー表示盤 -->
            <div class="relative bg-slate-900 text-white rounded-2xl py-7 px-4 text-center shadow-inner overflow-hidden">
              <div id="timer-subject-indicator" class="inline-flex items-center gap-1.5 text-xs font-semibold px-3 py-1 rounded-full bg-white/10 text-slate-200 mb-2">
                <span id="timer-subject-dot" class="w-2 h-2 rounded-full bg-indigo-400"></span>
                <span id="timer-subject-label">英語リーディング</span>
              </div>
              <div id="timer-display" class="text-5xl sm:text-6xl font-bold tracking-wider font-mono tabular-nums my-1">
                25:00
              </div>
              <p id="timer-status-text" class="text-xs text-slate-400 mt-2">準備完了・スタートを押してください</p>
            </div>

            <!-- タイマー操作ボタン -->
            <div class="grid grid-cols-3 gap-2.5 mt-4">
              <button type="button" id="timer-start-pause-btn" class="col-span-2 py-3 px-4 rounded-xl bg-indigo-600 hover:bg-indigo-700 text-white font-bold text-sm shadow-xs transition-all flex items-center justify-center gap-2">
                <i class="fa-solid fa-play"></i>
                <span>スタート</span>
              </button>
              <button type="button" id="timer-reset-btn" class="py-3 px-3 rounded-xl bg-slate-100 hover:bg-slate-200 text-slate-700 font-semibold text-sm transition-all flex items-center justify-center gap-1.5">
                <i class="fa-solid fa-rotate-left"></i>
                <span>リセット</span>
              </button>
            </div>
          </div>

          <!-- タイマー下の「科目」と「やる教材」設定（この科目のこの教材を何分勉強したか記録） -->
          <div class="pt-4 border-t border-slate-200 space-y-3">
            <div>
              <label for="timer-subject-select" class="block text-xs font-bold text-slate-700 mb-1">
                勉強する科目 <span class="text-indigo-600">*</span>
              </label>
              <select id="timer-subject-select" class="subject-select-target w-full rounded-xl border border-slate-300 bg-white px-3 py-2.5 text-sm font-medium text-slate-800 focus:border-indigo-500 focus:ring-2 focus:ring-indigo-200 focus:outline-none">
                <!-- JSで3大枠optgroup付きで動的生成 -->
              </select>
            </div>

            <div>
              <div class="flex items-center justify-between mb-1">
                <label for="timer-material-select" class="block text-xs font-bold text-slate-700">
                  使用する教材（学習ログに記録）
                </label>
                <button type="button" onclick="switchTab('materials')" class="text-[11px] font-semibold text-indigo-600 hover:underline">
                  + 新しい教材を登録
                </button>
              </div>
              <select id="timer-material-select" class="w-full rounded-xl border border-slate-300 bg-white px-3 py-2.5 text-sm text-slate-800 focus:border-indigo-500 focus:ring-2 focus:ring-indigo-200 focus:outline-none">
                <option value="">（教材指定なし / 自由演習）</option>
              </select>
            </div>

            <!-- 経過時間を学習ログに保存するボタン -->
            <button type="button" id="timer-save-log-btn" class="w-full py-3 px-4 rounded-xl bg-emerald-600 hover:bg-emerald-700 text-white font-bold text-sm shadow-xs transition-all flex items-center justify-center gap-2">
              <i class="fa-solid fa-check-circle"></i>
              <span id="timer-save-btn-label">学習時間を記録する（0分）</span>
            </button>
          </div>
        </div>

        <!-- カレンダーカード（7カラム） -->
        <div class="lg:col-span-7 bg-white rounded-2xl border border-slate-200 p-5 sm:p-6 shadow-xs flex flex-col justify-between">
          <div>
            <!-- カレンダーヘッダー -->
            <div class="flex flex-wrap items-center justify-between gap-3 mb-4">
              <div>
                <h2 class="text-base font-bold text-slate-900 flex items-center gap-2">
                  <i class="fa-regular fa-calendar-check text-indigo-600"></i>
                  <span>目標達成カレンダー</span>
                </h2>
                <p class="text-xs text-slate-400 mt-0.5">日付をタップするとその日の学習内訳を確認できます</p>
              </div>

              <div class="flex items-center gap-2">
                <button type="button" id="cal-prev-month" class="p-2 rounded-xl border border-slate-200 hover:bg-slate-50 text-slate-600" aria-label="前月">
                  <i class="fa-solid fa-chevron-left text-xs"></i>
                </button>
                <span id="cal-month-title" class="text-sm font-bold text-slate-800 min-w-[110px] text-center">2026年 11月</span>
                <button type="button" id="cal-next-month" class="p-2 rounded-xl border border-slate-200 hover:bg-slate-50 text-slate-600" aria-label="翌月">
                  <i class="fa-solid fa-chevron-right text-xs"></i>
                </button>
                <button type="button" id="cal-today-btn" class="px-2.5 py-1.5 text-xs font-semibold bg-slate-100 hover:bg-slate-200 text-slate-700 rounded-lg">
                  今日
                </button>
              </div>
            </div>

            <!-- 凡例 -->
            <div class="flex flex-wrap items-center gap-4 text-xs text-slate-500 mb-3 bg-slate-50 px-3 py-2 rounded-xl">
              <span class="flex items-center gap-1.5">
                <span class="w-3 h-3 rounded-sm bg-emerald-500 inline-block"></span>
                目標達成（100%以上）
              </span>
              <span class="flex items-center gap-1.5">
                <span class="w-3 h-3 rounded-sm bg-indigo-200 border border-indigo-400 inline-block"></span>
                学習あり（目標未満）
              </span>
              <span class="flex items-center gap-1.5">
                <span class="w-3 h-3 rounded-sm bg-white border border-slate-200 inline-block"></span>
                記録なし
              </span>
            </div>

            <!-- 曜日ヘッダー -->
            <div class="grid grid-cols-7 gap-1.5 text-center text-xs font-bold text-slate-500 mb-1.5">
              <div class="text-rose-500">日</div>
              <div>月</div>
              <div>火</div>
              <div>水</div>
              <div>木</div>
              <div>金</div>
              <div class="text-blue-500">土</div>
            </div>

            <!-- カレンダーグリッド -->
            <div id="calendar-grid" class="grid grid-cols-7 gap-1.5">
              <!-- JSで生成 -->
            </div>
          </div>

          <!-- 選択日の学習詳細パネル -->
          <div class="mt-5 pt-4 border-t border-slate-200">
            <div class="flex items-center justify-between mb-2">
              <h3 id="selected-date-heading" class="text-xs font-bold text-slate-700">選択日の記録</h3>
              <span id="selected-date-total" class="text-xs font-bold text-indigo-600">合計 0分</span>
            </div>
            <div id="selected-date-details" class="space-y-1.5 max-h-36 overflow-y-auto pr-1">
              <p class="text-xs text-slate-400 py-2">この日の学習記録はありません。</p>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- ===================================================================== -->
    <!-- ページ 2：分析・過去問（グラフ・過去問記録）                          -->
    <!-- ===================================================================== -->
    <section id="page-analytics" class="page-section hidden space-y-6">

      <!-- ヘッダー＆手入力記録追加ボタン -->
      <div class="bg-white rounded-2xl border border-slate-200 p-5 shadow-xs flex flex-col sm:flex-row sm:items-center justify-between gap-4">
        <div>
          <h2 class="text-lg font-bold text-slate-900">学習データの可視化・分析</h2>
          <p class="text-xs text-slate-500 mt-0.5">科目ごとの勉強比率と直近1週間の日別学習量を確認できます（※グラフは科目単位で集計されます）</p>
        </div>
        <div class="flex flex-wrap items-center gap-2">
          <select id="chart-period-select" class="rounded-xl border border-slate-300 bg-white px-3 py-2 text-xs font-semibold text-slate-700 focus:outline-none focus:border-indigo-500">
            <option value="all">比率集計：全期間</option>
            <option value="month">比率集計：今月</option>
            <option value="week">比率集計：直近7日間</option>
          </select>
          <button type="button" id="open-manual-log-modal-btn" class="inline-flex items-center gap-2 px-4 py-2 rounded-xl bg-indigo-600 hover:bg-indigo-700 text-white text-xs sm:text-sm font-bold shadow-xs transition-all">
            <i class="fa-solid fa-plus"></i>
            <span>勉強時間を手入力追加</span>
          </button>
        </div>
      </div>

      <!-- グラフエリア（2カラム） -->
      <div class="grid grid-cols-1 lg:grid-cols-12 gap-6">
        <!-- 科目比率ドーナツグラフ（5カラム） -->
        <div class="lg:col-span-5 bg-white rounded-2xl border border-slate-200 p-5 sm:p-6 shadow-xs flex flex-col justify-between">
          <div>
            <h3 class="text-base font-bold text-slate-900 mb-1">科目別 勉強時間比率</h3>
            <p class="text-xs text-slate-400 mb-4">各科目の固有カラーで学習バランスを表示</p>
            <div class="relative h-64 w-full flex items-center justify-center">
              <canvas id="subject-ratio-chart"></canvas>
              <div id="ratio-chart-empty" class="hidden absolute inset-0 flex flex-col items-center justify-center text-slate-400 text-xs">
                <i class="fa-solid fa-chart-pie text-2xl mb-2 text-slate-300"></i>
                <span>データがありません</span>
              </div>
            </div>
          </div>
          <!-- 科目別内訳リスト -->
          <div id="subject-ratio-legend" class="mt-4 pt-4 border-t border-slate-100 max-h-44 overflow-y-auto space-y-1.5 text-xs">
            <!-- JSで生成 -->
          </div>
        </div>

        <!-- 1週間の勉強量 積み上げ棒グラフ（7カラム） -->
        <div class="lg:col-span-7 bg-white rounded-2xl border border-slate-200 p-5 sm:p-6 shadow-xs flex flex-col justify-between">
          <div>
            <div class="flex items-center justify-between mb-1">
              <h3 class="text-base font-bold text-slate-900">1週間の勉強量（科目別積み上げ棒グラフ）</h3>
              <span id="weekly-total-badge" class="text-xs font-bold bg-indigo-50 text-indigo-700 px-3 py-1 rounded-full">週間合計: 0分</span>
            </div>
            <p class="text-xs text-slate-400 mb-4">直近7日間の日別勉強時間を科目ごとに積み上げて表示</p>
            <div class="relative h-80 w-full">
              <canvas id="weekly-stacked-chart"></canvas>
            </div>
          </div>
          <div class="mt-3 text-[11px] text-slate-400 flex items-center gap-1.5">
            <i class="fa-solid fa-circle-info"></i>
            <span>棒グラフには科目ごとの学習時間のみが反映されます。使用教材の詳細は「教材・学習ログ」ページで確認できます。</span>
          </div>
        </div>
      </div>

      <!-- 過去問演習記録セクション -->
      <div class="bg-white rounded-2xl border border-slate-200 p-5 sm:p-6 shadow-xs space-y-5">
        <div class="flex flex-col sm:flex-row sm:items-center justify-between gap-2 border-b border-slate-100 pb-4">
          <div>
            <h3 class="text-base sm:text-lg font-bold text-slate-900 flex items-center gap-2">
              <i class="fa-solid fa-file-signature text-indigo-600"></i>
              <span>過去問演習記録</span>
            </h3>
            <p class="text-xs text-slate-500 mt-0.5">年度・大学名・かかった時間（分）・正答率（％）を登録し、横並びで一覧比較できます</p>
          </div>
        </div>

        <!-- 過去問 手入力フォーム -->
        <form id="past-exam-form" class="bg-slate-50 border border-slate-200/80 rounded-2xl p-4 grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-12 gap-3 items-end">
          <div class="lg:col-span-2">
            <label for="exam-year" class="block text-xs font-bold text-slate-700 mb-1">年度 <span class="text-rose-500">*</span></label>
            <input type="text" id="exam-year" placeholder="例: 2025年度" required class="w-full rounded-xl border border-slate-300 bg-white px-3 py-2 text-sm focus:border-indigo-500 focus:outline-none" />
          </div>
          <div class="lg:col-span-3">
            <label for="exam-univ" class="block text-xs font-bold text-slate-700 mb-1">大学名・学部 <span class="text-rose-500">*</span></label>
            <input type="text" id="exam-univ" placeholder="例: 東京大学 理科一類" required class="w-full rounded-xl border border-slate-300 bg-white px-3 py-2 text-sm focus:border-indigo-500 focus:outline-none" />
          </div>
          <div class="lg:col-span-2">
            <label for="exam-subject" class="block text-xs font-bold text-slate-700 mb-1">科目</label>
            <select id="exam-subject" class="subject-select-target w-full rounded-xl border border-slate-300 bg-white px-2.5 py-2 text-sm focus:border-indigo-500 focus:outline-none">
              <!-- JSで3大枠optgroup付きで生成 -->
            </select>
          </div>
          <div class="lg:col-span-2">
            <label for="exam-minutes" class="block text-xs font-bold text-slate-700 mb-1">かかった時間(分) <span class="text-rose-500">*</span></label>
            <input type="number" id="exam-minutes" min="1" max="600" placeholder="例: 120" required class="w-full rounded-xl border border-slate-300 bg-white px-3 py-2 text-sm focus:border-indigo-500 focus:outline-none" />
          </div>
          <div class="lg:col-span-2">
            <label for="exam-accuracy" class="block text-xs font-bold text-slate-700 mb-1">正答率(%) <span class="text-rose-500">*</span></label>
            <input type="number" id="exam-accuracy" min="0" max="100" step="0.1" placeholder="例: 78.5" required class="w-full rounded-xl border border-slate-300 bg-white px-3 py-2 text-sm focus:border-indigo-500 focus:outline-none" />
          </div>
          <div class="lg:col-span-1">
            <button type="submit" class="w-full py-2 px-3 rounded-xl bg-indigo-600 hover:bg-indigo-700 text-white font-bold text-sm transition-colors shadow-xs">
              追加
            </button>
          </div>
        </form>

        <!-- 過去問一覧テーブル（年度・大学名・かかった時間・正答率を横で同時に確認） -->
        <div class="overflow-x-auto rounded-xl border border-slate-200">
          <table class="w-full text-left border-collapse min-w-[640px]">
            <thead>
              <tr class="bg-slate-50 border-b border-slate-200 text-xs font-bold text-slate-600">
                <th class="py-3 px-4">年度</th>
                <th class="py-3 px-4">大学名・学部</th>
                <th class="py-3 px-4">科目</th>
                <th class="py-3 px-4">かかった時間（分）</th>
                <th class="py-3 px-4">正答率（％）</th>
                <th class="py-3 px-4 text-right">操作</th>
              </tr>
            </thead>
            <tbody id="past-exam-tbody" class="divide-y divide-slate-100 text-sm">
              <!-- JSで生成 -->
            </tbody>
          </table>
        </div>
      </div>
    </section>

    <!-- ===================================================================== -->
    <!-- ページ 3：教材・学習ログ（教材登録・本棚・学習ログ表）                -->
    <!-- ===================================================================== -->
    <section id="page-materials" class="page-section hidden space-y-6">

      <!-- 上段：教材登録フォーム（カメラ撮影読み取り & 手入力） + マイ本棚 -->
      <div class="grid grid-cols-1 lg:grid-cols-12 gap-6">

        <!-- 教材登録カード（5カラム） -->
        <div class="lg:col-span-5 bg-white rounded-2xl border border-slate-200 p-5 sm:p-6 shadow-xs space-y-4">
          <div>
            <h2 class="text-base sm:text-lg font-bold text-slate-900 flex items-center gap-2">
              <i class="fa-solid fa-book-medical text-indigo-600"></i>
              <span>新しい教材を登録</span>
            </h2>
            <p class="text-xs text-slate-500 mt-0.5">カメラで表紙を撮影して読み取るか、手入力で本棚に追加できます</p>
          </div>

          <!-- カメラ起動 / 画像選択エリア -->
          <div class="bg-slate-50 border border-dashed border-slate-300 rounded-2xl p-4 text-center space-y-3">
            <!-- カメラ映像プレビュー（起動時のみ表示） -->
            <div id="camera-stream-wrapper" class="hidden relative rounded-xl overflow-hidden bg-black">
              <video id="camera-video" autoplay playsinline class="w-full h-48 object-cover"></video>
              <div class="absolute bottom-2 inset-x-0 flex justify-center gap-2">
                <button type="button" id="capture-photo-btn" class="px-4 py-1.5 rounded-full bg-emerald-600 text-white text-xs font-bold shadow">
                  <i class="fa-solid fa-camera mr-1"></i>撮影する
                </button>
                <button type="button" id="close-camera-btn" class="px-3 py-1.5 rounded-full bg-slate-800/80 text-white text-xs font-semibold">
                  閉じる
                </button>
              </div>
            </div>

            <!-- 撮影した表紙画像のプレビュー -->
            <div id="material-cover-preview-box" class="hidden relative mx-auto w-28 h-36 rounded-xl overflow-hidden shadow-md border border-slate-200 bg-white">
              <img id="material-cover-preview-img" src="" alt="教材表紙プレビュー" class="w-full h-full object-cover" />
              <button type="button" id="clear-cover-img-btn" class="absolute top-1 right-1 w-6 h-6 rounded-full bg-slate-900/70 text-white text-xs flex items-center justify-center" title="画像を削除">
                <i class="fa-solid fa-xmark"></i>
              </button>
            </div>

            <div class="flex flex-wrap items-center justify-center gap-2">
              <button type="button" id="start-camera-btn" class="inline-flex items-center gap-1.5 px-3.5 py-2 rounded-xl bg-indigo-600 hover:bg-indigo-700 text-white text-xs font-bold transition-colors shadow-xs">
                <i class="fa-solid fa-camera"></i>
                <span>カメラで読み取り</span>
              </button>
              <label class="inline-flex items-center gap-1.5 px-3.5 py-2 rounded-xl bg-white hover:bg-slate-100 text-slate-700 border border-slate-300 text-xs font-semibold cursor-pointer transition-colors">
                <i class="fa-regular fa-image"></i>
                <span>画像を選択</span>
                <input type="file" id="material-image-file" accept="image/*" capture="environment" class="hidden" />
              </label>
            </div>
            <p class="text-[11px] text-slate-400">表紙の撮影または画像選択で表紙カバー付き教材として登録されます</p>
            <canvas id="camera-capture-canvas" class="hidden"></canvas>
          </div>

          <!-- 手入力フォーム -->
          <form id="material-register-form" class="space-y-3">
            <div>
              <label for="material-name-input" class="block text-xs font-bold text-slate-700 mb-1">
                教材名（手入力） <span class="text-rose-500">*</span>
              </label>
              <input type="text" id="material-name-input" placeholder="例: システム英単語 / 青チャート数学I+A" required class="w-full rounded-xl border border-slate-300 bg-white px-3 py-2.5 text-sm focus:border-indigo-500 focus:outline-none" />
            </div>

            <div>
              <label for="material-subject-select" class="block text-xs font-bold text-slate-700 mb-1">
                対象科目 <span class="text-rose-500">*</span>
              </label>
              <select id="material-subject-select" class="subject-select-target w-full rounded-xl border border-slate-300 bg-white px-3 py-2.5 text-sm font-medium focus:border-indigo-500 focus:outline-none">
                <!-- JSで3大枠optgroup付きで生成 -->
              </select>
            </div>

            <div>
              <label for="material-memo-input" class="block text-xs font-bold text-slate-700 mb-1">
                メモ・目標周回数（任意）
              </label>
              <input type="text" id="material-memo-input" placeholder="例: 毎日100語ずつ復習・3周目標" class="w-full rounded-xl border border-slate-300 bg-white px-3 py-2 text-sm focus:border-indigo-500 focus:outline-none" />
            </div>

            <button type="submit" class="w-full py-3 px-4 rounded-xl bg-indigo-600 hover:bg-indigo-700 text-white font-bold text-sm shadow-xs transition-all flex items-center justify-center gap-2">
              <i class="fa-solid fa-plus"></i>
              <span>本棚に登録する</span>
            </button>
          </form>
        </div>

        <!-- マイ本棚（登録教材ライブラリ）（7カラム） -->
        <div class="lg:col-span-7 bg-white rounded-2xl border border-slate-200 p-5 sm:p-6 shadow-xs flex flex-col justify-between">
          <div>
            <div class="flex flex-wrap items-center justify-between gap-2 mb-4">
              <div>
                <h2 class="text-base sm:text-lg font-bold text-slate-900 flex items-center gap-2">
                  <i class="fa-solid fa-swatchbook text-indigo-600"></i>
                  <span>マイ本棚（登録教材一覧）</span>
                </h2>
                <p class="text-xs text-slate-500 mt-0.5">タイマー画面の教材リストと自動連携します</p>
              </div>
              <select id="bookshelf-filter-subject" class="rounded-xl border border-slate-300 bg-white px-3 py-1.5 text-xs font-semibold text-slate-700 focus:outline-none">
                <option value="ALL">すべての科目を表示</option>
              </select>
            </div>

            <!-- 本棚グリッド -->
            <div id="bookshelf-grid" class="grid grid-cols-1 sm:grid-cols-2 gap-3 max-h-[460px] overflow-y-auto pr-1">
              <!-- JSでカード生成 -->
            </div>
          </div>
        </div>
      </div>

      <!-- 下段：学習ログ一覧（記録グラフとは別の独立した表形式） -->
      <div class="bg-white rounded-2xl border border-slate-200 p-5 sm:p-6 shadow-xs space-y-4">
        <div class="flex flex-col sm:flex-row sm:items-center justify-between gap-3 border-b border-slate-100 pb-4">
          <div>
            <h2 class="text-base sm:text-lg font-bold text-slate-900 flex items-center gap-2">
              <i class="fa-solid fa-table-list text-indigo-600"></i>
              <span>学習ログ一覧（科目・教材・時間）</span>
            </h2>
            <p class="text-xs text-slate-500 mt-0.5">どの科目のどの教材を何分勉強したかを時系列の表形式で管理します</p>
          </div>
          <div class="flex flex-wrap items-center gap-2">
            <select id="log-filter-subject" class="rounded-xl border border-slate-300 bg-white px-3 py-2 text-xs font-semibold text-slate-700 focus:outline-none">
              <option value="ALL">全科目で絞り込み</option>
            </select>
            <button type="button" onclick="openManualLogModal()" class="inline-flex items-center gap-1.5 px-3.5 py-2 rounded-xl bg-indigo-50 hover:bg-indigo-100 text-indigo-700 text-xs font-bold transition-colors">
              <i class="fa-solid fa-plus"></i>
              <span>ログを手入力追加</span>
            </button>
          </div>
        </div>

        <!-- 学習ログ テーブル -->
        <div class="overflow-x-auto rounded-xl border border-slate-200">
          <table class="w-full text-left border-collapse min-w-[600px]">
            <thead>
              <tr class="bg-slate-50 border-b border-slate-200 text-xs font-bold text-slate-600">
                <th class="py-3 px-4">学習日</th>
                <th class="py-3 px-4">科目</th>
                <th class="py-3 px-4">使用した教材</th>
                <th class="py-3 px-4">勉強時間</th>
                <th class="py-3 px-4 text-right">削除</th>
              </tr>
            </thead>
            <tbody id="study-log-tbody" class="divide-y divide-slate-100 text-sm">
              <!-- JSで生成 -->
            </tbody>
          </table>
        </div>
      </div>

    </section>

  </main>

  <!-- ===================================================================== -->
  <!-- モーダル群（目標設定 / アラーム設定 / 手入力ログ追加 / データ管理）  -->
  <!-- ===================================================================== -->

  <!-- 1. 目標設定モーダル -->
  <div id="goal-modal" class="hidden fixed inset-0 z-50 bg-slate-900/50 backdrop-blur-xs flex items-center justify-center p-4">
    <div class="bg-white rounded-2xl max-w-md w-full p-6 shadow-xl border border-slate-200 space-y-4">
      <div class="flex items-center justify-between">
        <h3 id="goal-modal-title" class="text-base font-bold text-slate-900">目標設定</h3>
        <button type="button" onclick="closeModal('goal-modal')" class="text-slate-400 hover:text-slate-700 p-1">
          <i class="fa-solid fa-xmark text-lg"></i>
        </button>
      </div>
      <form id="goal-modal-form" class="space-y-4">
        <input type="hidden" id="goal-modal-type" />
        <div id="goal-daily-fields" class="hidden space-y-3">
          <div>
            <label for="input-daily-minutes" class="block text-xs font-bold text-slate-700 mb-1">1日の目標勉強時間（分）</label>
            <input type="number" id="input-daily-minutes" min="10" max="1440" step="10" class="w-full rounded-xl border border-slate-300 px-3 py-2.5 text-sm font-bold" />
            <p class="text-[11px] text-slate-400 mt-1">例: 180分（3時間）、300分（5時間）、480分（8時間）</p>
          </div>
        </div>
        <div id="goal-event-fields" class="hidden space-y-3">
          <div>
            <label for="input-event-title" class="block text-xs font-bold text-slate-700 mb-1">目標タイトル</label>
            <input type="text" id="input-event-title" placeholder="例: 共通テスト本番 / 〇〇大学 2次試験" class="w-full rounded-xl border border-slate-300 px-3 py-2.5 text-sm" />
          </div>
          <div>
            <label for="input-event-date" class="block text-xs font-bold text-slate-700 mb-1">目標日（試験日）</label>
            <input type="date" id="input-event-date" class="w-full rounded-xl border border-slate-300 px-3 py-2.5 text-sm" />
          </div>
        </div>
        <div class="flex justify-end gap-2 pt-2">
          <button type="button" onclick="closeModal('goal-modal')" class="px-4 py-2 rounded-xl text-xs font-semibold text-slate-600 hover:bg-slate-100">キャンセル</button>
          <button type="submit" class="px-5 py-2 rounded-xl text-xs font-bold bg-indigo-600 hover:bg-indigo-700 text-white">保存する</button>
        </div>
      </form>
    </div>
  </div>

  <!-- 2. アラーム設定モーダル -->
  <div id="alarm-modal" class="hidden fixed inset-0 z-50 bg-slate-900/50 backdrop-blur-xs flex items-center justify-center p-4">
    <div class="bg-white rounded-2xl max-w-md w-full p-6 shadow-xl border border-slate-200 space-y-4">
      <div class="flex items-center justify-between">
        <h3 class="text-base font-bold text-slate-900 flex items-center gap-2">
          <i class="fa-solid fa-bell text-indigo-600"></i>
          <span>タイマーアラーム設定</span>
        </h3>
        <button type="button" onclick="closeModal('alarm-modal')" class="text-slate-400 hover:text-slate-700 p-1">
          <i class="fa-solid fa-xmark text-lg"></i>
        </button>
      </div>
      <div class="space-y-4">
        <div class="flex items-center justify-between bg-slate-50 p-3 rounded-xl">
          <span class="text-sm font-semibold text-slate-700">終了時にアラームを鳴らす</span>
          <input type="checkbox" id="alarm-enabled-checkbox" class="w-5 h-5 accent-indigo-600 rounded cursor-pointer" checked />
        </div>

        <div>
          <label for="alarm-sound-select" class="block text-xs font-bold text-slate-700 mb-1">アラーム音の種類</label>
          <select id="alarm-sound-select" class="w-full rounded-xl border border-slate-300 bg-white px-3 py-2.5 text-sm">
            <option value="chime">やさしいチャイム（和音ベル）</option>
            <option value="digital">電子ピピピ音（デジタル時計風）</option>
            <option value="school">学校のチャイム風（キンコンカンコン）</option>
            <option value="soft">ソフトトーン（穏やかな単音）</option>
          </select>
        </div>

        <div>
          <div class="flex justify-between text-xs font-bold text-slate-700 mb-1">
            <span>音量</span>
            <span id="alarm-volume-label">70%</span>
          </div>
          <input type="range" id="alarm-volume-range" min="0" max="100" value="70" class="w-full accent-indigo-600" />
        </div>

        <button type="button" id="test-alarm-sound-btn" class="w-full py-2.5 px-4 rounded-xl bg-slate-100 hover:bg-slate-200 text-slate-800 text-xs font-bold flex items-center justify-center gap-2 transition-colors">
          <i class="fa-solid fa-volume-high text-indigo-600"></i>
          <span>アラーム音をテスト再生する</span>
        </button>

        <div class="flex justify-end gap-2 pt-2 border-t border-slate-100">
          <button type="button" id="save-alarm-settings-btn" class="px-5 py-2.5 rounded-xl text-xs font-bold bg-indigo-600 hover:bg-indigo-700 text-white">設定を保存</button>
        </div>
      </div>
    </div>
  </div>

  <!-- 3. 手入力での勉強記録追加モーダル -->
  <div id="manual-log-modal" class="hidden fixed inset-0 z-50 bg-slate-900/50 backdrop-blur-xs flex items-center justify-center p-4">
    <div class="bg-white rounded-2xl max-w-md w-full p-6 shadow-xl border border-slate-200 space-y-4">
      <div class="flex items-center justify-between">
        <h3 class="text-base font-bold text-slate-900 flex items-center gap-2">
          <i class="fa-solid fa-pen-to-square text-indigo-600"></i>
          <span>勉強時間を手入力で記録</span>
        </h3>
        <button type="button" onclick="closeModal('manual-log-modal')" class="text-slate-400 hover:text-slate-700 p-1">
          <i class="fa-solid fa-xmark text-lg"></i>
        </button>
      </div>
      <form id="manual-log-form" class="space-y-3">
        <div>
          <label for="manual-log-date" class="block text-xs font-bold text-slate-700 mb-1">学習日 <span class="text-rose-500">*</span></label>
          <input type="date" id="manual-log-date" required class="w-full rounded-xl border border-slate-300 px-3 py-2 text-sm" />
        </div>
        <div>
          <label for="manual-log-subject" class="block text-xs font-bold text-slate-700 mb-1">科目 <span class="text-rose-500">*</span></label>
          <select id="manual-log-subject" class="subject-select-target w-full rounded-xl border border-slate-300 bg-white px-3 py-2.5 text-sm font-medium">
            <!-- JSで生成 -->
          </select>
        </div>
        <div>
          <label for="manual-log-material" class="block text-xs font-bold text-slate-700 mb-1">使用した教材</label>
          <select id="manual-log-material" class="w-full rounded-xl border border-slate-300 bg-white px-3 py-2.5 text-sm">
            <option value="">（教材指定なし / 自由演習）</option>
          </select>
        </div>
        <div>
          <label for="manual-log-minutes" class="block text-xs font-bold text-slate-700 mb-1">勉強時間（分） <span class="text-rose-500">*</span></label>
          <input type="number" id="manual-log-minutes" min="1" max="1440" value="60" required class="w-full rounded-xl border border-slate-300 px-3 py-2.5 text-sm font-bold" />
        </div>
        <div class="flex justify-end gap-2 pt-2">
          <button type="button" onclick="closeModal('manual-log-modal')" class="px-4 py-2 rounded-xl text-xs font-semibold text-slate-600 hover:bg-slate-100">キャンセル</button>
          <button type="submit" class="px-5 py-2.5 rounded-xl text-xs font-bold bg-indigo-600 hover:bg-indigo-700 text-white">記録を追加する</button>
        </div>
      </form>
    </div>
  </div>

  <!-- 4. データ管理モーダル -->
  <div id="data-modal" class="hidden fixed inset-0 z-50 bg-slate-900/50 backdrop-blur-xs flex items-center justify-center p-4">
    <div class="bg-white rounded-2xl max-w-md w-full p-6 shadow-xl border border-slate-200 space-y-4">
      <div class="flex items-center justify-between">
        <h3 class="text-base font-bold text-slate-900">データ管理・サンプル操作</h3>
        <button type="button" onclick="closeModal('data-modal')" class="text-slate-400 hover:text-slate-700 p-1">
          <i class="fa-solid fa-xmark text-lg"></i>
        </button>
      </div>
      <p class="text-xs text-slate-500 leading-relaxed">
        すべての目標設定・学習ログ・教材・過去問データはブラウザ内に自動保存されています。動作確認用のサンプルデータを読み込むことや、全データを初期化することも可能です。
      </p>
      <div class="space-y-2.5 pt-1">
        <button type="button" id="load-sample-data-btn" class="w-full py-2.5 px-4 rounded-xl bg-indigo-50 hover:bg-indigo-100 text-indigo-700 text-xs font-bold flex items-center justify-center gap-2 transition-colors">
          <i class="fa-solid fa-wand-magic-sparkles"></i>
          <span>サンプルデータを読み込む（1週間分の記録等）</span>
        </button>
        <button type="button" id="reset-all-data-btn" class="w-full py-2.5 px-4 rounded-xl bg-rose-50 hover:bg-rose-100 text-rose-700 text-xs font-bold flex items-center justify-center gap-2 transition-colors">
          <i class="fa-solid fa-trash-can"></i>
          <span>すべてのデータを初期化する</span>
        </button>
      </div>
    </div>
  </div>

  <!-- ===================================================================== -->
  <!-- アプリケーションロジック（JavaScript）                                -->
  <!-- ===================================================================== -->
  <script>
    /**
     * 1. 全19科目のマスター定義（3つの大枠グループ + 固有カラーコーディング）
     */
    const SUBJECT_GROUPS = [
      {
        groupName: '文理共通',
        subjects: [
          { id: 'eng_listening', name: '英語リスニング', color: '#0EA5E9', bgSoft: '#E0F2FE', textDark: '#0369A1' },
          { id: 'eng_reading',   name: '英語リーディング', color: '#2563EB', bgSoft: '#DBEAFE', textDark: '#1D4ED8' },
          { id: 'modern_jp',     name: '現代文',           color: '#F43F5E', bgSoft: '#FFE4E6', textDark: '#BE123C' },
          { id: 'math_1',        name: '数学1',            color: '#4F46E5', bgSoft: '#E0E7FF', textDark: '#4338CA' },
          { id: 'math_2',        name: '数学2',            color: '#7C3AED', bgSoft: '#EDE9FE', textDark: '#6D28D9' },
          { id: 'info',          name: '情報',             color: '#06B6D4', bgSoft: '#CFFAFE', textDark: '#0E7490' }
        ]
      },
      {
        groupName: '文系科目',
        subjects: [
          { id: 'classic_jp',    name: '古典',             color: '#EC4899', bgSoft: '#FCE7F3', textDark: '#BE185D' },
          { id: 'geography',     name: '地理',             color: '#D97706', bgSoft: '#FEF3C7', textDark: '#B45309' },
          { id: 'world_history', name: '世界史',           color: '#EA580C', bgSoft: '#FFEDD5', textDark: '#C2410C' },
          { id: 'jp_history',    name: '日本史',           color: '#DC2626', bgSoft: '#FEE2E2', textDark: '#B91C1C' },
          { id: 'public_civics', name: '公共',             color: '#F59E0B', bgSoft: '#FEF3C7', textDark: '#B45309' },
          { id: 'ethics',        name: '倫理',             color: '#A855F7', bgSoft: '#F3E8FF', textDark: '#7E22CE' },
          { id: 'pol_econ',      name: '政治・経済',       color: '#E11D48', bgSoft: '#FFE4E6', textDark: '#9F1239' },
          { id: 'essay',         name: '小論文',           color: '#64748B', bgSoft: '#F1F5F9', textDark: '#334155' }
        ]
      },
      {
        groupName: '理系科目',
        subjects: [
          { id: 'physics',       name: '物理',             color: '#0284C7', bgSoft: '#E0F2FE', textDark: '#0369A1' },
          { id: 'chemistry',     name: '化学',             color: '#10B981', bgSoft: '#D1FAE5', textDark: '#047857' },
          { id: 'biology',       name: '生物',             color: '#16A34A', bgSoft: '#DCFCE7', textDark: '#15803D' },
          { id: 'earth_science', name: '地学',             color: '#84CC16', bgSoft: '#ECFCCB', textDark: '#4D7C0F' },
          { id: 'math_3',        name: '数学3',            color: '#9333EA', bgSoft: '#F3E8FF', textDark: '#7E22CE' }
        ]
      }
    ];

    // 高速ルックアップ用マップ
    const SUBJECT_MAP = {};
    SUBJECT_GROUPS.forEach(group => {
      group.subjects.forEach(subj => {
        SUBJECT_MAP[subj.id] = { ...subj, groupName: group.groupName };
      });
    });

    /**
     * 2. 日付ユーティリティ（ローカルタイムゾーン基準 YYYY-MM-DD）
     */
    function getLocalDateStr(dateObj = new Date()) {
      const y = dateObj.getFullYear();
      const m = String(dateObj.getMonth() + 1).padStart(2, '0');
      const d = String(dateObj.getDate()).padStart(2, '0');
      return `${y}-${m}-${d}`;
    }

    function addDays(dateObj, days) {
      const copy = new Date(dateObj);
      copy.setDate(copy.getDate() + days);
      return copy;
    }

    function calcDaysRemaining(targetDateStr) {
      if (!targetDateStr) return null;
      const today = new Date();
      today.setHours(0, 0, 0, 0);
      const [y, m, d] = targetDateStr.split('-').map(Number);
      const target = new Date(y, m - 1, d);
      target.setHours(0, 0, 0, 0);
      const diffMs = target - today;
      return Math.ceil(diffMs / (1000 * 60 * 60 * 24));
    }

    /**
     * 3. アプリケーション状態と localStorage 自動保存
     */
    const STORAGE_KEY = 'studylog_pro_data_v1';

    let appState = {
      goals: {
        dailyMinutes: 180,
        milestone: { title: '第2回 全統記述模試', date: getLocalDateStr(addDays(new Date(), 24)) },
        goal: { title: '第一志望大学 合格（本番試験）', date: getLocalDateStr(addDays(new Date(), 95)) }
      },
      alarm: {
        enabled: true,
        sound: 'chime',
        volume: 70
      },
      materials: [], // { id, name, subjectId, memo, coverImage, createdAt }
      logs: [],      // { id, date, subjectId, materialId, minutes, createdAt }
      pastExams: []  // { id, year, univ, subjectId, minutes, accuracy, createdAt }
    };

    function saveState() {
      try {
        localStorage.setItem(STORAGE_KEY, JSON.stringify(appState));
        const badge = document.getElementById('autosave-badge');
        if (badge) {
          badge.innerHTML = `<span class="w-1.5 h-1.5 rounded-full bg-emerald-500"></span> 自動保存済`;
        }
      } catch (e) {
        console.error('保存エラー:', e);
        showToast('ブラウザの保存容量上限に達しました。不要な画像を削除してください。', 'error');
      }
    }

    function loadState() {
      const raw = localStorage.getItem(STORAGE_KEY);
      if (raw) {
        try {
          const parsed = JSON.parse(raw);
          appState = {
            ...appState,
            ...parsed,
            goals: { ...appState.goals, ...(parsed.goals || {}) },
            alarm: { ...appState.alarm, ...(parsed.alarm || {}) }
          };
          return;
        } catch (e) {
          console.error('読込エラー:', e);
        }
      }
      // 初回起動時はわかりやすいサンプルデータを投入
      seedSampleData(false);
    }

    function seedSampleData(notify = true) {
      const today = new Date();
      appState.materials = [
        { id: 'mat_1', name: 'システム英単語 5訂版', subjectId: 'eng_reading', memo: '毎日100語チェック', coverImage: '' },
        { id: 'mat_2', name: '青チャート 数学I+A', subjectId: 'math_1', memo: '例題コンパス4まで', coverImage: '' },
        { id: 'mat_3', name: '物理のエッセンス 力学・波動', subjectId: 'physics', memo: '2周目演習中', coverImage: '' },
        { id: 'mat_4', name: '化学重要問題集', subjectId: 'chemistry', memo: 'A問題を完璧にする', coverImage: '' }
      ];

      appState.logs = [
        { id: 'log_1', date: getLocalDateStr(addDays(today, -6)), subjectId: 'eng_reading', materialId: 'mat_1', minutes: 90 },
        { id: 'log_2', date: getLocalDateStr(addDays(today, -6)), subjectId: 'math_1', materialId: 'mat_2', minutes: 100 },
        { id: 'log_3', date: getLocalDateStr(addDays(today, -5)), subjectId: 'physics', materialId: 'mat_3', minutes: 120 },
        { id: 'log_4', date: getLocalDateStr(addDays(today, -5)), subjectId: 'modern_jp', materialId: '', minutes: 60 },
        { id: 'log_5', date: getLocalDateStr(addDays(today, -4)), subjectId: 'chemistry', materialId: 'mat_4', minutes: 110 },
        { id: 'log_6', date: getLocalDateStr(addDays(today, -4)), subjectId: 'eng_listening', materialId: '', minutes: 45 },
        { id: 'log_7', date: getLocalDateStr(addDays(today, -3)), subjectId: 'math_2', materialId: '', minutes: 130 },
        { id: 'log_8', date: getLocalDateStr(addDays(today, -3)), subjectId: 'eng_reading', materialId: 'mat_1', minutes: 70 },
        { id: 'log_9', date: getLocalDateStr(addDays(today, -2)), subjectId: 'math_3', materialId: '', minutes: 140 },
        { id: 'log_10', date: getLocalDateStr(addDays(today, -2)), subjectId: 'jp_history', materialId: '', minutes: 50 },
        { id: 'log_11', date: getLocalDateStr(addDays(today, -1)), subjectId: 'physics', materialId: 'mat_3', minutes: 95 },
        { id: 'log_12', date: getLocalDateStr(addDays(today, -1)), subjectId: 'chemistry', materialId: 'mat_4', minutes: 95 },
        { id: 'log_13', date: getLocalDateStr(today), subjectId: 'eng_reading', materialId: 'mat_1', minutes: 60 },
        { id: 'log_14', date: getLocalDateStr(today), subjectId: 'math_1', materialId: 'mat_2', minutes: 65 }
      ];

      appState.pastExams = [
        { id: 'pe_1', year: '2025年度', univ: '東京大学 理科一類', subjectId: 'eng_reading', minutes: 120, accuracy: 76.5 },
        { id: 'pe_2', year: '2024年度', univ: '東京大学 理科一類', subjectId: 'math_3', minutes: 150, accuracy: 68.0 },
        { id: 'pe_3', year: '2025年度', univ: '早稲田大学 先進理工学部', subjectId: 'physics', minutes: 90, accuracy: 82.0 }
      ];

      saveState();
      refreshAllUI();
      if (notify) showToast('サンプルデータを読み込みました', 'success');
    }

    /**
     * 4. トースト通知システム（alert()不使用）
     */
    function showToast(message, type = 'info') {
      const container = document.getElementById('toast-container');
      const toast = document.createElement('div');
      const colors = {
        success: 'bg-emerald-600 text-white',
        error: 'bg-rose-600 text-white',
        info: 'bg-slate-800 text-white'
      };
      const icons = {
        success: 'fa-circle-check',
        error: 'fa-circle-exclamation',
        info: 'fa-circle-info'
      };
      toast.className = `${colors[type] || colors.info} px-4 py-3 rounded-xl shadow-lg text-xs sm:text-sm font-semibold flex items-center gap-2.5 pointer-events-auto transition-all duration-300 transform translate-y-0 opacity-100`;
      toast.innerHTML = `<i class="fa-solid ${icons[type] || icons.info}"></i><span>${escapeHTML(message)}</span>`;
      container.appendChild(toast);

      setTimeout(() => {
        toast.classList.add('opacity-0', '-translate-y-2');
        setTimeout(() => toast.remove(), 300);
      }, 3200);
    }

    function escapeHTML(str) {
      return String(str)
        .replace(/&/g, '&amp;')
        .replace(/</g, '&lt;')
        .replace(/>/g, '&gt;')
        .replace(/"/g, '&quot;');
    }

    /**
     * 5. 科目セレクトボックス（3つの大枠 optgroup）の初期化
     */
    function populateSubjectSelects() {
      const targets = document.querySelectorAll('.subject-select-target');
      targets.forEach(select => {
        const currentVal = select.value;
        select.innerHTML = '';
        SUBJECT_GROUPS.forEach(group => {
          const optgroup = document.createElement('optgroup');
          optgroup.label = `【${group.groupName}】`;
          group.subjects.forEach(subj => {
            const opt = document.createElement('option');
            opt.value = subj.id;
            opt.textContent = subj.name;
            optgroup.appendChild(opt);
          });
          select.appendChild(optgroup);
        });
        if (currentVal && SUBJECT_MAP[currentVal]) {
          select.value = currentVal;
        } else {
          select.value = 'eng_reading';
        }
      });

      // フィルタ用セレクトにも追加
      ['bookshelf-filter-subject', 'log-filter-subject'].forEach(id => {
        const sel = document.getElementById(id);
        if (!sel) return;
        const prev = sel.value;
        sel.innerHTML = `<option value="ALL">${id === 'bookshelf-filter-subject' ? 'すべての科目を表示' : '全科目で絞り込み'}</option>`;
        SUBJECT_GROUPS.forEach(group => {
          const optgroup = document.createElement('optgroup');
          optgroup.label = `【${group.groupName}】`;
          group.subjects.forEach(subj => {
            const opt = document.createElement('option');
            opt.value = subj.id;
            opt.textContent = subj.name;
            optgroup.appendChild(opt);
          });
          sel.appendChild(optgroup);
        });
        if (prev) sel.value = prev;
      });
    }

    function renderSubjectBadge(subjectId) {
      const s = SUBJECT_MAP[subjectId] || { name: '未分類', color: '#64748B', bgSoft: '#F1F5F9', textDark: '#334155' };
      return `<span class="inline-flex items-center gap-1.5 px-2.5 py-0.5 rounded-full text-xs font-bold" style="background-color:${s.bgSoft}; color:${s.textDark};">
        <span class="w-2 h-2 rounded-full" style="background-color:${s.color};"></span>
        ${escapeHTML(s.name)}
      </span>`;
    }

    /**
     * 6. ページ切替タブ制御
     */
    let currentTab = 'home';
    function switchTab(tabName) {
      currentTab = tabName;
      document.querySelectorAll('.page-section').forEach(sec => sec.classList.add('hidden'));
      const targetSec = document.getElementById(`page-${tabName}`);
      if (targetSec) targetSec.classList.remove('hidden');

      document.querySelectorAll('.nav-tab-btn').forEach(btn => {
        const isMatch = btn.dataset.tab === tabName;
        if (btn.closest('nav').classList.contains('hidden')) {
          // デスクトップタブ
          btn.className = isMatch
            ? 'nav-tab-btn px-4 py-2 rounded-lg text-sm font-semibold transition-all flex items-center gap-2 bg-white text-indigo-600 shadow-xs'
            : 'nav-tab-btn px-4 py-2 rounded-lg text-sm font-semibold transition-all flex items-center gap-2 text-slate-600 hover:text-slate-900';
        } else {
          // モバイルタブ
          btn.className = isMatch
            ? 'nav-tab-btn flex-1 py-2 px-2 rounded-xl text-xs font-semibold transition-all flex flex-col items-center gap-1 bg-indigo-50 text-indigo-600 border border-indigo-200'
            : 'nav-tab-btn flex-1 py-2 px-2 rounded-xl text-xs font-semibold transition-all flex flex-col items-center gap-1 text-slate-600 bg-slate-50 border border-transparent';
        }
      });

      if (tabName === 'analytics') {
        renderCharts();
        renderPastExams();
      } else if (tabName === 'materials') {
        renderBookshelf();
        renderStudyLogsTable();
      } else if (tabName === 'home') {
        renderGoals();
        renderCalendar();
      }
    }

    /**
     * 7. 目標設定と達成率レンダリング
     */
    function renderGoals() {
      const todayStr = getLocalDateStr();
      const todayMinutes = appState.logs
        .filter(l => l.date === todayStr)
        .reduce((sum, l) => sum + Number(l.minutes), 0);

      const targetMin = Math.max(1, Number(appState.goals.dailyMinutes) || 180);
      const pct = Math.min(100, Math.round((todayMinutes / targetMin) * 100));
      const remain = Math.max(0, targetMin - todayMinutes);

      document.getElementById('daily-current-min').textContent = todayMinutes;
      document.getElementById('daily-target-min').textContent = targetMin;
      const pctBadge = document.getElementById('daily-percent-badge');
      pctBadge.textContent = `${Math.round((todayMinutes / targetMin) * 100)}%`;
      pctBadge.className = todayMinutes >= targetMin
        ? 'text-xs font-bold px-2.5 py-1 rounded-full bg-emerald-100 text-emerald-700'
        : 'text-xs font-bold px-2.5 py-1 rounded-full bg-slate-100 text-slate-600';

      document.getElementById('daily-progress-bar').style.width = `${pct}%`;
      document.getElementById('daily-remaining-text').textContent = remain === 0
        ? '🎉 本日のデイリー目標を達成しました！'
        : `目標達成まであと ${remain} 分`;

      // マイルストーン
      const ms = appState.goals.milestone;
      document.getElementById('milestone-title-display').textContent = ms.title || '未設定';
      document.getElementById('milestone-date-display').textContent = ms.date ? `目標日: ${ms.date}` : '目標日: 未設定';
      const msDays = calcDaysRemaining(ms.date);
      const msBox = document.getElementById('milestone-countdown-box');
      if (msDays === null) {
        msBox.innerHTML = `<span class="text-sm font-bold text-slate-400">日付未設定</span>`;
      } else if (msDays > 0) {
        msBox.innerHTML = `<span class="text-xs text-slate-500">あと</span><span class="text-2xl sm:text-3xl font-bold text-amber-600 mx-1">${msDays}</span><span class="text-xs font-bold text-slate-700">日</span>`;
      } else if (msDays === 0) {
        msBox.innerHTML = `<span class="text-base font-bold text-rose-600">本日が目標日！</span>`;
      } else {
        msBox.innerHTML = `<span class="text-xs font-semibold text-slate-400">目標日から ${Math.abs(msDays)}日経過</span>`;
      }

      // ゴール
      const gl = appState.goals.goal;
      document.getElementById('goal-title-display').textContent = gl.title || '未設定';
      document.getElementById('goal-date-display').textContent = gl.date ? `目標日: ${gl.date}` : '目標日: 未設定';
      const glDays = calcDaysRemaining(gl.date);
      const glBox = document.getElementById('goal-countdown-box');
      if (glDays === null) {
        glBox.innerHTML = `<span class="text-sm font-bold text-slate-400">日付未設定</span>`;
      } else if (glDays > 0) {
        glBox.innerHTML = `<span class="text-xs text-slate-500">あと</span><span class="text-2xl sm:text-3xl font-bold text-indigo-600 mx-1">${glDays}</span><span class="text-xs font-bold text-slate-700">日</span>`;
      } else if (glDays === 0) {
        glBox.innerHTML = `<span class="text-base font-bold text-rose-600">本日が本番！</span>`;
      } else {
        glBox.innerHTML = `<span class="text-xs font-semibold text-slate-400">目標日から ${Math.abs(glDays)}日経過</span>`;
      }
    }

    function openGoalModal(type) {
      document.getElementById('goal-modal-type').value = type;
      const dailyFields = document.getElementById('goal-daily-fields');
      const eventFields = document.getElementById('goal-event-fields');
      const titleEl = document.getElementById('goal-modal-title');

      if (type === 'daily') {
        titleEl.textContent = 'デイリー目標（1日の目標時間）の設定';
        dailyFields.classList.remove('hidden');
        eventFields.classList.add('hidden');
        document.getElementById('input-daily-minutes').value = appState.goals.dailyMinutes;
      } else if (type === 'milestone') {
        titleEl.textContent = 'マイルストーン目標の設定';
        dailyFields.classList.add('hidden');
        eventFields.classList.remove('hidden');
        document.getElementById('input-event-title').value = appState.goals.milestone.title || '';
        document.getElementById('input-event-date').value = appState.goals.milestone.date || '';
      } else {
        titleEl.textContent = 'ゴール目標（最終目標）の設定';
        dailyFields.classList.add('hidden');
        eventFields.classList.remove('hidden');
        document.getElementById('input-event-title').value = appState.goals.goal.title || '';
        document.getElementById('input-event-date').value = appState.goals.goal.date || '';
      }
      document.getElementById('goal-modal').classList.remove('hidden');
    }

    function closeModal(modalId) {
      document.getElementById(modalId).classList.add('hidden');
    }

    /**
     * 8. カウントアップ / カウントダウン学習タイマー & Web Audio API アラーム
     */
    let timerState = {
      mode: 'countdown', // 'countdown' | 'countup'
      targetSeconds: 25 * 60,
      remainingSeconds: 25 * 60,
      elapsedSeconds: 0,
      isRunning: false,
      intervalId: null
    };

    function playAlarmSound(soundType = appState.alarm.sound, volumePercent = appState.alarm.volume) {
      try {
        const AudioCtx = window.AudioContext || window.webkitAudioContext;
        const ctx = new AudioCtx();
        const gainVal = Math.max(0.01, (volumePercent / 100) * 0.4);

        const playNote = (freq, startTime, duration, type = 'sine') => {
          const osc = ctx.createOscillator();
          const gain = ctx.createGain();
          osc.type = type;
          osc.frequency.setValueAtTime(freq, startTime);
          gain.gain.setValueAtTime(gainVal, startTime);
          gain.gain.exponentialRampToValueAtTime(0.001, startTime + duration);
          osc.connect(gain);
          gain.connect(ctx.destination);
          osc.start(startTime);
          osc.stop(startTime + duration);
        };

        const now = ctx.currentTime;
        if (soundType === 'chime') {
          playNote(523.25, now, 0.5, 'sine');       // C5
          playNote(659.25, now + 0.18, 0.5, 'sine'); // E5
          playNote(783.99, now + 0.36, 0.7, 'sine'); // G5
          playNote(1046.5, now + 0.54, 0.9, 'sine'); // C6
        } else if (soundType === 'digital') {
          for (let i = 0; i < 4; i++) {
            playNote(880, now + i * 0.22, 0.12, 'square');
          }
        } else if (soundType === 'school') {
          playNote(698.46, now, 0.55, 'triangle');       // F5
          playNote(554.37, now + 0.55, 0.55, 'triangle'); // C#5
          playNote(622.25, now + 1.1, 0.55, 'triangle');  // D#5
          playNote(415.30, now + 1.65, 0.9, 'triangle');  // G#4
        } else {
          playNote(587.33, now, 0.8, 'sine');
          playNote(587.33, now + 0.9, 0.8, 'sine');
        }
      } catch (e) {
        console.warn('Web Audio再生制限:', e);
      }
    }

    function formatSeconds(sec) {
      const total = Math.max(0, Math.floor(sec));
      const h = Math.floor(total / 3600);
      const m = Math.floor((total % 3600) / 60);
      const s = total % 60;
      if (h > 0) {
        return `${String(h).padStart(2, '0')}:${String(m).padStart(2, '0')}:${String(s).padStart(2, '0')}`;
      }
      return `${String(m).padStart(2, '0')}:${String(s).padStart(2, '0')}`;
    }

    function updateTimerUI() {
      const displaySec = timerState.mode === 'countdown' ? timerState.remainingSeconds : timerState.elapsedSeconds;
      document.getElementById('timer-display').textContent = formatSeconds(displaySec);

      const recordedMins = Math.floor(timerState.elapsedSeconds / 60);
      const saveBtnLabel = document.getElementById('timer-save-btn-label');
      if (timerState.elapsedSeconds > 0 && recordedMins === 0) {
        saveBtnLabel.textContent = `学習時間を記録する（1分未満→1分として記録）`;
      } else {
        saveBtnLabel.textContent = `学習時間を記録する（${recordedMins}分）`;
      }

      const startBtn = document.getElementById('timer-start-pause-btn');
      if (timerState.isRunning) {
        startBtn.innerHTML = `<i class="fa-solid fa-pause"></i><span>一時停止</span>`;
        startBtn.classList.replace('bg-indigo-600', 'bg-amber-600');
        startBtn.classList.replace('hover:bg-indigo-700', 'hover:bg-amber-700');
      } else {
        startBtn.innerHTML = `<i class="fa-solid fa-play"></i><span>スタート</span>`;
        startBtn.classList.replace('bg-amber-600', 'bg-indigo-600');
        startBtn.classList.replace('hover:bg-amber-700', 'hover:bg-indigo-700');
      }

      // 選択科目のバッジ反映
      const subjId = document.getElementById('timer-subject-select').value;
      const subj = SUBJECT_MAP[subjId];
      if (subj) {
        document.getElementById('timer-subject-label').textContent = subj.name;
        document.getElementById('timer-subject-dot').style.backgroundColor = subj.color;
      }
    }

    function setTimerMinutes(mins) {
      const clamped = Math.min(300, Math.max(1, Number(mins) || 25));
      document.getElementById('timer-minutes-input').value = clamped;
      if (!timerState.isRunning) {
        timerState.targetSeconds = clamped * 60;
        timerState.remainingSeconds = clamped * 60;
        timerState.elapsedSeconds = 0;
        updateTimerUI();
      }
    }

    function toggleTimer() {
      if (timerState.isRunning) {
        clearInterval(timerState.intervalId);
        timerState.isRunning = false;
        document.getElementById('timer-status-text').textContent = '一時停止中';
        updateTimerUI();
        return;
      }

      if (timerState.mode === 'countdown' && timerState.remainingSeconds <= 0) {
        const mins = Number(document.getElementById('timer-minutes-input').value) || 25;
        timerState.remainingSeconds = mins * 60;
        timerState.elapsedSeconds = 0;
      }

      timerState.isRunning = true;
      document.getElementById('timer-status-text').textContent = '計測中... 集中して取り組みましょう';
      updateTimerUI();

      timerState.intervalId = setInterval(() => {
        timerState.elapsedSeconds += 1;
        if (timerState.mode === 'countdown') {
          timerState.remainingSeconds -= 1;
          if (timerState.remainingSeconds <= 0) {
            clearInterval(timerState.intervalId);
            timerState.isRunning = false;
            timerState.remainingSeconds = 0;
            document.getElementById('timer-status-text').textContent = '⏰ 設定時間が終了しました！記録ボタンを押してください';
            if (appState.alarm.enabled) {
              playAlarmSound();
            }
            showToast('タイマー終了！お疲れさまでした。学習を記録しましょう。', 'success');
          }
        }
        updateTimerUI();
      }, 1000);
    }

    function resetTimer() {
      clearInterval(timerState.intervalId);
      timerState.isRunning = false;
      timerState.elapsedSeconds = 0;
      const mins = Number(document.getElementById('timer-minutes-input').value) || 25;
      timerState.targetSeconds = mins * 60;
      timerState.remainingSeconds = mins * 60;
      document.getElementById('timer-status-text').textContent = 'リセットしました';
      updateTimerUI();
    }

    function recordTimerSession() {
      if (timerState.elapsedSeconds <= 0) {
        showToast('計測時間が0秒です。タイマーを動かしてから記録してください。', 'error');
        return;
      }
      if (timerState.isRunning) {
        clearInterval(timerState.intervalId);
        timerState.isRunning = false;
      }
      const minutesToSave = Math.max(1, Math.round(timerState.elapsedSeconds / 60));
      const subjectId = document.getElementById('timer-subject-select').value;
      const materialId = document.getElementById('timer-material-select').value;

      appState.logs.unshift({
        id: 'log_' + Date.now(),
        date: getLocalDateStr(),
        subjectId,
        materialId,
        minutes: minutesToSave
      });

      saveState();
      const subjName = SUBJECT_MAP[subjectId]?.name || '';
      const matName = appState.materials.find(m => m.id === materialId)?.name || '教材指定なし';
      showToast(`記録完了：「${subjName} / ${matName}」を ${minutesToSave}分 保存しました！`, 'success');

      resetTimer();
      refreshAllUI();
    }

    /**
     * 9. カレンダー描画 & 日付詳細表示
     */
    let calendarViewDate = new Date();
    let selectedCalendarDateStr = getLocalDateStr();

    function renderCalendar() {
      const year = calendarViewDate.getFullYear();
      const month = calendarViewDate.getMonth();
      document.getElementById('cal-month-title').textContent = `${year}年 ${month + 1}月`;

      const grid = document.getElementById('calendar-grid');
      grid.innerHTML = '';

      const firstDay = new Date(year, month, 1).getDay();
      const daysInMonth = new Date(year, month + 1, 0).getDate();
      const todayStr = getLocalDateStr();
      const dailyGoal = Number(appState.goals.dailyMinutes) || 180;

      // 日付ごとの合計時間マップ
      const dailyTotals = {};
      appState.logs.forEach(log => {
        dailyTotals[log.date] = (dailyTotals[log.date] || 0) + Number(log.minutes);
      });

      // 前月空白セル
      for (let i = 0; i < firstDay; i++) {
        const empty = document.createElement('div');
        empty.className = 'h-14 sm:h-16 rounded-xl bg-slate-50/50';
        grid.appendChild(empty);
      }

      // 各日付セル
      for (let d = 1; d <= daysInMonth; d++) {
        const dateStr = `${year}-${String(month + 1).padStart(2, '0')}-${String(d).padStart(2, '0')}`;
        const totalMin = dailyTotals[dateStr] || 0;
        const isAchieved = totalMin >= dailyGoal && totalMin > 0;
        const hasStudy = totalMin > 0;
        const isSelected = dateStr === selectedCalendarDateStr;
        const isToday = dateStr === todayStr;

        const btn = document.createElement('button');
        btn.type = 'button';
        let baseClass = 'h-14 sm:h-16 rounded-xl p-1.5 flex flex-col justify-between items-center border transition-all text-left relative ';
        if (isAchieved) {
          baseClass += 'bg-emerald-50 border-emerald-300 text-emerald-900 hover:bg-emerald-100 ';
        } else if (hasStudy) {
          baseClass += 'bg-indigo-50/70 border-indigo-200 text-indigo-900 hover:bg-indigo-100 ';
        } else {
          baseClass += 'bg-white border-slate-100 text-slate-700 hover:bg-slate-50 ';
        }
        if (isSelected) {
          baseClass += 'ring-2 ring-indigo-600 ring-offset-1 ';
        }

        btn.className = baseClass;
        btn.innerHTML = `
          <div class="w-full flex items-center justify-between">
            <span class="text-xs font-bold ${isToday ? 'bg-indigo-600 text-white w-5 h-5 rounded-full flex items-center justify-center' : ''}">${d}</span>
            ${isAchieved ? '<i class="fa-solid fa-circle-check text-emerald-600 text-[11px]" title="目標達成"></i>' : ''}
          </div>
          <div class="w-full text-center">
            ${hasStudy
              ? `<span class="text-[10px] sm:text-xs font-bold ${isAchieved ? 'text-emerald-700' : 'text-indigo-700'}">${totalMin}分</span>`
              : `<span class="text-[10px] text-slate-300">-</span>`}
          </div>
        `;
        btn.addEventListener('click', () => {
          selectedCalendarDateStr = dateStr;
          renderCalendar();
          renderSelectedDateDetails();
        });
        grid.appendChild(btn);
      }

      renderSelectedDateDetails();
    }

    function renderSelectedDateDetails() {
      const logs = appState.logs.filter(l => l.date === selectedCalendarDateStr);
      const total = logs.reduce((sum, l) => sum + Number(l.minutes), 0);
      document.getElementById('selected-date-heading').textContent = `${selectedCalendarDateStr} の学習記録`;
      document.getElementById('selected-date-total').textContent = `合計 ${total}分`;

      const container = document.getElementById('selected-date-details');
      if (logs.length === 0) {
        container.innerHTML = `<p class="text-xs text-slate-400 py-2">この日の学習記録はありません。</p>`;
        return;
      }

      container.innerHTML = logs.map(log => {
        const mat = appState.materials.find(m => m.id === log.materialId);
        return `
          <div class="flex items-center justify-between bg-slate-50 px-3 py-2 rounded-xl text-xs">
            <div class="flex items-center gap-2 min-w-0">
              ${renderSubjectBadge(log.subjectId)}
              <span class="text-slate-600 truncate">${mat ? escapeHTML(mat.name) : '教材指定なし'}</span>
            </div>
            <span class="font-bold text-slate-800 shrink-0 ml-2">${log.minutes}分</span>
          </div>
        `;
      }).join('');
    }

    /**
     * 10. グラフ描画（ドーナツグラフ & 1週間積み上げ棒グラフ）
     * ※仕様厳守：グラフに記録・表示されるのは「科目」のみとし、教材名は出さない
     */
    let ratioChartInstance = null;
    let weeklyChartInstance = null;

    function renderCharts() {
      const period = document.getElementById('chart-period-select').value;
      const today = new Date();
      today.setHours(0, 0, 0, 0);

      const filteredLogs = appState.logs.filter(log => {
        if (period === 'all') return true;
        const [y, m, d] = log.date.split('-').map(Number);
        const logDate = new Date(y, m - 1, d);
        if (period === 'week') {
          const sevenDaysAgo = addDays(today, -6);
          return logDate >= sevenDaysAgo;
        }
        if (period === 'month') {
          return logDate.getFullYear() === today.getFullYear() && logDate.getMonth() === today.getMonth();
        }
        return true;
      });

      // 科目別集計
      const subjectTotals = {};
      filteredLogs.forEach(log => {
        subjectTotals[log.subjectId] = (subjectTotals[log.subjectId] || 0) + Number(log.minutes);
      });

      const sortedEntries = Object.entries(subjectTotals).sort((a, b) => b[1] - a[1]);
      const grandTotal = sortedEntries.reduce((s, [, v]) => s + v, 0);

      const ratioCtx = document.getElementById('subject-ratio-chart').getContext('2d');
      if (ratioChartInstance) ratioChartInstance.destroy();

      const emptyEl = document.getElementById('ratio-chart-empty');
      const legendEl = document.getElementById('subject-ratio-legend');

      if (sortedEntries.length === 0) {
        emptyEl.classList.remove('hidden');
        legendEl.innerHTML = `<p class="text-slate-400 text-center py-2">表示するデータがありません</p>`;
      } else {
        emptyEl.classList.add('hidden');
        const labels = sortedEntries.map(([id]) => SUBJECT_MAP[id]?.name || id);
        const dataVals = sortedEntries.map(([, val]) => val);
        const bgColors = sortedEntries.map(([id]) => SUBJECT_MAP[id]?.color || '#64748B');

        ratioChartInstance = new Chart(ratioCtx, {
          type: 'doughnut',
          data: {
            labels,
            datasets: [{
              data: dataVals,
              backgroundColor: bgColors,
              borderWidth: 2,
              borderColor: '#FFFFFF'
            }]
          },
          options: {
            responsive: true,
            maintainAspectRatio: false,
            cutout: '62%',
            plugins: {
              legend: { display: false },
              tooltip: {
                callbacks: {
                  label: (ctx) => {
                    const val = ctx.parsed;
                    const pct = grandTotal > 0 ? ((val / grandTotal) * 100).toFixed(1) : 0;
                    return ` ${ctx.label}: ${val}分 (${pct}%)`;
                  }
                }
              }
            }
          }
        });

        legendEl.innerHTML = sortedEntries.map(([id, val]) => {
          const s = SUBJECT_MAP[id] || { name: id, color: '#64748B' };
          const pct = ((val / grandTotal) * 100).toFixed(1);
          return `
            <div class="flex items-center justify-between py-1 px-2 rounded-lg hover:bg-slate-50">
              <div class="flex items-center gap-2">
                <span class="w-2.5 h-2.5 rounded-full shrink-0" style="background-color:${s.color};"></span>
                <span class="font-semibold text-slate-700">${escapeHTML(s.name)}</span>
              </div>
              <div class="text-slate-600">
                <strong class="text-slate-900">${val}分</strong>
                <span class="text-slate-400 ml-1">(${pct}%)</span>
              </div>
            </div>
          `;
        }).join('');
      }

      // 1週間の積み上げ棒グラフ（直近7日間）
      const last7Days = [];
      const dayLabels = [];
      const weekdays = ['日', '月', '火', '水', '木', '金', '土'];
      for (let i = 6; i >= 0; i--) {
        const dObj = addDays(today, -i);
        const dStr = getLocalDateStr(dObj);
        last7Days.push(dStr);
        dayLabels.push(`${dObj.getMonth() + 1}/${dObj.getDate()}(${weekdays[dObj.getDay()]})`);
      }

      // 7日間に登場した科目のみデータセット化
      const activeSubjectIds = new Set();
      let weeklySum = 0;
      appState.logs.forEach(log => {
        if (last7Days.includes(log.date)) {
          activeSubjectIds.add(log.subjectId);
          weeklySum += Number(log.minutes);
        }
      });
      document.getElementById('weekly-total-badge').textContent = `週間合計: ${weeklySum}分`;

      const datasets = Array.from(activeSubjectIds).map(subjId => {
        const s = SUBJECT_MAP[subjId] || { name: subjId, color: '#64748B' };
        const data = last7Days.map(dStr => {
          return appState.logs
            .filter(l => l.date === dStr && l.subjectId === subjId)
            .reduce((sum, l) => sum + Number(l.minutes), 0);
        });
        return {
          label: s.name,
          data,
          backgroundColor: s.color,
          borderRadius: 4,
          maxBarThickness: 38
        };
      });

      const weeklyCtx = document.getElementById('weekly-stacked-chart').getContext('2d');
      if (weeklyChartInstance) weeklyChartInstance.destroy();

      weeklyChartInstance = new Chart(weeklyCtx, {
        type: 'bar',
        data: {
          labels: dayLabels,
          datasets: datasets.length > 0 ? datasets : [{ label: '学習なし', data: [0,0,0,0,0,0,0], backgroundColor: '#E2E8F0' }]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          scales: {
            x: {
              stacked: true,
              grid: { display: false }
            },
            y: {
              stacked: true,
              beginAtZero: true,
              title: { display: true, text: '勉強時間（分）', font: { size: 11 } },
              grid: { color: '#F1F5F9' }
            }
          },
          plugins: {
            legend: {
              position: 'bottom',
              labels: { boxWidth: 12, usePointStyle: true, font: { size: 11 } }
            },
            tooltip: {
              callbacks: {
                label: (ctx) => ` ${ctx.dataset.label}: ${ctx.parsed.y}分`
              }
            }
          }
        }
      });
    }

    /**
     * 11. 過去問記録レンダリング
     */
    function renderPastExams() {
      const tbody = document.getElementById('past-exam-tbody');
      if (appState.pastExams.length === 0) {
        tbody.innerHTML = `<tr><td colspan="6" class="py-8 text-center text-xs text-slate-400">まだ過去問記録がありません。上のフォームから追加してください。</td></tr>`;
        return;
      }

      tbody.innerHTML = appState.pastExams.map(item => {
        const acc = Number(item.accuracy);
        const accBadgeClass = acc >= 80
          ? 'bg-emerald-50 text-emerald-700 border-emerald-200'
          : acc >= 60
            ? 'bg-indigo-50 text-indigo-700 border-indigo-200'
            : 'bg-amber-50 text-amber-700 border-amber-200';

        return `
          <tr class="hover:bg-slate-50/80 transition-colors">
            <td class="py-3 px-4 font-bold text-slate-800 whitespace-nowrap">${escapeHTML(item.year)}</td>
            <td class="py-3 px-4 font-semibold text-slate-900">${escapeHTML(item.univ)}</td>
            <td class="py-3 px-4 whitespace-nowrap">${renderSubjectBadge(item.subjectId)}</td>
            <td class="py-3 px-4 font-semibold text-slate-700 whitespace-nowrap">${item.minutes} 分</td>
            <td class="py-3 px-4 whitespace-nowrap">
              <span class="inline-block px-2.5 py-0.5 rounded-full text-xs font-bold border ${accBadgeClass}">
                ${acc}%
              </span>
            </td>
            <td class="py-3 px-4 text-right whitespace-nowrap">
              <button type="button" onclick="deletePastExam('${item.id}')" class="text-slate-400 hover:text-rose-600 p-1.5 rounded-lg hover:bg-rose-50 transition-colors" title="削除">
                <i class="fa-solid fa-trash-can text-xs"></i>
              </button>
            </td>
          </tr>
        `;
      }).join('');
    }

    function deletePastExam(id) {
      appState.pastExams = appState.pastExams.filter(p => p.id !== id);
      saveState();
      renderPastExams();
      showToast('過去問記録を削除しました', 'info');
    }

    /**
     * 12. 教材登録（カメラ撮影・画像リサイズ圧縮） & 本棚 & 学習ログテーブル
     */
    let activeCameraStream = null;
    let capturedCoverBase64 = '';

    function compressImageToDataURL(sourceElementOrImg, maxWidth = 360) {
      const canvas = document.getElementById('camera-capture-canvas');
      const w = sourceElementOrImg.videoWidth || sourceElementOrImg.naturalWidth || 300;
      const h = sourceElementOrImg.videoHeight || sourceElementOrImg.naturalHeight || 400;
      const scale = Math.min(1, maxWidth / w);
      canvas.width = Math.round(w * scale);
      canvas.height = Math.round(h * scale);
      const ctx = canvas.getContext('2d');
      ctx.drawImage(sourceElementOrImg, 0, 0, canvas.width, canvas.height);
      return canvas.toDataURL('image/jpeg', 0.78);
    }

    function stopCameraStream() {
      if (activeCameraStream) {
        activeCameraStream.getTracks().forEach(track => track.stop());
        activeCameraStream = null;
      }
      document.getElementById('camera-stream-wrapper').classList.add('hidden');
    }

    function setCapturedPreview(dataUrl) {
      capturedCoverBase64 = dataUrl;
      const box = document.getElementById('material-cover-preview-box');
      const img = document.getElementById('material-cover-preview-img');
      if (dataUrl) {
        img.src = dataUrl;
        box.classList.remove('hidden');
      } else {
        img.src = '';
        box.classList.add('hidden');
      }
    }

    function syncMaterialDropdowns() {
      const selects = [
        document.getElementById('timer-material-select'),
        document.getElementById('manual-log-material')
      ];
      selects.forEach(sel => {
        if (!sel) return;
        const prev = sel.value;
        sel.innerHTML = `<option value="">（教材指定なし / 自由演習）</option>`;
        appState.materials.forEach(mat => {
          const subjName = SUBJECT_MAP[mat.subjectId]?.name || '';
          const opt = document.createElement('option');
          opt.value = mat.id;
          opt.textContent = `【${subjName}】${mat.name}`;
          sel.appendChild(opt);
        });
        if (prev) sel.value = prev;
      });
    }

    function renderBookshelf() {
      const filterSubj = document.getElementById('bookshelf-filter-subject').value;
      const grid = document.getElementById('bookshelf-grid');

      const filtered = appState.materials.filter(m => filterSubj === 'ALL' || m.subjectId === filterSubj);
      if (filtered.length === 0) {
        grid.innerHTML = `<div class="col-span-full py-10 text-center text-xs text-slate-400">登録されている教材がありません。左のフォームから登録してください。</div>`;
        return;
      }

      grid.innerHTML = filtered.map(mat => {
        const s = SUBJECT_MAP[mat.subjectId] || { name: '未分類', color: '#64748B', bgSoft: '#F1F5F9', textDark: '#334155' };
        const totalMin = appState.logs
          .filter(l => l.materialId === mat.id)
          .reduce((sum, l) => sum + Number(l.minutes), 0);

        const coverHtml = mat.coverImage
          ? `<img src="${mat.coverImage}" alt="${escapeHTML(mat.name)}" class="w-16 h-22 object-cover rounded-lg border border-slate-200 shrink-0" />`
          : `<div class="w-16 h-22 rounded-lg shrink-0 flex flex-col items-center justify-center text-white p-1.5 text-center shadow-xs" style="background-color:${s.color};">
               <i class="fa-solid fa-book text-base mb-1 opacity-90"></i>
               <span class="text-[9px] font-bold leading-tight line-clamp-2">${escapeHTML(s.name)}</span>
             </div>`;

        return `
          <div class="border border-slate-200 rounded-2xl p-3.5 bg-slate-50/50 hover:bg-white transition-all flex gap-3 justify-between">
            <div class="flex gap-3 min-w-0">
              ${coverHtml}
              <div class="min-w-0 flex flex-col justify-between">
                <div>
                  ${renderSubjectBadge(mat.subjectId)}
                  <h4 class="text-sm font-bold text-slate-900 mt-1.5 truncate" title="${escapeHTML(mat.name)}">${escapeHTML(mat.name)}</h4>
                  ${mat.memo ? `<p class="text-[11px] text-slate-500 truncate mt-0.5">${escapeHTML(mat.memo)}</p>` : ''}
                </div>
                <p class="text-xs font-semibold text-indigo-600 mt-2">
                  <i class="fa-regular fa-clock mr-1"></i>累計 ${totalMin}分
                </p>
              </div>
            </div>
            <button type="button" onclick="deleteMaterial('${mat.id}')" class="text-slate-400 hover:text-rose-600 self-start p-1 rounded-lg hover:bg-rose-50 transition-colors" title="教材を削除">
              <i class="fa-solid fa-trash-can text-xs"></i>
            </button>
          </div>
        `;
      }).join('');
    }

    function deleteMaterial(id) {
      appState.materials = appState.materials.filter(m => m.id !== id);
      saveState();
      syncMaterialDropdowns();
      renderBookshelf();
      renderStudyLogsTable();
      showToast('教材を本棚から削除しました', 'info');
    }

    function renderStudyLogsTable() {
      const filterSubj = document.getElementById('log-filter-subject').value;
      const tbody = document.getElementById('study-log-tbody');

      const sortedLogs = [...appState.logs]
        .filter(l => filterSubj === 'ALL' || l.subjectId === filterSubj)
        .sort((a, b) => b.date.localeCompare(a.date));

      if (sortedLogs.length === 0) {
        tbody.innerHTML = `<tr><td colspan="5" class="py-8 text-center text-xs text-slate-400">該当する学習ログがありません。</td></tr>`;
        return;
      }

      tbody.innerHTML = sortedLogs.map(log => {
        const mat = appState.materials.find(m => m.id === log.materialId);
        return `
          <tr class="hover:bg-slate-50/80 transition-colors">
            <td class="py-3 px-4 font-semibold text-slate-700 whitespace-nowrap">${escapeHTML(log.date)}</td>
            <td class="py-3 px-4 whitespace-nowrap">${renderSubjectBadge(log.subjectId)}</td>
            <td class="py-3 px-4 font-medium text-slate-800">
              ${mat
                ? `<span class="inline-flex items-center gap-1.5"><i class="fa-solid fa-book text-slate-400 text-xs"></i>${escapeHTML(mat.name)}</span>`
                : `<span class="text-slate-400 text-xs">（教材指定なし）</span>`}
            </td>
            <td class="py-3 px-4 font-bold text-indigo-700 whitespace-nowrap">${log.minutes} 分</td>
            <td class="py-3 px-4 text-right whitespace-nowrap">
              <button type="button" onclick="deleteStudyLog('${log.id}')" class="text-slate-400 hover:text-rose-600 p-1.5 rounded-lg hover:bg-rose-50 transition-colors" title="ログを削除">
                <i class="fa-solid fa-trash-can text-xs"></i>
              </button>
            </td>
          </tr>
        `;
      }).join('');
    }

    function deleteStudyLog(id) {
      appState.logs = appState.logs.filter(l => l.id !== id);
      saveState();
      refreshAllUI();
      showToast('学習ログを削除しました', 'info');
    }

    function openManualLogModal() {
      document.getElementById('manual-log-date').value = getLocalDateStr();
      syncMaterialDropdowns();
      document.getElementById('manual-log-modal').classList.remove('hidden');
    }

    /**
     * 13. 全体UIの再同期
     */
    function refreshAllUI() {
      renderGoals();
      syncMaterialDropdowns();
      updateTimerUI();
      renderCalendar();
      if (currentTab === 'analytics') {
        renderCharts();
        renderPastExams();
      }
      if (currentTab === 'materials') {
        renderBookshelf();
        renderStudyLogsTable();
      }
    }

    /**
     * 14. イベントリスナー登録（DOMContentLoaded）
     */
    document.addEventListener('DOMContentLoaded', () => {
      populateSubjectSelects();
      loadState();
      refreshAllUI();

      // ナビゲーションタブ切替
      document.querySelectorAll('.nav-tab-btn').forEach(btn => {
        btn.addEventListener('click', () => switchTab(btn.dataset.tab));
      });

      // タイマーモード切替
      const countdownBtn = document.getElementById('mode-countdown-btn');
      const countupBtn = document.getElementById('mode-countup-btn');
      const setupPanel = document.getElementById('countdown-setup-panel');

      countdownBtn.addEventListener('click', () => {
        if (timerState.isRunning) return;
        timerState.mode = 'countdown';
        countdownBtn.className = 'py-2 px-3 rounded-lg text-xs sm:text-sm font-semibold bg-white text-indigo-600 shadow-xs transition-all';
        countupBtn.className = 'py-2 px-3 rounded-lg text-xs sm:text-sm font-semibold text-slate-600 hover:text-slate-900 transition-all';
        setupPanel.classList.remove('hidden');
        resetTimer();
      });

      countupBtn.addEventListener('click', () => {
        if (timerState.isRunning) return;
        timerState.mode = 'countup';
        countupBtn.className = 'py-2 px-3 rounded-lg text-xs sm:text-sm font-semibold bg-white text-indigo-600 shadow-xs transition-all';
        countdownBtn.className = 'py-2 px-3 rounded-lg text-xs sm:text-sm font-semibold text-slate-600 hover:text-slate-900 transition-all';
        setupPanel.classList.add('hidden');
        resetTimer();
      });

      // カウントダウン1分単位調整
      const minInput = document.getElementById('timer-minutes-input');
      document.getElementById('timer-minus-5').addEventListener('click', () => setTimerMinutes(Number(minInput.value) - 5));
      document.getElementById('timer-minus-1').addEventListener('click', () => setTimerMinutes(Number(minInput.value) - 1));
      document.getElementById('timer-plus-1').addEventListener('click', () => setTimerMinutes(Number(minInput.value) + 1));
      document.getElementById('timer-plus-5').addEventListener('click', () => setTimerMinutes(Number(minInput.value) + 5));
      minInput.addEventListener('change', () => setTimerMinutes(minInput.value));

      // タイマー操作
      document.getElementById('timer-start-pause-btn').addEventListener('click', toggleTimer);
      document.getElementById('timer-reset-btn').addEventListener('click', resetTimer);
      document.getElementById('timer-save-log-btn').addEventListener('click', recordTimerSession);
      document.getElementById('timer-subject-select').addEventListener('change', updateTimerUI);

      // カレンダー月移動
      document.getElementById('cal-prev-month').addEventListener('click', () => {
        calendarViewDate = new Date(calendarViewDate.getFullYear(), calendarViewDate.getMonth() - 1, 1);
        renderCalendar();
      });
      document.getElementById('cal-next-month').addEventListener('click', () => {
        calendarViewDate = new Date(calendarViewDate.getFullYear(), calendarViewDate.getMonth() + 1, 1);
        renderCalendar();
      });
      document.getElementById('cal-today-btn').addEventListener('click', () => {
        calendarViewDate = new Date();
        selectedCalendarDateStr = getLocalDateStr();
        renderCalendar();
      });

      // 目標設定フォーム保存
      document.getElementById('goal-modal-form').addEventListener('submit', (e) => {
        e.preventDefault();
        const type = document.getElementById('goal-modal-type').value;
        if (type === 'daily') {
          appState.goals.dailyMinutes = Math.max(10, Number(document.getElementById('input-daily-minutes').value) || 180);
        } else if (type === 'milestone') {
          appState.goals.milestone = {
            title: document.getElementById('input-event-title').value.trim() || 'マイルストーン目標',
            date: document.getElementById('input-event-date').value
          };
        } else if (type === 'goal') {
          appState.goals.goal = {
            title: document.getElementById('input-event-title').value.trim() || '最終ゴール目標',
            date: document.getElementById('input-event-date').value
          };
        }
        saveState();
        renderGoals();
        renderCalendar();
        closeModal('goal-modal');
        showToast('目標設定を保存しました', 'success');
      });

      // アラーム設定モーダル
      document.getElementById('open-alarm-modal-btn').addEventListener('click', () => {
        document.getElementById('alarm-enabled-checkbox').checked = appState.alarm.enabled;
        document.getElementById('alarm-sound-select').value = appState.alarm.sound;
        document.getElementById('alarm-volume-range').value = appState.alarm.volume;
        document.getElementById('alarm-volume-label').textContent = `${appState.alarm.volume}%`;
        document.getElementById('alarm-modal').classList.remove('hidden');
      });
      document.getElementById('alarm-volume-range').addEventListener('input', (e) => {
        document.getElementById('alarm-volume-label').textContent = `${e.target.value}%`;
      });
      document.getElementById('test-alarm-sound-btn').addEventListener('click', () => {
        const sound = document.getElementById('alarm-sound-select').value;
        const vol = Number(document.getElementById('alarm-volume-range').value);
        playAlarmSound(sound, vol);
      });
      document.getElementById('save-alarm-settings-btn').addEventListener('click', () => {
        appState.alarm = {
          enabled: document.getElementById('alarm-enabled-checkbox').checked,
          sound: document.getElementById('alarm-sound-select').value,
          volume: Number(document.getElementById('alarm-volume-range').value)
        };
        saveState();
        closeModal('alarm-modal');
        showToast('アラーム設定を保存しました', 'success');
      });

      // 分析ページ：期間切替 & 手入力モーダル
      document.getElementById('chart-period-select').addEventListener('change', renderCharts);
      document.getElementById('open-manual-log-modal-btn').addEventListener('click', openManualLogModal);

      document.getElementById('manual-log-form').addEventListener('submit', (e) => {
        e.preventDefault();
        const date = document.getElementById('manual-log-date').value || getLocalDateStr();
        const subjectId = document.getElementById('manual-log-subject').value;
        const materialId = document.getElementById('manual-log-material').value;
        const minutes = Math.max(1, Number(document.getElementById('manual-log-minutes').value) || 30);

        appState.logs.unshift({
          id: 'log_' + Date.now(),
          date,
          subjectId,
          materialId,
          minutes
        });
        saveState();
        closeModal('manual-log-modal');
        refreshAllUI();
        showToast(`${date} の学習記録（${minutes}分）を追加しました`, 'success');
      });

      // 過去問登録フォーム
      document.getElementById('past-exam-form').addEventListener('submit', (e) => {
        e.preventDefault();
        const year = document.getElementById('exam-year').value.trim();
        const univ = document.getElementById('exam-univ').value.trim();
        const subjectId = document.getElementById('exam-subject').value;
        const minutes = Number(document.getElementById('exam-minutes').value);
        const accuracy = Number(document.getElementById('exam-accuracy').value);

        appState.pastExams.unshift({
          id: 'pe_' + Date.now(),
          year,
          univ,
          subjectId,
          minutes,
          accuracy
        });
        saveState();
        e.target.reset();
        renderPastExams();
        showToast('過去問演習の記録を追加しました', 'success');
      });

      // カメラ読み取り & 画像アップロード（教材登録）
      document.getElementById('start-camera-btn').addEventListener('click', async () => {
        try {
          if (!navigator.mediaDevices || !navigator.mediaDevices.getUserMedia) {
            showToast('このブラウザではカメラ起動に対応していません。「画像を選択」をご利用ください。', 'error');
            return;
          }
          activeCameraStream = await navigator.mediaDevices.getUserMedia({
            video: { facingMode: 'environment' },
            audio: false
          });
          const video = document.getElementById('camera-video');
          video.srcObject = activeCameraStream;
          document.getElementById('camera-stream-wrapper').classList.remove('hidden');
        } catch (err) {
          showToast('カメラへのアクセスが許可されませんでした。画像選択ボタンからも登録できます。', 'error');
        }
      });

      document.getElementById('capture-photo-btn').addEventListener('click', () => {
        const video = document.getElementById('camera-video');
        const compressed = compressImageToDataURL(video, 320);
        setCapturedPreview(compressed);
        stopCameraStream();
        showToast('表紙を撮影しました！教材名を入力して登録してください', 'success');
      });

      document.getElementById('close-camera-btn').addEventListener('click', stopCameraStream);
      document.getElementById('clear-cover-img-btn').addEventListener('click', () => setCapturedPreview(''));

      document.getElementById('material-image-file').addEventListener('change', (e) => {
        const file = e.target.files[0];
        if (!file) return;
        const reader = new FileReader();
        reader.onload = (ev) => {
          const img = new Image();
          img.onload = () => {
            const compressed = compressImageToDataURL(img, 320);
            setCapturedPreview(compressed);
          };
          img.src = ev.target.result;
        };
        reader.readAsDataURL(file);
      });

      // 教材登録フォーム送信
      document.getElementById('material-register-form').addEventListener('submit', (e) => {
        e.preventDefault();
        const name = document.getElementById('material-name-input').value.trim();
        const subjectId = document.getElementById('material-subject-select').value;
        const memo = document.getElementById('material-memo-input').value.trim();

        appState.materials.unshift({
          id: 'mat_' + Date.now(),
          name,
          subjectId,
          memo,
          coverImage: capturedCoverBase64
        });

        saveState();
        e.target.reset();
        setCapturedPreview('');
        syncMaterialDropdowns();
        renderBookshelf();
        showToast(`教材「${name}」を本棚に登録しました`, 'success');
      });

      // 本棚・学習ログのフィルタ切替
      document.getElementById('bookshelf-filter-subject').addEventListener('change', renderBookshelf);
      document.getElementById('log-filter-subject').addEventListener('change', renderStudyLogsTable);

      // データ管理モーダル
      document.getElementById('open-data-modal-btn').addEventListener('click', () => {
        document.getElementById('data-modal').classList.remove('hidden');
      });
      document.getElementById('load-sample-data-btn').addEventListener('click', () => {
        seedSampleData(true);
        closeModal('data-modal');
      });
      document.getElementById('reset-all-data-btn').addEventListener('click', () => {
        appState.materials = [];
        appState.logs = [];
        appState.pastExams = [];
        saveState();
        refreshAllUI();
        closeModal('data-modal');
        showToast('すべての学習データ・教材・過去問記録を初期化しました', 'info');
      });
    });
  </script>
</body>
</html>
```

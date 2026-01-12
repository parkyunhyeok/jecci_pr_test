<html lang="ko" translate="no">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <meta name="google" content="notranslate">
  <title>JCCEI 보도자료 캘린더 MVP</title>

  <!-- Excel(.xlsx) 생성용 (SheetJS CDN) -->
  <script src="https://cdn.jsdelivr.net/npm/xlsx@0.19.3/dist/xlsx.full.min.js"></script>

  <style>
    :root{
      --bg:#f6f7fb;
      --card:#ffffff;
      --text:#0f172a;
      --muted:#64748b;
      --line:#e2e8f0;
      --ok:#16a34a;
      --bad:#dc2626;
      --accent:#2563eb;
      --shadow: 0 10px 25px rgba(2,6,23,.06);
      --radius: 14px;
    }
    *{box-sizing:border-box}
    body{
      margin:0;
      font-family: system-ui, -apple-system, "Segoe UI", Roboto, "Noto Sans KR", sans-serif;
      background:var(--bg);
      color:var(--text);
    }
    .wrap{max-width:1200px; margin:0 auto; padding:18px;}
    header{
      display:flex; flex-wrap:wrap; gap:12px; align-items:flex-start; justify-content:space-between;
      margin-bottom:14px;
    }
    .title{display:flex; flex-direction:column; gap:6px; min-width:280px; flex:1;}
    .title h1{margin:0; font-size:20px;}
    .title p{margin:0; color:var(--muted); font-size:13px;}
    .bar{display:flex; gap:8px; flex-wrap:wrap; align-items:center; justify-content:flex-end;}
    .btn{
      border:1px solid var(--line);
      background:var(--card);
      padding:10px 12px;
      border-radius:12px;
      cursor:pointer;
      font-weight:950;
      box-shadow: 0 2px 8px rgba(2,6,23,.04);
      white-space:nowrap;
    }
    .btn.primary{background:var(--accent); color:white; border-color:transparent;}
    .btn.danger{background:var(--bad); color:white; border-color:transparent;}
    .btn.ghost{background:transparent; box-shadow:none;}
    .btn.small{padding:8px 10px; font-size:13px;}
    .btn:active{transform:translateY(1px)}
    .grid{
      display:grid;
      grid-template-columns: 1.15fr .85fr;
      gap:14px;
    }
    @media (max-width: 980px){ .grid{grid-template-columns:1fr} }

    .card{
      background:var(--card);
      border:1px solid var(--line);
      border-radius: var(--radius);
      padding:14px;
      box-shadow: var(--shadow);
      min-width:0;
    }
    .card h2{margin:0 0 10px; font-size:16px;}
    .row{display:flex; gap:10px; flex-wrap:wrap; align-items:center; min-width:0;}
    .pill{
      font-size:12px; color:#0f172a; background:#f1f5f9; border:1px solid var(--line);
      border-radius:999px; padding:4px 8px; display:inline-flex; gap:6px; align-items:center;
      font-weight:900;
      max-width:100%;
      overflow:hidden;
      text-overflow:ellipsis;
      white-space:nowrap;
    }
    .muted{color:var(--muted); font-size:13px;}
    .small{font-size:12px; color:var(--muted);}
    .divider{height:1px; background:var(--line); margin:12px 0;}

    input, textarea, select{
      width:100%;
      padding:10px 12px;
      border-radius:12px;
      border:1px solid var(--line);
      background:white;
      font-size:14px;
      outline:none;
      min-width:0;
    }
    textarea{min-height:160px; resize:vertical;}
    label{display:grid; gap:6px; font-size:13px; color:var(--muted); min-width:0;}
    .two{display:grid; grid-template-columns:1fr 1fr; gap:10px;}
    @media (max-width: 680px){ .two{grid-template-columns:1fr} }

    .list{display:grid; gap:10px; min-width:0;}
    .item{
      border:1px solid var(--line);
      border-radius:14px;
      padding:12px;
      background:#fff;
      min-width:0;
    }
    .item .top{display:flex; justify-content:space-between; gap:10px; align-items:flex-start; min-width:0;}
    .item .t{font-weight:950; min-width:0; overflow-wrap:anywhere;}

    .calendar{
      display:grid;
      grid-template-columns: repeat(7, 1fr);
      gap:8px;
      user-select:none;
      min-width:0;
    }
    .dow{
      font-size:12px; color:var(--muted); text-align:center; padding:6px 0;
      font-weight:900;
    }
    .day{
      border:1px solid var(--line);
      border-radius:14px;
      padding:10px;
      background:#fff;
      position:relative;
      min-height:84px;
      overflow:hidden;
    }
    .day.out{background:#f8fafc; color:#94a3b8;}
    .day .n{font-weight:950; font-size:13px;}

    @media (max-width: 520px){
      .wrap{padding:12px;}
      .day{min-height:72px; padding:8px;}
      .day .n{font-size:12px;}
      .badge{top:8px; right:8px; font-size:10px; padding:3px 7px;}
      .dow{font-size:11px;}
    }

    .badge{
      position:absolute; top:10px; right:10px;
      font-size:10.5px; font-weight:950;
      padding:3px 8px; border-radius:999px;
      border:1px solid var(--line);
      background:#f8fafc;
      cursor:pointer;
      line-height:1.1;
      white-space:nowrap;
      user-select:none;
      max-width: calc(100% - 14px);
      overflow:hidden;
      text-overflow:ellipsis;
    }
    .badge.ok{color:var(--ok); background:#ecfdf5; border-color:#bbf7d0;}
    .badge.bad{color:var(--bad); background:#fef2f2; border-color:#fecaca;}
    .badge.approved{color:#0b3c8a; background:#eff6ff; border-color:#dbeafe;}

    .badge.disabled{
      opacity:.55;
      cursor:not-allowed;
      filter:grayscale(.2);
    }

    @keyframes flashGreen {
      0%{ box-shadow: 0 0 0 0 rgba(22,163,74,.45); transform:translateY(0); }
      60%{ box-shadow: 0 0 0 10px rgba(22,163,74,0); transform:translateY(-1px); }
      100%{ box-shadow: 0 0 0 0 rgba(22,163,74,0); transform:translateY(0); }
    }
    @keyframes flashRed {
      0%{ box-shadow: 0 0 0 0 rgba(220,38,38,.45); transform:translateY(0); }
      60%{ box-shadow: 0 0 0 10px rgba(220,38,38,0); transform:translateY(-1px); }
      100%{ box-shadow: 0 0 0 0 rgba(220,38,38,0); transform:translateY(0); }
    }
    .flash-green{ animation: flashGreen .55s ease-out; }
    .flash-red{ animation: flashRed .55s ease-out; }

    .tabs{display:flex; gap:8px; flex-wrap:wrap;}
    .tab{
      padding:8px 12px; border-radius:999px;
      border:1px solid var(--line);
      background:#fff; cursor:pointer; font-weight:950; font-size:13px;
      white-space:nowrap;
    }
    .tab.active{background:var(--accent); color:white; border-color:transparent;}
    .hidden{display:none;}

    .note{
      padding:10px 12px; border:1px dashed #cbd5e1; border-radius:14px; background:#f8fafc;
      font-size:13px; color:#334155;
      overflow-wrap:anywhere;
    }
    .helpBox{
      padding:12px;
      border:1px solid var(--line);
      border-radius:14px;
      background:#fff;
    }
    .helpBox details > summary{list-style:none;}
    .helpBox details > summary::-webkit-details-marker{display:none;}
    .helpBox summary{
      display:flex; align-items:center; justify-content:space-between; gap:10px;
      cursor:pointer; font-weight:950;
    }
    .helpBox .helpBody{margin-top:10px; color:#334155; font-size:13px; line-height:1.6;}
    .helpBox ul{margin:8px 0 0 18px; padding:0;}
    .helpBox li{margin:4px 0;}

    table{
      width:100%;
      border-collapse:separate;
      border-spacing:0;
      overflow:hidden;
      border:1px solid var(--line);
      border-radius:14px;
      background:#fff;
    }
    th, td{
      padding:10px 10px;
      border-bottom:1px solid var(--line);
      font-size:13px;
      vertical-align:top;
    }
    th{
      background:#f8fafc;
      color:#334155;
      font-weight:950;
      text-align:left;
    }
    tr:last-child td{border-bottom:none;}
    .kstatus{
      display:inline-flex;
      padding:4px 8px;
      border-radius:999px;
      font-weight:950;
      font-size:12px;
      border:1px solid var(--line);
      background:#f8fafc;
      white-space:nowrap;
    }
    .kstatus.pending{color:#0b3c8a; background:#eff6ff; border-color:#dbeafe;}
    .kstatus.approved{color:var(--ok); background:#ecfdf5; border-color:#bbf7d0;}
    .kstatus.rejected{color:var(--bad); background:#fef2f2; border-color:#fecaca;}
    .mono{font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;}

    .imgRow{display:flex; gap:10px; flex-wrap:wrap; margin-top:8px;}
    .thumbWrap{ width:92px; display:flex; flex-direction:column; gap:6px; align-items:center; }
    .thumbBox{
      position:relative;
      width:92px; height:92px;
      border:1px solid var(--line);
      border-radius:14px;
      background:#fff;
      overflow:hidden;
    }
    .thumb{ width:100%; height:100%; object-fit:cover; display:block; }
    .xbtn{
      position:absolute; top:6px; right:6px;
      width:24px; height:24px;
      border-radius:999px;
      border:1px solid rgba(15,23,42,.15);
      background:rgba(255,255,255,.92);
      cursor:pointer;
      font-weight:950;
      display:flex; align-items:center; justify-content:center;
      line-height:1; padding:0;
    }
    .xbtn:active{transform:translateY(1px)}
    .fname{
      max-width:92px;
      font-size:11px;
      color:var(--muted);
      text-align:center;
      overflow:hidden;
      text-overflow:ellipsis;
      white-space:nowrap;
    }

    .toast{
      position:fixed;
      left:50%;
      bottom:22px;
      transform:translateX(-50%);
      background:#0f172a;
      color:white;
      border:1px solid rgba(255,255,255,.12);
      padding:10px 12px;
      border-radius:14px;
      box-shadow: 0 12px 30px rgba(0,0,0,.18);
      max-width:min(720px, calc(100vw - 24px));
      font-size:13px;
      line-height:1.35;
      opacity:0;
      pointer-events:none;
      transition: opacity .18s ease, transform .18s ease;
      z-index:9999;
      white-space:pre-wrap;
    }
    .toast.show{
      opacity:1;
      transform:translateX(-50%) translateY(-2px);
    }

    .exportBar{
      display:flex; gap:8px; flex-wrap:wrap; align-items:flex-end;
      margin-top:8px;
    }
    .exportBar label{min-width:160px}
    .exportBar .btn{white-space:nowrap}

    .searchBar{
      display:flex;
      gap:10px;
      flex-wrap:wrap;
      align-items:flex-end;
      margin:8px 0 10px;
    }
    .searchBar label{min-width:240px; flex:1;}

    details > summary{list-style:none;}
    details > summary::-webkit-details-marker{display:none;}
    .summaryBtn{
      display:inline-flex; align-items:center; gap:6px;
      padding:6px 10px;
      border-radius:999px;
      border:1px solid var(--line);
      background:#fff;
      cursor:pointer;
      font-weight:950;
      font-size:12px;
      color:#0f172a;
    }

    dialog{
      border:none;
      border-radius:16px;
      padding:0;
      width:min(920px, calc(100vw - 24px));
      box-shadow: 0 30px 80px rgba(0,0,0,.25);
    }
    dialog::backdrop{background:rgba(2,6,23,.55)}
    .modalHead{
      padding:14px 14px 10px;
      border-bottom:1px solid var(--line);
      display:flex; justify-content:space-between; align-items:center; gap:10px;
      background:#fff;
    }
    .modalBody{padding:14px; background:#fff;}
    .modalFoot{
      padding:12px 14px;
      border-top:1px solid var(--line);
      background:#fff;
      display:flex; gap:8px; justify-content:flex-end; flex-wrap:wrap;
    }
    .modalTitle{font-weight:950;}
    .tag{
      display:inline-flex; align-items:center; gap:6px;
      padding:4px 8px; border-radius:999px;
      font-size:12px; font-weight:950;
      border:1px solid var(--line);
      background:#f8fafc;
    }
    .tag.edited{background:#fff7ed; border-color:#fed7aa; color:#9a3412;}
    .diff-red{color:var(--bad); font-weight:950; background:#fee2e2; padding:0 2px; border-radius:4px;}
    .diff-del{color:var(--bad); font-weight:950; text-decoration:line-through; background:#fee2e2; padding:0 2px; border-radius:4px;}

    .diffBox{
      border:1px solid var(--line);
      border-radius:12px;
      background:#f8fafc;
      padding:10px;
      white-space:pre-wrap;
      font-size:12px;
      color:#0f172a;
    }
  </style>
</head>

<body>
<div class="wrap">
  <header>
    <div class="title">
      <h1>JCCEI 보도자료 캘린더 MVP</h1>
      <p>정적사이트 프로토타입 · 주말/공휴일/1일1개 승인 규칙 반영</p>

      <div class="exportBar">
        <label>
          엑셀 기간 시작
          <input id="exportFrom" type="date">
        </label>
        <label>
          엑셀 기간 종료
          <input id="exportTo" type="date">
        </label>
        <button class="btn primary" id="btnExportXlsx">엑셀 내려받기</button>
        <span class="small">※ 기간 내 “신청/배포(승인)”된 보도자료 목록을 내려받습니다.</span>
      </div>
    </div>
    <div class="bar"></div>
  </header>

  <!-- ✅ 안내문구: 한 곳에 모아 노출 -->
  <div class="card">
    <div class="row" style="justify-content:space-between;">
      <div class="tabs">
        <button class="tab active" data-view="staff" id="tabStaff">신청</button>
        <button class="tab" data-view="admin" id="tabAdmin">승인</button>
        <button class="tab" data-view="settings" id="tabSettings">설정</button>
      </div>
      <div class="row">
        <span class="pill">관리자 패스코드: <span class="mono" id="adminCodeHint"></span></span>
      </div>
    </div>

    <div class="divider"></div>

    <div class="helpBox" id="helpBox">
      <details open>
        <summary>
          <span>📌 사용 안내 / 규칙</span>
          <span class="pill">펼치기/접기</span>
        </summary>
        <div class="helpBody" id="helpBody"></div>
      </details>
    </div>
  </div>

  <div class="grid" style="margin-top:14px;">
    <!-- Left: Calendar -->
    <div class="card">
      <div class="row" style="justify-content:space-between;">
        <h2 style="margin:0;">배포 캘린더</h2>
        <div class="row">
          <button class="btn ghost" id="prevMonth">←</button>
          <div class="pill" id="monthLabel"></div>
          <button class="btn ghost" id="nextMonth">→</button>
        </div>
      </div>

      <div class="divider"></div>
      <div class="calendar" id="dowRow"></div>
      <div class="calendar" id="cal"></div>

      <div class="divider"></div>
      <h2 style="margin:0 0 8px;">해당 날짜 승인 보도자료(참고)</h2>
      <div class="list" id="approvedTitles">
        <div class="muted">아직 선택된 날짜가 없습니다.</div>
      </div>
    </div>

    <!-- Right: Views -->
    <div class="card">
      <!-- 신청 -->
      <div id="view_staff">
        <h2>보도자료 신청</h2>

        <div class="two">
          <label>
            내 이름(필수)
            <input id="staffName" placeholder="예: 박윤혁" />
          </label>
          <label>
            내 연락처(필수)
            <input id="staffPhone" placeholder="예: 010-1234-5678" required />
          </label>
        </div>

        <div class="two" style="margin-top:10px;">
          <label>
            이메일(필수)
            <input id="staffEmail" type="email" placeholder="예: example@jccei.kr" required />
          </label>
          <label>
            승인 관리자(필수)
            <select id="approver" required>
              <option value="" selected disabled>선택하세요</option>
              <option>이재형 본부장</option>
              <option>이경호 본부장</option>
              <option>김희정 본부장</option>
              <option>이한솔 팀장</option>
              <option>고덕훈 팀장</option>
              <option>이병선 대표</option>
            </select>
          </label>
        </div>

        <div class="divider"></div>

        <form id="formSubmit" class="list">
          <label>
            제목(필수)
            <input id="title" required placeholder="예: 제주창조경제혁신센터, ○○ 프로그램 성료" />
          </label>

          <label>
            부제목(선택)
            <input id="subtitle" placeholder="예: 도내 스타트업 20개사 참여…" />
          </label>

          <div class="row" style="justify-content:space-between; align-items:flex-end;">
            <label style="flex:1;">
              본문(필수)
              <textarea id="body" required></textarea>
            </label>
            <div style="width:180px;">
              <button class="btn small" type="button" id="btnInsertTips">작성팁 예시 넣기</button>
              <div class="small" style="margin-top:6px;">※ 클릭 시 본문에 템플릿이 자동 입력됩니다.</div>
            </div>
          </div>

          <div class="two">
            <label>
              배포 희망일(필수)
              <input id="desiredDate" type="date" required />
            </label>

            <label>
              보도용 사진 업로드(필수: 업로드 또는 링크, 여러 장 가능)
              <input id="imageFiles" type="file" accept="image/*" multiple />
              <span class="small" id="imgHelp"></span>
            </label>
          </div>

          <label>
            대용량 파일 전달 링크(Agit/드라이브 등, 사진이 없으면 필수)
            <textarea id="bigFileLinks" placeholder="예) https://drive.google.com/...&#10;예) https://agit..."></textarea>
          </label>

          <div id="previewArea" class="imgRow" aria-label="사진 미리보기" style="display:none;"></div>

          <button class="btn primary" type="submit">신청하기</button>
          <div class="note" id="staffMsg">신청 후 관리자가 승인하면 캘린더에 등록됩니다.</div>
        </form>

        <div class="divider"></div>
        <h2>내 신청 목록</h2>
        <div class="list" id="myList"></div>

        <div class="divider"></div>

        <!-- 승인 클릭 시 여기로 스크롤 -->
        <div id="boardSection"></div>

        <h2>배포 예정/대기 현황</h2>

        <div class="searchBar">
          <label>
            검색(제목/작성자/상태/날짜)
            <input id="boardSearch" placeholder="예: 1월, 박윤혁, 배포 예정, 오픈그라운드..." />
          </label>
          <button class="btn primary" id="btnSearch" type="button">검색</button>
          <button class="btn" id="btnClearSearch" type="button">초기화</button>
        </div>

        <div style="overflow:auto;">
          <table>
            <thead>
              <tr>
                <th style="min-width:90px;">상태</th>
                <th style="min-width:260px;">제목</th>
                <th style="min-width:110px;">희망일</th>
                <th style="min-width:110px;">배포일</th>
                <th style="min-width:120px;">작성자</th>
                <th style="min-width:110px;">다운로드</th>
              </tr>
            </thead>
            <tbody id="boardTableBody">
              <tr><td colspan="6" class="muted">데이터가 없습니다.</td></tr>
            </tbody>
          </table>
        </div>
      </div>

      <!-- 승인 -->
      <div id="view_admin" class="hidden">
        <h2>관리자 승인/반려</h2>

        <label>
          관리자 패스코드(필수)
          <input id="adminPass" type="password" placeholder="설정 탭에서 변경 가능" />
        </label>

        <div class="divider"></div>

        <h2>승인 대기</h2>
        <div class="list" id="pendingList"></div>

        <div class="divider"></div>

        <h2>승인 완료</h2>
        <div class="list" id="approvedList"></div>

        <div class="divider"></div>

        <h2>카카오톡 안내문(복사해서 보내기)</h2>
        <textarea id="kakaoText" placeholder="승인/반려/첨삭 저장을 하면 여기에 문구가 생성됩니다."></textarea>
        <div class="row" style="margin-top:10px;">
          <button class="btn" id="btnCopyKakao">문구 복사</button>
          <span class="small" id="copyHint"></span>
        </div>

        <div class="divider"></div>

        <h2>데이터 관리</h2>
        <button class="btn danger" id="btnResetAdmin">전체 초기화(관리자)</button>
      </div>

      <!-- 설정 -->
      <div id="view_settings" class="hidden">
        <h2>설정</h2>

        <div class="divider"></div>

        <label>
          관리자 패스코드
          <input id="setAdminCode" />
        </label>

        <label style="margin-top:10px;">
          공휴일 목록(YYYY-MM-DD, 한 줄에 하나)
          <textarea id="setHolidays" placeholder="2026-01-01&#10;2026-02-09"></textarea>
        </label>

        <div class="row" style="margin-top:10px;">
          <button class="btn primary" id="btnSaveSettings">설정 저장</button>
          <span class="small" id="settingsHint"></span>
        </div>

        <div class="divider"></div>

        <div class="note">
          이 HTML 버전은 데이터가 <b>각자 브라우저에만 저장</b>됩니다.<br/>
          “직원 모두가 같은 데이터를 공유”하려면 중앙 저장소(예: Google Sheet/Firebase)가 필요합니다.
        </div>
      </div>
    </div>
  </div>
</div>

<!-- ✅ 신청자 수정 모달 -->
<dialog id="dlgEditUser">
  <div class="modalHead">
    <div class="modalTitle">내 보도자료 수정</div>
    <button class="btn ghost" id="dlgEditUserClose">닫기</button>
  </div>
  <div class="modalBody">
    <div class="two">
      <label>제목 <input id="uEditTitle"></label>
      <label>부제목 <input id="uEditSubtitle"></label>
    </div>
    <label style="margin-top:10px;">본문 <textarea id="uEditBody"></textarea></label>
    <div class="two" style="margin-top:10px;">
      <label>배포 희망일(필수) <input id="uEditDesiredDate" type="date"></label>
      <label>대용량 링크(선택) <textarea id="uEditLinks" style="min-height:84px;"></textarea></label>
    </div>
  </div>
  <div class="modalFoot">
    <button class="btn" id="uEditCancel">취소</button>
    <button class="btn primary" id="uEditSave">저장</button>
  </div>
</dialog>

<!-- ✅ 관리자 첨삭 모달 -->
<dialog id="dlgEditAdmin">
  <div class="modalHead">
    <div class="modalTitle">관리자 첨삭/수정</div>
    <div class="row" style="gap:8px;">
      <span class="tag edited">변경내역 자동 기록</span>
      <button class="btn ghost" id="dlgEditAdminClose">닫기</button>
    </div>
  </div>
  <div class="modalBody">
    <div class="two">
      <label>제목 <input id="aEditTitle"></label>
      <label>부제목 <input id="aEditSubtitle"></label>
    </div>
    <label style="margin-top:10px;">본문 <textarea id="aEditBody"></textarea></label>

    <div class="two" style="margin-top:10px;">
      <label>희망일(선택) <input id="aEditDesiredDate" type="date"></label>
      <label>대용량 링크(선택) <textarea id="aEditLinks" style="min-height:84px;"></textarea></label>
    </div>

    <div class="divider"></div>
    <h2 style="margin:0 0 8px;">변경 내역(최근 1회)</h2>
    <div class="diffBox" id="aLastDiff">아직 변경 내역이 없습니다.</div>
  </div>
  <div class="modalFoot">
    <button class="btn" id="aEditCancel">취소</button>
    <button class="btn primary" id="aEditSave">첨삭 저장</button>
  </div>
</dialog>

<div id="toast" class="toast" role="status" aria-live="polite"></div>

<script>
/** 이미지 업로드 제한 */
const MAX_IMAGE_MB = 2;
const MAX_IMAGE_BYTES = MAX_IMAGE_MB * 1024 * 1024;
const MAX_IMAGE_COUNT = 10;

/** 저장 키 */
const LS_KEY = "JCCEI_PRESS_MVP_DATA_V7";
const LS_SETTINGS = "JCCEI_PRESS_MVP_SETTINGS_V7";

const DEFAULT_SETTINGS = {
  adminCode: "admin1234",
  holidays: ["2026-01-01","2026-02-09","2026-02-10","2026-02-11"]
};

function loadSettings(){
  try{
    const s = JSON.parse(localStorage.getItem(LS_SETTINGS) || "null");
    if(!s) return structuredClone(DEFAULT_SETTINGS);
    return {
      adminCode: s.adminCode || DEFAULT_SETTINGS.adminCode,
      holidays: Array.isArray(s.holidays) ? s.holidays : structuredClone(DEFAULT_SETTINGS.holidays)
    };
  }catch(e){
    return structuredClone(DEFAULT_SETTINGS);
  }
}
function saveSettings(settings){
  localStorage.setItem(LS_SETTINGS, JSON.stringify(settings));
}
function loadData(){
  try{
    const d = JSON.parse(localStorage.getItem(LS_KEY) || "null");
    if(!d) return { press: [] };
    if(!Array.isArray(d.press)) d.press = [];
    d.press.forEach(p=>{
      if(!Array.isArray(p.editHistory)) p.editHistory = [];
    });
    return d;
  }catch(e){
    return { press: [] };
  }
}
function saveData(data){
  localStorage.setItem(LS_KEY, JSON.stringify(data));
}

/** 날짜 유틸 */
function ymd(date){
  const y = date.getFullYear();
  const m = String(date.getMonth()+1).padStart(2,"0");
  const d = String(date.getDate()).padStart(2,"0");
  return `${y}-${m}-${d}`;
}
function parseYMD(s){
  const [y,m,d] = s.split("-").map(Number);
  return new Date(y, m-1, d);
}
function dateToYmdFromMillis(ms){
  const dt = new Date(ms);
  return ymd(dt);
}
function isWeekend(ymdStr){
  const dt = parseYMD(ymdStr);
  const day = dt.getDay();
  return day===0 || day===6;
}
function isHoliday(ymdStr, settings){
  return new Set(settings.holidays).has(ymdStr);
}

/** ✅ 영업일 계산(주말/공휴일 제외) */
function addBusinessDays(fromYmdStr, businessDays, settings){
  let dt = parseYMD(fromYmdStr);
  let added = 0;
  while(added < businessDays){
    dt.setDate(dt.getDate() + 1);
    const dstr = ymd(dt);
    if(isWeekend(dstr)) continue;
    if(isHoliday(dstr, settings)) continue;
    added++;
  }
  return ymd(dt);
}
function earliestDesiredYmd(settings){
  // 오늘 기준 3영업일 이후부터 신청 가능
  return addBusinessDays(ymd(new Date()), 3, settings);
}

/** ✅ 캘린더 비활성(과거 + 3영업일 이내) */
function todayYmd(){ return ymd(new Date()); }
function isPastDate(dstr){ return dstr < todayYmd(); }
function isBlockedByLeadTime(dstr, settings){
  const minYmd = earliestDesiredYmd(settings);
  // 오늘 포함~(minYmd 이전) 신청불가 표기
  return dstr < minYmd;
}

function validateDesiredDateBusinessRule(inputEl, ymdStr, settings){
  if(!ymdStr) return true;
  const minYmd = earliestDesiredYmd(settings);
  if(ymdStr < minYmd){
    inputEl.value = "";
    showToast(`${ymdStr} : 접수 불가\n사유: 신청일(오늘) 기준 주말/공휴일 제외 3영업일 이전에 미리 신청해야 합니다.\n(가장 빠른 가능일: ${minYmd})`);
    return false;
  }
  return true;
}
function hasApprovedOn(ymdStr, data){
  return data.press.some(p => p.status==="APPROVED" && p.approvedDate===ymdStr);
}
function checkPublishable(ymdStr, data, settings){
  if(isWeekend(ymdStr)) return {ok:false, reason:"주말은 배포 불가"};
  if(isHoliday(ymdStr, settings)) return {ok:false, reason:"공휴일은 배포 불가"};
  if(hasApprovedOn(ymdStr, data)) return {ok:false, reason:"이미 승인된 보도자료가 있는 날짜(1일 1개)"};
  return {ok:true};
}
function isDesiredDateBlockedByApproved(ymdStr, data){
  if(!ymdStr) return false;
  return hasApprovedOn(ymdStr, data);
}

/** DOM */
const el = (id)=>document.getElementById(id);

const tabs = Array.from(document.querySelectorAll(".tab"));
const viewStaff = el("view_staff");
const viewAdmin = el("view_admin");
const viewSettings = el("view_settings");

const adminCodeHint = el("adminCodeHint");
const adminPass = el("adminPass");

const helpBody = el("helpBody");

const monthLabel = el("monthLabel");
const cal = el("cal");
const dowRow = el("dowRow");
const approvedTitles = el("approvedTitles");

const staffName = el("staffName");
const staffPhone = el("staffPhone");
const staffEmail = el("staffEmail");
const approver = el("approver");

const formSubmit = el("formSubmit");
const title = el("title");
const subtitle = el("subtitle");
const body = el("body");
const desiredDate = el("desiredDate");
const imageFiles = el("imageFiles");
const bigFileLinks = el("bigFileLinks");
const previewArea = el("previewArea");
const staffMsg = el("staffMsg");
const myList = el("myList");
const boardTableBody = el("boardTableBody");
const boardSection = el("boardSection");

const boardSearch = el("boardSearch");
const btnSearch = el("btnSearch");
const btnClearSearch = el("btnClearSearch");

const btnInsertTips = el("btnInsertTips");
const imgHelp = el("imgHelp");

const pendingList = el("pendingList");
const approvedList = el("approvedList");
const kakaoText = el("kakaoText");
const btnCopyKakao = el("btnCopyKakao");
const copyHint = el("copyHint");

const setAdminCode = el("setAdminCode");
const setHolidays = el("setHolidays");
const btnSaveSettings = el("btnSaveSettings");
const settingsHint = el("settingsHint");

const btnResetAdmin = el("btnResetAdmin");

const prevMonth = el("prevMonth");
const nextMonth = el("nextMonth");

const exportFrom = el("exportFrom");
const exportTo = el("exportTo");
const btnExportXlsx = el("btnExportXlsx");

const toast = el("toast");

/** 모달 - 신청자 수정 */
const dlgEditUser = el("dlgEditUser");
const dlgEditUserClose = el("dlgEditUserClose");
const uEditTitle = el("uEditTitle");
const uEditSubtitle = el("uEditSubtitle");
const uEditBody = el("uEditBody");
const uEditDesiredDate = el("uEditDesiredDate");
const uEditLinks = el("uEditLinks");
const uEditCancel = el("uEditCancel");
const uEditSave = el("uEditSave");

/** 모달 - 관리자 첨삭 */
const dlgEditAdmin = el("dlgEditAdmin");
const dlgEditAdminClose = el("dlgEditAdminClose");
const aEditTitle = el("aEditTitle");
const aEditSubtitle = el("aEditSubtitle");
const aEditBody = el("aEditBody");
const aEditDesiredDate = el("aEditDesiredDate");
const aEditLinks = el("aEditLinks");
const aLastDiff = el("aLastDiff");
const aEditCancel = el("aEditCancel");
const aEditSave = el("aEditSave");

/** 상태 */
let settings = loadSettings();
let data = loadData();
let cursor = new Date();
let selectedFiles = [];
let editingUserId = null;
let editingAdminId = null;

/** 작성팁 */
const PRESS_TIPS_TEMPLATE =
`[작성 팁 예시] 아래 형식대로 채우면 보도자료가 빠르게 완성됩니다.

1) 한 줄 요약(리드문, 2~3문장)
- 언제/어디서/누가/무엇을 했는지 먼저 요약합니다.

2) 핵심 포인트(3개)
- 참여 규모 / 주요 내용 / 기대 효과

3) 상세 내용
- 배경 → 진행 → 성과 → 향후 계획

4) 인용문(선택)
- 기관장/담당자 멘트를 1개 넣으면 기사 완성도가 올라갑니다.

5) 문의처(필수)
- 부서/담당자/연락처/이메일

--------------------------
[아래부터 본문 작성 시작]
`;
body.placeholder = PRESS_TIPS_TEMPLATE;

/** 토스트 */
let toastTimer = null;
function showToast(message){
  toast.textContent = message;
  toast.classList.add("show");
  if(toastTimer) clearTimeout(toastTimer);
  toastTimer = setTimeout(()=> toast.classList.remove("show"), 1800);
}

/** ✅ 안내문구(한 곳) */
function renderHelp(){
  const minYmd = earliestDesiredYmd(settings);
  helpBody.innerHTML = `
    <div><b>핵심 규칙</b></div>
    <ul>
      <li><b>주말/공휴일 배포 불가</b></li>
      <li><b>승인 기준 1일 1개</b> (이미 승인된 날짜는 신청 불가)</li>
      <li><b>신청은 오늘 기준 주말/공휴일 제외 3영업일 이후</b>부터 가능 (가장 빠른 가능일: <b>${minYmd}</b>)</li>
      <li>사진은 <b>업로드</b> 또는 <b>드라이브/Agit 링크</b> 중 하나는 필수</li>
      <li>캘린더에서 <b>[가능]/[불가]/[승인]</b>을 눌러 사유/내역을 확인할 수 있어요</li>
      <li>‘배포 예정/대기 현황’은 <b>검색 버튼</b>을 눌러 조회합니다</li>
    </ul>
  `;
}

/** ✅ 승인된 보도자료를 DOC(워드 호환)로 다운로드 */
function sanitizeFilename(name){
  return (name || "press")
    .replace(/[\\/:*?"<>|]/g, "_")
    .replace(/\s+/g, " ")
    .trim()
    .slice(0, 80);
}
function nl2br(s){
  return escapeHtml(String(s ?? "")).replace(/\n/g, "<br/>");
}
function downloadPressAsDoc(id){
  const p = data.press.find(x=>x.id===id);
  if(!p){ alert("대상을 찾을 수 없습니다."); return; }
  if(p.status !== "APPROVED"){ alert("승인된 보도자료만 다운로드할 수 있습니다."); return; }

  const imgHtml = (p.images && p.images.length)
    ? `<h3>보도용 사진</h3>` + p.images.map(im=>`<div style="margin:10px 0;"><div style="font-size:12px;color:#64748b;margin-bottom:6px;">${escapeHtml(im.name||"")}</div><img src="${im.dataUrl}" style="max-width:680px;width:100%;border:1px solid #e2e8f0;border-radius:10px;"/></div>`).join("")
    : "";

  const linkHtml = (p.bigFileLinks && String(p.bigFileLinks).trim())
    ? `<h3>첨부 링크</h3><div style="font-size:14px;line-height:1.6;">${nl2br(p.bigFileLinks)}</div>`
    : "";

  const authorLine = `${escapeHtml(p.authorName||"-")}${p.authorPhone ? ` (${escapeHtml(p.authorPhone)})` : ""}`;

  // ✅ 요청사항: 상단 고정 문구 + 배포승인일(approvedDate) 표시 안함
  const html = `<!doctype html>
<html><head><meta charset="utf-8">
<title>${escapeHtml(p.title)}</title>
</head>
<body style="font-family: 'Noto Sans KR', Arial, sans-serif; line-height:1.6;">
  <div style="font-size:12px;color:#334155;margin-bottom:10px;">
    <b>발송기관</b> : 제주창조경제혁신센터<br/>
    <b>작성자</b> : ${authorLine}
  </div>
  <hr style="border:none;border-top:1px solid #e2e8f0;margin:12px 0;"/>

  <h1 style="margin:0 0 8px;">${escapeHtml(p.title)}</h1>
  ${p.subtitle ? `<h2 style="margin:0 0 14px;font-size:16px;color:#334155;">${escapeHtml(p.subtitle)}</h2>` : ""}

  <div style="font-size:12px;color:#64748b;margin-bottom:14px;">
    배포 희망일: ${escapeHtml(p.desiredDate||"-")}<br/>
    승인 관리자: ${escapeHtml(p.approver||"-")}<br/>
    이메일: ${escapeHtml(p.authorEmail||"-")}
  </div>

  <hr style="border:none;border-top:1px solid #e2e8f0;margin:14px 0;"/>
  <div style="font-size:14px;">${nl2br(p.body)}</div>
  ${imgHtml}
  ${linkHtml}
</body></html>`;

  const blob = new Blob([html], {type: "application/msword;charset=utf-8"});
  const url = URL.createObjectURL(blob);
  const a = document.createElement("a");
  a.href = url;
  a.download = sanitizeFilename(`${p.title || "보도자료"}.doc`);
  document.body.appendChild(a);
  a.click();
  a.remove();
  setTimeout(()=> URL.revokeObjectURL(url), 1000);
}

/** 상태 라벨 */
function statusKorean(status){
  if(status==="APPROVED") return {label:"배포 예정", cls:"approved"};
  if(status==="SUBMITTED") return {label:"대기중", cls:"pending"};
  if(status==="REJECTED") return {label:"반려", cls:"rejected"};
  return {label:"임시", cls:"pending"};
}

/** 탭 전환 */
function activateTab(view){
  tabs.forEach(x=>x.classList.remove("active"));
  document.querySelector(`.tab[data-view="${view}"]`)?.classList.add("active");
  viewStaff.classList.toggle("hidden", view!=="staff");
  viewAdmin.classList.toggle("hidden", view!=="admin");
  viewSettings.classList.toggle("hidden", view!=="settings");
}
tabs.forEach(t=>{
  t.addEventListener("click", ()=>{
    const v = t.getAttribute("data-view");
    activateTab(v);
  });
});

/** 힌트 */
function setHints(){
  adminCodeHint.textContent = settings.adminCode;
}
setHints();

/** 요일 */
function renderDow(){
  const dows = ["일","월","화","수","목","금","토"];
  dowRow.innerHTML = dows.map(d => `<div class="dow">${d}</div>`).join("");
}
renderDow();

/** 승인 제목 참고 */
function renderApprovedTitlesForDate(ymdStr){
  const list = data.press
    .filter(p => p.status==="APPROVED" && p.approvedDate===ymdStr)
    .sort((a,b)=> (a.approvedAt||0) - (b.approvedAt||0));

  if(list.length===0){
    approvedTitles.innerHTML = `<div class="muted">해당 날짜에 승인된 보도자료가 없습니다.</div>`;
    return;
  }
  approvedTitles.innerHTML = list.map(p=>`
    <div class="item">
      <div class="t">${escapeHtml(p.title)}</div>
      <div class="muted" style="margin-top:6px;">
        배포일: <b>${escapeHtml(p.approvedDate||"-")}</b> · 작성자: <b>${escapeHtml(p.authorName)}</b>
      </div>
    </div>
  `).join("");
}

/** 배지 반응 */
function flash(elm, color){
  elm.classList.remove("flash-green","flash-red");
  void elm.offsetWidth;
  if(color==="green") elm.classList.add("flash-green");
  if(color==="red") elm.classList.add("flash-red");
}

/** ✅ 캘린더 표기 로직(과거/3영업일 이내도 불가로 표시) */
function calendarStatusForDate(dstr){
  const approved = data.press.find(p=>p.status==="APPROVED" && p.approvedDate===dstr);
  if(approved){
    return { badgeText:"승인", badgeClass:"approved", disabled:false, reason:`승인된 보도자료가 있습니다.\n- ${approved.title}` };
  }

  // 1) 오늘 기준 지난 날짜 비활성
  if(isPastDate(dstr)){
    return { badgeText:"불가", badgeClass:"bad disabled", disabled:true, reason:"지난 날짜는 선택할 수 없습니다." };
  }

  // 2) 3영업일 리드타임 이내 캘린더에서도 신청불가 표기
  if(isBlockedByLeadTime(dstr, settings)){
    const minYmd = earliestDesiredYmd(settings);
    return { badgeText:"불가", badgeClass:"bad disabled", disabled:true, reason:`신청 불가(3영업일 규칙)\n가장 빠른 가능일: ${minYmd}` };
  }

  // 3) 일반 규칙(주말/공휴일/1일1개)
  const chk = checkPublishable(dstr, data, settings);
  if(!chk.ok){
    return { badgeText:"불가", badgeClass:"bad", disabled:false, reason:chk.reason };
  }
  return { badgeText:"가능", badgeClass:"ok", disabled:false, reason:"배포 가능합니다." };
}

/** 캘린더 */
function renderCalendar(){
  const y = cursor.getFullYear();
  const m = cursor.getMonth();
  monthLabel.textContent = `${y}년 ${m+1}월`;

  const first = new Date(y, m, 1);
  const startDow = first.getDay();
  const last = new Date(y, m+1, 0);
  const daysInMonth = last.getDate();

  const prevLast = new Date(y, m, 0);
  const prevDays = prevLast.getDate();

  const cells = [];
  for(let i=0;i<startDow;i++){
    const dayNum = prevDays - (startDow-1-i);
    const dt = new Date(y, m-1, dayNum);
    cells.push({date: dt, inMonth:false});
  }
  for(let d=1; d<=daysInMonth; d++){
    cells.push({date: new Date(y,m,d), inMonth:true});
  }
  while(cells.length < 42){
    const dt = new Date(y, m, daysInMonth + (cells.length - (startDow + daysInMonth) + 1));
    cells.push({date: dt, inMonth:false});
  }

  cal.innerHTML = "";
  cells.forEach(c=>{
    const dstr = ymd(c.date);
    const st = calendarStatusForDate(dstr);

    const out = !c.inMonth ? "out" : "";
    const dayDiv = document.createElement("div");
    dayDiv.className = `day ${out}`;
    dayDiv.innerHTML = `
      <div class="n">${c.date.getDate()}</div>
      <span class="badge ${st.badgeClass}" data-date="${dstr}" data-type="${st.badgeText}" data-reason="${escapeHtml(st.reason)}">[${st.badgeText}]</span>
    `;

    const badge = dayDiv.querySelector(".badge");

    badge.addEventListener("click", (e)=>{
      e.stopPropagation();
      const type = badge.getAttribute("data-type");
      const dateStr = badge.getAttribute("data-date");
      const reason = badge.getAttribute("data-reason") || "";

      renderApprovedTitlesForDate(dateStr);

      if(type === "가능"){
        showToast(`${dateStr} : 배포 가능합니다.`);
        flash(badge, "green");
        desiredDate.value = dateStr;
        validateDesiredDateImmediate(desiredDate, dateStr);
        return;
      }

      if(type === "불가"){
        showToast(`${dateStr} : 배포 불가\n사유: ${unescapeHtml(reason)}`);
        flash(badge, "red");
        // ✅ 불가(과거/3영업일/주말/공휴일/1일1개)는 희망일 자동 입력하지 않음
        return;
      }

      if(type === "승인"){
        const titles = data.press
          .filter(p => p.status==="APPROVED" && p.approvedDate===dateStr)
          .map(p=>p.title);
        showToast(`${dateStr} : 승인 ${titles.length}건\n- ${titles.slice(0,2).join("\n- ")}${titles.length>2 ? "\n- ..." : ""}`);

        activateTab("staff");
        setTimeout(()=> boardSection.scrollIntoView({behavior:"smooth", block:"start"}), 80);
      }
    });

    cal.appendChild(dayDiv);
  });
}

/** ======================
 * ✅ 희망일 즉시 차단(승인 날짜 겹침 + 3영업일 + 주말/공휴일)
 * ====================== */
function validateDesiredDateImmediate(inputEl, ymdStr){
  if(!ymdStr) return true;

  // 0) 과거 선택 불가
  if(isPastDate(ymdStr)){
    inputEl.value = "";
    showToast(`${ymdStr} : 선택 불가\n사유: 지난 날짜는 선택할 수 없습니다.`);
    return false;
  }

  // 1) 3영업일 사전 신청 규칙
  if(!validateDesiredDateBusinessRule(inputEl, ymdStr, settings)) return false;

  // 2) 주말/공휴일/1일1개
  const chk = checkPublishable(ymdStr, data, settings);
  if(!chk.ok){
    inputEl.value = "";
    showToast(`${ymdStr} : 배포 불가\n사유: ${chk.reason}`);
    return false;
  }

  // 3) 이미 승인된 날짜 겹침(보강)
  if(isDesiredDateBlockedByApproved(ymdStr, data)){
    inputEl.value = "";
    showToast(`${ymdStr} : 배포 불가\n사유: 이미 승인된 보도자료가 있는 날짜(1일 1개)`);
    return false;
  }
  return true;
}

/** 리스트/표 렌더 */
function renderLists(){
  const name = staffName.value.trim();
  const mine = name ? data.press.filter(p => p.authorName === name).sort((a,b)=>b.createdAt-a.createdAt) : [];
  myList.innerHTML = mine.length ? mine.map(p => pressCard(p, {admin:false, mine:true})).join("") : `<div class="muted">이름을 입력하면 내 신청 목록이 보입니다.</div>`;

  const pending = data.press.filter(p => p.status==="SUBMITTED").sort((a,b)=>b.createdAt-a.createdAt);
  const approved = data.press.filter(p => p.status==="APPROVED").sort((a,b)=> (a.approvedDate||"").localeCompare(b.approvedDate||""));
  pendingList.innerHTML = pending.length ? pending.map(p => pressCard(p, {admin:true, mine:false})).join("") : `<div class="muted">대기중 신청이 없습니다.</div>`;
  approvedList.innerHTML = approved.length ? approved.map(p => pressCard(p, {admin:false, mine:false})).join("") : `<div class="muted">승인된 보도자료가 없습니다.</div>`;

  bindCardActions();
  renderBoardTable();
}

/** 검색 */
function matchesSearch(p, q){
  if(!q) return true;
  const st = statusKorean(p.status).label;
  const createdYmd = p.createdAt ? dateToYmdFromMillis(p.createdAt) : "";
  const text = [
    st,
    p.title || "",
    p.subtitle || "",
    p.authorName || "",
    p.authorPhone || "",
    p.authorEmail || "",
    p.approver || "",
    p.desiredDate || "",
    p.approvedDate || "",
    createdYmd
  ].join(" ").toLowerCase();
  return text.includes(q.toLowerCase());
}
function renderBoardTable(){
  const q = (boardSearch.value || "").trim();
  const rows = data.press
    .filter(p => (p.status==="APPROVED" || p.status==="SUBMITTED"))
    .filter(p => matchesSearch(p, q))
    .slice()
    .sort((a,b)=>{
      const aKey = a.status==="APPROVED" ? (a.approvedDate || "9999-12-31") : "9999-12-31";
      const bKey = b.status==="APPROVED" ? (b.approvedDate || "9999-12-31") : "9999-12-31";
      if(aKey !== bKey) return aKey.localeCompare(bKey);
      return (b.createdAt||0) - (a.createdAt||0);
    });

  if(rows.length===0){
    boardTableBody.innerHTML = `<tr><td colspan="6" class="muted">검색 결과가 없습니다.</td></tr>`;
    return;
  }

  boardTableBody.innerHTML = rows.map(p=>{
    const st = statusKorean(p.status);
    return `
      <tr>
        <td><span class="kstatus ${st.cls}">${st.label}</span></td>
        <td>${escapeHtml(p.title)}</td>
        <td>${escapeHtml(p.desiredDate || "-")}</td>
        <td>${escapeHtml(p.approvedDate || "-")}</td>
        <td>${escapeHtml(p.authorName || "-")}</td>
        <td>${p.status==="APPROVED" ? `<button class="btn small" type="button" data-act="downloadDoc" data-id="${p.id}">다운로드</button>` : `<span class="muted">-</span>`}</td>
      </tr>
    `;
  }).join("");

  bindBoardActions();
}

/** 변경 내역(최근/전체) */
function formatEditHistory(p){
  const h = Array.isArray(p.editHistory) ? p.editHistory : [];
  if(h.length===0) return `<div class="muted">변경 내역이 없습니다.</div>`;

  const items = h.slice().sort((a,b)=>(b.at||0)-(a.at||0)).slice(0,6);
  return items.map(e=>{
    const who = e.by === "admin" ? "관리자" : "신청자";
    const when = e.at ? new Date(e.at).toLocaleString("ko-KR") : "-";
    const changes = e.changes || {};
    const keys = Object.keys(changes);
    const fieldsKor = {
      title:"제목", subtitle:"부제목", body:"본문", desiredDate:"희망일", bigFileLinks:"대용량 링크"
    };
    const list = keys.map(k=>{
      const from = (changes[k]?.from ?? "");
      const to = (changes[k]?.to ?? "");
      if(k === "body"){
        return `
          <details style="margin-top:6px;">
            <summary class="summaryBtn">본문 변경(전/후)</summary>
            <div class="two" style="margin-top:10px;">
              <div>
                <div class="small" style="margin-bottom:6px;">변경 전</div>
                <div class="diffBox">${highlightBodyDiff(String(from).slice(0,2000) || "", String(to).slice(0,2000) || "").beforeHtml}</div>
              </div>
              <div>
                <div class="small" style="margin-bottom:6px;">변경 후</div>
                <div class="diffBox">${highlightBodyDiff(String(from).slice(0,2000) || "", String(to).slice(0,2000) || "").afterHtml}</div>
              </div>
            </div>
          </details>
        `;
      }
      return `<div class="small" style="margin-top:6px;"><b>${fieldsKor[k]||k}</b>: "${escapeHtml(String(from))}" → "${escapeHtml(String(to))}"</div>`;
    }).join("");

    return `
      <div class="item" style="background:#fff;">
        <div class="row" style="justify-content:space-between;">
          <div class="t">${who} 수정</div>
          <span class="pill">${when}</span>
        </div>
        ${list}
      </div>
    `;
  }).join("");
}

/** 카드 */
function pressCard(p, {admin, mine}){
  const desired = p.desiredDate || "-";
  const approved = p.approvedDate || "-";
  const st = statusKorean(p.status);

  const editedByAdmin = (p.editHistory || []).some(e=>e.by==="admin");
  const tagEdited = editedByAdmin ? `<span class="tag edited">관리자 첨삭 있음</span>` : "";

  const rejectReason = p.rejectReason ? `<div class="muted" style="margin-top:6px;">반려사유: ${escapeHtml(p.rejectReason)}</div>` : "";
  const imgs = (p.images && p.images.length)
    ? `<div class="muted" style="margin-top:6px;">사진: ${p.images.length}장</div>`
    : `<div class="muted" style="margin-top:6px;">사진: -</div>`;

  const links = (p.bigFileLinks && p.bigFileLinks.trim())
    ? `<div class="muted" style="margin-top:6px;">대용량 링크: <span class="mono">${escapeHtml(p.bigFileLinks.trim()).slice(0,120)}${p.bigFileLinks.trim().length>120 ? "..." : ""}</span></div>`
    : `<div class="muted" style="margin-top:6px;">대용량 링크: -</div>`;

  const canUserEdit = mine && (p.status==="SUBMITTED" || p.status==="REJECTED");
  const userEditBtn = canUserEdit ? `<button class="btn small" data-act="userEdit" data-id="${p.id}">수정</button>` : "";

  const adminEditBtn = admin ? `<button class="btn small" data-act="adminEdit" data-id="${p.id}">첨삭/수정</button>` : "";

  const adminBtns = admin ? `
    <div class="divider"></div>
    <div class="two">
      <label>
        승인 배포일(비어있으면 희망일)
        <input type="date" data-act="approveDate" data-id="${p.id}" value="${p.desiredDate || ""}">
      </label>
      <label>
        반려 사유(선택)
        <input data-act="rejectReason" data-id="${p.id}" placeholder="예: 문구/오탈자 수정 필요">
      </label>
    </div>
    <div class="row" style="margin-top:10px;">
      <button class="btn primary" data-act="approve" data-id="${p.id}">승인</button>
      <button class="btn danger" data-act="reject" data-id="${p.id}">반려</button>
    </div>
  ` : "";

  const historySection = (mine || admin) ? `
    <div class="divider"></div>
    <details>
      <summary class="summaryBtn">변경 내역 보기</summary>
      <div class="list" style="margin-top:10px;">
        ${formatEditHistory(p)}
      </div>
    </details>
  ` : "";

  return `
    <div class="item">
      <div class="top">
        <div style="min-width:0;">
          <div class="row" style="justify-content:space-between;">
            <div class="t">${escapeHtml(p.title)}</div>
            <div class="row" style="gap:8px;">
              ${tagEdited}
              <span class="kstatus ${st.cls}">${st.label}</span>
            </div>
          </div>
          <div class="muted" style="margin-top:6px;">
            작성자: <b>${escapeHtml(p.authorName)}</b>${p.authorPhone ? ` · ${escapeHtml(p.authorPhone)}` : ""}${p.authorEmail ? ` · ${escapeHtml(p.authorEmail)}` : ""} ·
            희망: <b>${escapeHtml(desired)}</b> · 배포: <b>${escapeHtml(approved)}</b>
          </div>
          ${p.approver ? `<div class="muted" style="margin-top:4px;">승인 관리자: <b>${escapeHtml(p.approver)}</b></div>` : ""}
          ${p.subtitle ? `<div class="muted" style="margin-top:4px;">부제: ${escapeHtml(p.subtitle)}</div>` : ""}
          ${imgs}
          ${links}
          ${rejectReason}
          <div class="row" style="margin-top:10px;">
            ${userEditBtn}
            ${adminEditBtn}
          </div>
        </div>
      </div>

      <details style="margin-top:10px;">
        <summary class="summaryBtn">본문 보기</summary>
        <div class="diffBox" style="margin-top:10px;">${escapeHtml(p.body)}</div>
      </details>

      ${historySection}
      ${adminBtns}
    </div>
  `;
}

/** 배포 예정/대기 현황: 다운로드 버튼 바인딩 */
function bindBoardActions(){
  document.querySelectorAll('[data-act="downloadDoc"]').forEach(btn=>{
    btn.onclick = ()=> downloadPressAsDoc(btn.getAttribute("data-id"));
  });
}
function bindCardActions(){
  document.querySelectorAll('[data-act="approve"]').forEach(btn=>{
    btn.onclick = ()=> adminApprove(btn.getAttribute("data-id"));
  });
  document.querySelectorAll('[data-act="reject"]').forEach(btn=>{
    btn.onclick = ()=> adminReject(btn.getAttribute("data-id"));
  });
  document.querySelectorAll('[data-act="adminEdit"]').forEach(btn=>{
    btn.onclick = ()=> openAdminEdit(btn.getAttribute("data-id"));
  });
  document.querySelectorAll('[data-act="userEdit"]').forEach(btn=>{
    btn.onclick = ()=> openUserEdit(btn.getAttribute("data-id"));
  });
}

/** 관리자 액션 */
function getAdminInput(id, act){
  const elx = document.querySelector(`[data-act="${act}"][data-id="${id}"]`);
  return elx ? elx.value : "";
}
function adminGuard(){
  const pass = (adminPass.value || "").trim();
  if(pass !== settings.adminCode){
    alert("관리자 패스코드가 올바르지 않습니다.");
    return false;
  }
  return true;
}

/** 변경 기록 유틸 */
function diffChanges(before, after){
  const fields = ["title","subtitle","body","desiredDate","bigFileLinks"];
  const changes = {};
  fields.forEach(k=>{
    const b = (before[k] ?? "");
    const a = (after[k] ?? "");
    if(String(b) !== String(a)){
      changes[k] = { from: b, to: a };
    }
  });
  return changes;
}
function pushHistory(p, by, changes){
  const keys = Object.keys(changes || {});
  if(keys.length===0) return;
  if(!Array.isArray(p.editHistory)) p.editHistory = [];
  p.editHistory.push({ by, at: Date.now(), changes });
}

/** 관리자 첨삭 모달 */
function openAdminEdit(id){
  if(!adminGuard()) return;

  const p = data.press.find(x=>x.id===id);
  if(!p){ alert("대상을 찾을 수 없습니다."); return; }
  if(p.status !== "SUBMITTED"){
    alert("대기중(접수) 상태에서만 첨삭할 수 있습니다.");
    return;
  }

  editingAdminId = id;
  aEditTitle.value = p.title || "";
  aEditSubtitle.value = p.subtitle || "";
  aEditBody.value = p.body || "";
  aEditDesiredDate.value = p.desiredDate || "";
  aEditLinks.value = p.bigFileLinks || "";
  aLastDiff.textContent = "아직 변경 내역이 없습니다.";

  // min 설정(과거/3영업일 방지 보조)
  const minYmd = earliestDesiredYmd(settings);
  aEditDesiredDate.min = minYmd;

  dlgEditAdmin.showModal();
}
function adminEditSave(){
  if(!adminGuard()) return;
  const id = editingAdminId;
  const p = data.press.find(x=>x.id===id);
  if(!p) return;

  const dd = aEditDesiredDate.value || "";
  if(dd){
    if(isPastDate(dd)){ aEditDesiredDate.value=""; showToast(`${dd} : 선택 불가\n사유: 지난 날짜`); return; }
    if(!validateDesiredDateBusinessRule(aEditDesiredDate, dd, settings)) return;
    const chk = checkPublishable(dd, data, settings);
    if(!chk.ok){ aEditDesiredDate.value=""; showToast(`${dd} : 배포 불가\n사유: ${chk.reason}`); return; }
  }

  const before = {
    title: p.title || "",
    subtitle: p.subtitle || "",
    body: p.body || "",
    desiredDate: p.desiredDate || "",
    bigFileLinks: p.bigFileLinks || ""
  };
  const after = {
    title: aEditTitle.value.trim(),
    subtitle: aEditSubtitle.value.trim(),
    body: aEditBody.value.trim(),
    desiredDate: aEditDesiredDate.value || "",
    bigFileLinks: aEditLinks.value || ""
  };

  const changes = diffChanges(before, after);
  pushHistory(p, "admin", changes);

  p.title = after.title;
  p.subtitle = after.subtitle || null;
  p.body = after.body;
  p.desiredDate = after.desiredDate || null;
  p.bigFileLinks = after.bigFileLinks || "";

  saveData(data);
  renderHelp();
  renderCalendar();
  renderLists();

  const keys = Object.keys(changes);
  if(keys.length===0){
    aLastDiff.textContent = "변경된 내용이 없습니다.";
  }else{
    const lines = keys.map(k=>{
      if(k==="body") return `- 본문: (변경됨)`;
      const kor = ({title:"제목",subtitle:"부제목",desiredDate:"희망일",bigFileLinks:"대용량 링크"})[k] || k;
      return `- ${kor}: "${String(changes[k].from)}" → "${String(changes[k].to)}"`;
    });
    aLastDiff.textContent = `저장 완료!\n${lines.join("\n")}`;
  }

  kakaoText.value =
`[제주창조경제혁신센터] 보도자료 첨삭 완료 안내
- 제목: ${p.title}
- 상태: 대기중(접수)
※ ‘내 신청 목록’에서 “변경 내역 보기”로 수정 내용을 확인할 수 있습니다.`;
}

/** 신청자 수정 모달 */
function openUserEdit(id){
  const name = staffName.value.trim();
  if(!name){ alert("내 이름을 먼저 입력해주세요."); return; }

  const p = data.press.find(x=>x.id===id);
  if(!p){ alert("대상을 찾을 수 없습니다."); return; }
  if(p.authorName !== name){
    alert("본인이 신청한 보도자료만 수정할 수 있습니다.");
    return;
  }
  if(!(p.status==="SUBMITTED" || p.status==="REJECTED")){
    alert("대기중/반려 상태에서만 수정할 수 있습니다.");
    return;
  }

  editingUserId = id;
  uEditTitle.value = p.title || "";
  uEditSubtitle.value = p.subtitle || "";
  uEditBody.value = p.body || "";
  uEditDesiredDate.value = p.desiredDate || "";
  uEditLinks.value = p.bigFileLinks || "";

  // min 설정(과거/3영업일 방지 보조)
  const minYmd = earliestDesiredYmd(settings);
  uEditDesiredDate.min = minYmd;

  dlgEditUser.showModal();
}
function userEditSave(){
  const id = editingUserId;
  const name = staffName.value.trim();
  const p = data.press.find(x=>x.id===id);
  if(!p || p.authorName !== name) return;

  const dd = uEditDesiredDate.value || "";
  if(dd){
    if(isPastDate(dd)){ uEditDesiredDate.value=""; showToast(`${dd} : 선택 불가\n사유: 지난 날짜`); return; }
    if(!validateDesiredDateBusinessRule(uEditDesiredDate, dd, settings)) return;
    const chk = checkPublishable(dd, data, settings);
    if(!chk.ok){ uEditDesiredDate.value=""; showToast(`${dd} : 배포 불가\n사유: ${chk.reason}`); return; }
  }

  const before = {
    title: p.title || "",
    subtitle: p.subtitle || "",
    body: p.body || "",
    desiredDate: p.desiredDate || "",
    bigFileLinks: p.bigFileLinks || ""
  };
  const after = {
    title: uEditTitle.value.trim(),
    subtitle: uEditSubtitle.value.trim(),
    body: uEditBody.value.trim(),
    desiredDate: uEditDesiredDate.value || "",
    bigFileLinks: uEditLinks.value || ""
  };

  if(!after.title || !after.body){
    alert("제목/본문은 필수입니다.");
    return;
  }

  const changes = diffChanges(before, after);
  pushHistory(p, "author", changes);

  p.title = after.title;
  p.subtitle = after.subtitle || null;
  p.body = after.body;
  p.desiredDate = after.desiredDate || null;
  p.bigFileLinks = after.bigFileLinks || "";

  saveData(data);
  renderHelp();
  renderCalendar();
  renderLists();

  dlgEditUser.close();
  showToast("수정 저장 완료");
}

/** 승인/반려 */
function adminApprove(id){
  if(!adminGuard()) return;

  const date = getAdminInput(id, "approveDate") || "";
  const pr = data.press.find(x=>x.id===id);
  if(!pr){ alert("대상을 찾을 수 없습니다."); return; }
  const target = date || pr.desiredDate;

  if(!target){
    alert("승인 배포일 또는 희망일이 필요합니다.");
    return;
  }

  // ✅ 승인도 과거 금지
  if(isPastDate(target)){
    alert("과거 날짜는 배포일로 승인할 수 없습니다.");
    return;
  }

  const chk = checkPublishable(target, data, settings);
  if(!chk.ok){
    alert("배포 불가: " + chk.reason);
    return;
  }

  pr.status = "APPROVED";
  pr.approvedDate = target;
  pr.approvedAt = Date.now();

  saveData(data);
  renderHelp();
  renderCalendar();
  renderLists();
  renderApprovedTitlesForDate(target);

  kakaoText.value =
`[제주창조경제혁신센터] 보도자료 승인 완료
- 제목: ${pr.title}
- 배포일: ${pr.approvedDate}
(확인 필요 시 담당자에게 문의 부탁드립니다.)`;
}
function adminReject(id){
  if(!adminGuard()) return;

  const reason = getAdminInput(id, "rejectReason") || "반려";
  const pr = data.press.find(x=>x.id===id);
  if(!pr){ alert("대상을 찾을 수 없습니다."); return; }

  pr.status = "REJECTED";
  pr.rejectReason = reason;
  pr.approvedDate = null;
  pr.approvedAt = null;

  saveData(data);
  renderHelp();
  renderCalendar();
  renderLists();

  kakaoText.value =
`[제주창조경제혁신센터] 보도자료 반려 안내
- 제목: ${pr.title}
- 사유: ${reason}
수정 후 다시 신청 부탁드립니다.`;
}

/** 이미지 업로드 */
imgHelp.textContent = `※ ${MAX_IMAGE_MB}MB 이하 이미지 권장 · 최대 ${MAX_IMAGE_COUNT}장 (큰 파일은 링크로 공유)`;
imageFiles.addEventListener("change", async ()=>{
  const files = Array.from(imageFiles.files || []);
  if(files.length===0) return;

  if(selectedFiles.length + files.length > MAX_IMAGE_COUNT){
    alert(`사진은 최대 ${MAX_IMAGE_COUNT}장까지 업로드할 수 있습니다.`);
    imageFiles.value = "";
    return;
  }

  for(const f of files){
    if(f.size > MAX_IMAGE_BYTES){
      alert(`"${f.name}" 파일 용량이 큽니다.\n- 권장: ${MAX_IMAGE_MB}MB 이하\n- 큰 파일은 Agit/드라이브 링크로 전달해주세요.`);
      continue;
    }
    const dataUrl = await readAsDataURL(f);
    selectedFiles.push({ name: f.name, type: f.type, dataUrl });
  }

  imageFiles.value = "";
  renderPreview();
});
function renderPreview(){
  if(selectedFiles.length===0){
    previewArea.style.display = "none";
    previewArea.innerHTML = "";
    return;
  }
  previewArea.style.display = "flex";
  previewArea.innerHTML = selectedFiles.map((im, idx)=>`
    <div class="thumbWrap">
      <div class="thumbBox">
        <img class="thumb" src="${im.dataUrl}" alt="${escapeHtml(im.name)}">
        <button class="xbtn" type="button" data-del="${idx}" aria-label="삭제">×</button>
      </div>
      <div class="fname" title="${escapeHtml(im.name)}">${escapeHtml(im.name)}</div>
    </div>
  `).join("");

  previewArea.querySelectorAll("[data-del]").forEach(btn=>{
    btn.addEventListener("click", (e)=>{
      e.preventDefault();
      e.stopPropagation();
      const idx = Number(btn.getAttribute("data-del"));
      selectedFiles.splice(idx, 1);
      renderPreview();
    });
  });
}
function readAsDataURL(file){
  return new Promise((resolve, reject)=>{
    const r = new FileReader();
    r.onload = ()=> resolve(r.result);
    r.onerror = reject;
    r.readAsDataURL(file);
  });
}

/** 작성팁 버튼 */
btnInsertTips.addEventListener("click", ()=>{
  if(body.value && body.value.trim().length > 0){
    const ok = confirm("본문에 이미 내용이 있습니다.\n작성팁 예시 템플릿을 앞에 추가할까요?");
    if(!ok) return;
    body.value = PRESS_TIPS_TEMPLATE + "\n" + body.value;
  }else{
    body.value = PRESS_TIPS_TEMPLATE;
  }
  body.focus();
  body.setSelectionRange(body.value.length, body.value.length);
});

/** ✅ 희망일 입력 즉시 검증 */
desiredDate.addEventListener("change", ()=>{
  const v = desiredDate.value || "";
  if(!v) return;
  validateDesiredDateImmediate(desiredDate, v);
});

/** 신청 제출 */
formSubmit.addEventListener("submit", (e)=>{
  e.preventDefault();
  staffMsg.textContent = "";
  staffMsg.style.borderColor = "#cbd5e1";

  const name = staffName.value.trim();
  const phone = staffPhone.value.trim();
  const email = staffEmail.value.trim();
  const apv = (approver.value || "").trim();

  if(!name){ staffMsg.textContent = "내 이름을 입력해주세요."; staffMsg.style.borderColor = "#fecaca"; return; }
  if(!phone){ staffMsg.textContent = "내 연락처는 필수입니다."; staffMsg.style.borderColor = "#fecaca"; return; }
  if(!email){ staffMsg.textContent = "이메일은 필수입니다."; staffMsg.style.borderColor = "#fecaca"; return; }
  if(!apv){ staffMsg.textContent = "승인 관리자를 선택해주세요."; staffMsg.style.borderColor = "#fecaca"; return; }

  const t = title.value.trim();
  const b = body.value.trim();
  if(!t || !b){
    staffMsg.textContent = "제목/본문은 필수입니다.";
    staffMsg.style.borderColor = "#fecaca";
    return;
  }

  const want = desiredDate.value || null;
  if(!want){
    staffMsg.textContent = "배포 희망일은 필수입니다.";
    staffMsg.style.borderColor = "#fecaca";
    return;
  }

  if(!validateDesiredDateImmediate(desiredDate, want)){
    staffMsg.textContent = "배포 희망일을 다시 선택해주세요.";
    staffMsg.style.borderColor = "#fecaca";
    return;
  }

  const linkText = (bigFileLinks.value || "").trim();
  if(selectedFiles.length === 0 && !linkText){
    staffMsg.textContent = "보도용 사진이 없으면, 반드시 대용량 파일 전달 링크(드라이브/Agit 등)를 입력해야 신청할 수 있습니다.";
    staffMsg.style.borderColor = "#fecaca";
    return;
  }

  const pr = {
    id: cryptoRandomId(),
    authorName: name,
    authorPhone: phone,
    authorEmail: email,
    approver: apv,
    title: t,
    subtitle: subtitle.value.trim() || null,
    body: b,
    desiredDate: want,
    approvedDate: null,
    status: "SUBMITTED",
    rejectReason: null,
    images: selectedFiles.slice(),
    bigFileLinks: bigFileLinks.value || "",
    createdAt: Date.now(),
    approvedAt: null,
    editHistory: []
  };

  data.press.unshift(pr);

  try{
    saveData(data);
  }catch(err){
    data.press = data.press.filter(x=>x.id!==pr.id);
    alert("저장에 실패했어요.\n- 사진 용량/장수를 줄여서 다시 시도해주세요.\n- 큰 파일은 Agit/드라이브 링크로 전달해주세요.");
    return;
  }

  title.value = "";
  subtitle.value = "";
  body.value = "";
  bigFileLinks.value = "";
  desiredDate.value = "";
  selectedFiles = [];
  renderPreview();

  staffMsg.textContent = "신청 완료! 관리자 승인 대기중입니다.";
  staffMsg.style.borderColor = "#bbf7d0";

  renderHelp();
  renderCalendar();
  renderLists();
});

/** 내 신청 목록 리렌더 */
staffName.addEventListener("input", ()=> renderLists());

/** ✅ 검색: 버튼 클릭 시에만 실행 + Enter 지원 */
btnSearch.addEventListener("click", ()=> renderBoardTable());
boardSearch.addEventListener("keydown", (e)=>{
  if(e.key === "Enter"){
    e.preventDefault();
    renderBoardTable();
  }
});
btnClearSearch.addEventListener("click", ()=>{
  boardSearch.value = "";
  renderBoardTable();
});

/** 캘린더 이동 */
prevMonth.onclick = ()=>{ cursor = new Date(cursor.getFullYear(), cursor.getMonth()-1, 1); renderCalendar(); };
nextMonth.onclick = ()=>{ cursor = new Date(cursor.getFullYear(), cursor.getMonth()+1, 1); renderCalendar(); };

/** 설정 */
function renderSettingsUI(){
  setAdminCode.value = settings.adminCode;
  setHolidays.value = settings.holidays.join("\n");
}
renderSettingsUI();

function applyDateMins(){
  const minYmd = earliestDesiredYmd(settings);
  desiredDate.min = minYmd;
  uEditDesiredDate.min = minYmd;
  aEditDesiredDate.min = minYmd;
}
applyDateMins();

btnSaveSettings.onclick = ()=>{
  const ac = setAdminCode.value.trim() || DEFAULT_SETTINGS.adminCode;
  const hs = setHolidays.value.split("\n").map(s=>s.trim()).filter(Boolean);

  settings = { adminCode: ac, holidays: hs };
  saveSettings(settings);
  setHints();
  settingsHint.textContent = "저장 완료!";
  setTimeout(()=> settingsHint.textContent="", 1500);

  applyDateMins();
  renderHelp();
  renderCalendar();
};

/** 카톡 문구 복사 */
btnCopyKakao.onclick = async ()=>{
  try{
    await navigator.clipboard.writeText(kakaoText.value || "");
    copyHint.textContent = "복사 완료! 카카오톡에 붙여넣기 하세요.";
    setTimeout(()=> copyHint.textContent="", 2000);
  }catch(e){
    copyHint.textContent = "복사 실패: 브라우저가 클립보드를 막았을 수 있어요. 직접 드래그해서 복사하세요.";
    setTimeout(()=> copyHint.textContent="", 3000);
  }
};

/** 관리자 초기화 */
btnResetAdmin.addEventListener("click", ()=>{
  if(!adminGuard()) return;
  const ok = confirm("정말 전체 데이터를 초기화할까요? (되돌릴 수 없음)");
  if(!ok) return;

  localStorage.removeItem(LS_KEY);
  localStorage.removeItem(LS_SETTINGS);
  settings = loadSettings();
  data = loadData();

  adminPass.value = "";
  setHints();
  renderSettingsUI();
  applyDateMins();
  renderHelp();
  renderCalendar();
  renderLists();
  approvedTitles.innerHTML = `<div class="muted">아직 선택된 날짜가 없습니다.</div>`;
  selectedFiles = [];
  renderPreview();
  showToast("초기화 완료");
});

/** 엑셀 내보내기 */
btnExportXlsx.onclick = ()=>{
  const fromStr = exportFrom.value;
  const toStr = exportTo.value;

  if(!fromStr || !toStr){
    alert("엑셀 기간 시작/종료 날짜를 모두 선택해주세요.");
    return;
  }
  if(fromStr > toStr){
    alert("기간이 올바르지 않습니다. 시작일이 종료일보다 늦습니다.");
    return;
  }

  const from = parseYMD(fromStr);
  const to = parseYMD(toStr);
  to.setHours(23,59,59,999);

  const rows = data.press.filter(p=>{
    const created = new Date(p.createdAt || 0);
    const createdIn = (created >= from && created <= to);
    const desiredIn = p.desiredDate ? (parseYMD(p.desiredDate) >= from && parseYMD(p.desiredDate) <= to) : false;
    const approvedIn = p.approvedDate ? (parseYMD(p.approvedDate) >= from && parseYMD(p.approvedDate) <= to) : false;
    return createdIn || desiredIn || approvedIn;
  });

  if(rows.length === 0){
    alert("해당 기간에 포함되는 보도자료가 없습니다.");
    return;
  }

  const aoa = [];
  aoa.push([
    "상태", "제목", "부제목", "작성자", "연락처", "이메일", "승인 관리자",
    "신청일", "희망일", "배포일",
    "반려사유", "대용량 링크", "이미지 장수", "수정기록(건수)"
  ]);

  rows.slice().sort((a,b)=>(b.createdAt||0)-(a.createdAt||0)).forEach(p=>{
    const st = statusKorean(p.status).label;
    const createdYmd = p.createdAt ? dateToYmdFromMillis(p.createdAt) : "";
    aoa.push([
      st,
      p.title || "",
      p.subtitle || "",
      p.authorName || "",
      p.authorPhone || "",
      p.authorEmail || "",
      p.approver || "",
      createdYmd,
      p.desiredDate || "",
      p.approvedDate || "",
      p.rejectReason || "",
      (p.bigFileLinks || "").replace(/\n/g, " "),
      (p.images && p.images.length) ? p.images.length : 0,
      (p.editHistory && p.editHistory.length) ? p.editHistory.length : 0
    ]);
  });

  const ws = XLSX.utils.aoa_to_sheet(aoa);
  ws["!cols"] = [
    {wch:10},{wch:50},{wch:36},{wch:12},{wch:16},{wch:26},{wch:14},
    {wch:12},{wch:12},{wch:12},
    {wch:28},{wch:40},{wch:10},{wch:12}
  ];
  const wb = XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb, ws, "보도자료");

  const filename = `보도자료_${fromStr}_~_${toStr}.xlsx`;
  XLSX.writeFile(wb, filename);
  showToast(`엑셀 다운로드 완료: ${filename}`);
};

/** 모달 이벤트 */
dlgEditUserClose.onclick = ()=> dlgEditUser.close();
uEditCancel.onclick = ()=> dlgEditUser.close();
uEditSave.onclick = ()=> userEditSave();

dlgEditAdminClose.onclick = ()=> dlgEditAdmin.close();
aEditCancel.onclick = ()=> dlgEditAdmin.close();
aEditSave.onclick = ()=> adminEditSave();

/** 초기 렌더 */
renderHelp();
applyDateMins();
renderCalendar();
renderLists();

/** 도우미 */
function cryptoRandomId(){
  const s = Array.from(crypto.getRandomValues(new Uint8Array(12))).map(b=>b.toString(16).padStart(2,"0")).join("");
  return "pr_" + s;
}
function highlightBodyDiff(beforeText, afterText){
  const b = String(beforeText ?? "");
  const a = String(afterText ?? "");
  if(b === a) return { beforeHtml: escapeHtml(b), afterHtml: escapeHtml(a) };

  const minLen = Math.min(b.length, a.length);
  let i = 0;
  while(i < minLen && b[i] === a[i]) i++;

  let j = 0;
  while(j < (minLen - i) && b[b.length - 1 - j] === a[a.length - 1 - j]) j++;

  const bMid = b.slice(i, b.length - j);
  const aMid = a.slice(i, a.length - j);

  const bHtml = escapeHtml(b.slice(0,i)) + (bMid ? `<span class="diff-del">${escapeHtml(bMid)}</span>` : "") + escapeHtml(b.slice(b.length - j));
  const aHtml = escapeHtml(a.slice(0,i)) + (aMid ? `<span class="diff-red">${escapeHtml(aMid)}</span>` : "") + escapeHtml(a.slice(a.length - j));
  return { beforeHtml: bHtml, afterHtml: aHtml };
}
function escapeHtml(str){
  return String(str ?? "")
    .replaceAll("&","&amp;")
    .replaceAll("<","&lt;")
    .replaceAll(">","&gt;")
    .replaceAll('"',"&quot;")
    .replaceAll("'","&#039;");
}
function unescapeHtml(str){
  return String(str ?? "")
    .replaceAll("&lt;","<")
    .replaceAll("&gt;",">")
    .replaceAll("&quot;",'"')
    .replaceAll("&#039;","'")
    .replaceAll("&amp;","&");
}
</script>

</body>
</html>

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

    /* ✅ 레이아웃: 1열(신청폼 메인), 캘린더는 접기/펼치기 */
    .grid{ display:grid; grid-template-columns: 1fr; gap:14px; }

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
    .two{display:grid; grid-template-columns:1fr 1fr; gap:10px; align-items:start;}
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

    /* Calendar */
    .calendar{
      display:grid;
      grid-template-columns: repeat(7, 1fr);
      gap:8px;
      user-select:none;
      min-width:0;
    }
    .dow{font-size:12px; color:var(--muted); text-align:center; padding:6px 0; font-weight:900;}
    .day{
      border:1px solid var(--line);
      border-radius:14px;
      padding:10px;
      background:#fff;
      position:relative;
      min-height:72px;
      overflow:hidden;
    }
    .day.out{background:#f8fafc; color:#94a3b8;}
    .day.disabled{opacity:.55; filter: grayscale(.1); pointer-events:none;}
    .day .n{font-weight:950; font-size:13px;}

    @media (max-width: 520px){
      .wrap{padding:12px;}
      .day{min-height:64px; padding:8px;}
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
    th{background:#f8fafc; color:#334155; font-weight:950; text-align:left;}
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
    .toast.show{opacity:1; transform:translateX(-50%) translateY(-2px);}

    .searchBar{
      display:flex;
      gap:10px;
      flex-wrap:wrap;
      align-items:flex-end;
      margin:8px 0 10px;
    }
    .searchBar label{min-width:240px; flex:1;}

    /* ✅ 엑셀(보드로 이동) */
    .exportBarBoard{
      display:flex;
      gap:10px;
      flex-wrap:wrap;
      align-items:flex-end;
      margin:8px 0 10px;
    }
    .exportBarBoard label{min-width:200px; flex:1;}
    .exportBarBoard .btn{white-space:nowrap;}

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

    /* 모달 */
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

    /* ✅ 필수 표시 + 에러 표시 */
    label.required{ font-weight:950; color:#0f172a; }
    label.required .reqMark{ color: var(--bad); margin-left:4px; font-weight:950; }
    .inputError{
      border-color: #fecaca !important;
      box-shadow: 0 0 0 3px rgba(220,38,38,.12);
    }
    .errorText{
      margin-top:6px;
      font-size:12px;
      color: var(--bad);
      font-weight:900;
      min-height: 14px;
    }
  </style>
</head>

<body>
<div class="wrap">
  <header>
    <div class="title">
      <h1>JCCEI 보도자료 캘린더 MVP</h1>
      <p>정적사이트 프로토타입 · 주말/공휴일/1일1개 승인 규칙 반영</p>
    </div>
    <div class="bar"></div>
  </header>

  <div class="card">
    <div class="row" style="justify-content:space-between;">
      <div class="tabs">
        <button class="tab active" data-view="staff" id="tabStaff">신청</button>
        <button class="tab" data-view="admin" id="tabAdmin">승인</button>
        <button class="tab" data-view="settings" id="tabSettings">설정</button>
      </div>
      <div class="row">
        <!-- ✅ 다중 관리자 코드: 선택한 관리자 기준 힌트 -->
        <span class="pill">선택한 관리자 코드: <span class="mono" id="adminCodeHint"></span></span>
      </div>
    </div>
  </div>

  <!-- ✅ 안내문구는 한 곳에 모아 노출 -->
  <div class="card" id="guideBox" style="margin-top:14px;">
    <div class="row" style="justify-content:space-between;">
      <h2 style="margin:0;">안내</h2>
      <button class="btn ghost small" id="btnToggleGuide" type="button">접기</button>
    </div>
    <div class="divider"></div>
    <div class="note" id="guideBody">
      <b>캘린더의 [가능]/[불가]/[승인]을 눌러 확인하세요.</b><br/>
      - [가능]: 배포 가능 안내 팝업<br/>
      - [불가]: 불가 사유 팝업<br/>
      - [승인]: “배포 예정/대기 현황”으로 이동하여 승인 건 확인<br/><br/>
      <b>규칙</b>: 주말 배포 불가 · 공휴일 배포 불가 · 승인 기준 1일 1개 · 신청일(오늘) 기준 주말/공휴일 제외 3영업일 이내는 신청 불가
    </div>
  </div>

  <div class="grid" style="margin-top:14px;">
    <div class="card">
      <!-- 신청 -->
      <div id="view_staff">
        <h2>보도자료 신청</h2>

        <div class="two">
           <label class="required">
            <span style="font-weight: normal;">내 이름</span>
            <span class="reqMark">*</span>
            <input id="staffName" placeholder="예: 홍길동" />
            <div class="errorText" id="err_staffName"></div>
          </label>
          <label class="required">
          <span style="font-weight: normal;">내 연락처</span>
          <span class="reqMark">*</span>
          <input id="staffPhone" placeholder="예: 010-1234-5678" />
          <div class="errorText" id="err_staffPhone"></div>
        </label>
        </div>

        <div class="two" style="margin-top:10px;">
          <label class="required">
            이메일 <span class="reqMark">*</span>
            <input id="staffEmail" placeholder="예: example@jccei.kr" />
            <div class="errorText" id="err_staffEmail"></div>
          </label>

          <label class="required">
            승인 관리자 <span class="reqMark">*</span>
            <select id="approver"></select>
            <div class="errorText" id="err_approver"></div>
          </label>
        </div>

        <div class="divider"></div>

        <form id="formSubmit" class="list">
          <label class="required">
            제목 <span class="reqMark">*</span>
            <input id="title" required placeholder="예: 제주창조경제혁신센터, ○○ 프로그램 성료" />
            <div class="errorText" id="err_title"></div>
          </label>

          <label>
            부제목(선택)
            <input id="subtitle" placeholder="예: 도내 스타트업 20개사 참여…" />
          </label>

          <div class="row" style="justify-content:space-between; align-items:flex-start;">
            <label class="required" style="flex:1; min-width:260px;">
              본문 <span class="reqMark">*</span>
              <textarea id="body" required></textarea>
              <div class="errorText" id="err_body"></div>
            </label>
            <div style="width:180px; min-width:180px;">
              <button class="btn small" type="button" id="btnInsertTips">작성팁 예시 넣기</button>
              <div class="small" style="margin-top:6px;">※ 클릭 시 본문에 템플릿이 자동 입력됩니다.</div>
            </div>
          </div>

          <div class="two">
            <label class="required">
              배포 희망일 <span class="reqMark">*</span>
              <input id="desiredDate" type="date" required />
              <div class="errorText" id="err_desiredDate"></div>
              <span class="small">※ 승인된 날짜/주말/공휴일/3영업일 이내는 선택 불가</span>
            </label>

            <div style="min-width:0;">
              <button class="btn" type="button" id="btnOpenCalendar">캘린더 열기</button>
              <div class="small" style="margin-top:6px;">※ 캘린더에서 날짜를 누르면 희망일이 자동 입력됩니다.</div>
            </div>
          </div>

          <div class="two">
            <label>
              보도용 사진 업로드(업로드 또는 링크, 여러 장 가능)
              <input id="imageFiles" type="file" accept="image/*" multiple />
              <span class="small" id="imgHelp"></span>
              <span class="small">※ 용량이 큰 파일은 <b>Agit/드라이브 링크</b>로 전달해 주세요.</span>
            </label>

            <label>
              대용량 파일 전달 링크(Agit/드라이브 등, 사진이 없으면 필수)
              <textarea id="bigFileLinks" placeholder="예) https://drive.google.com/...&#10;예) https://agit..."></textarea>
              <div class="errorText" id="err_bigFileLinks"></div>
              <span class="small">※ 이미지/자료가 크면 업로드 대신 링크로 공유해 주세요.</span>
            </label>
          </div>

          <div id="previewArea" class="imgRow" aria-label="사진 미리보기" style="display:none;"></div>

          <button class="btn primary" type="submit">신청하기</button>
          <div class="note" id="staffMsg">신청 후 관리자가 승인하면 캘린더에 등록됩니다.</div>
        </form>

        <div class="divider"></div>
        <h2>내 신청 목록</h2>
        <div class="note" style="margin-bottom:10px;">
          내 신청 목록에서 <b>대기중/반려</b> 건은 <b>수정</b>할 수 있습니다.
        </div>
        <div class="list" id="myList"></div>

        <div class="divider"></div>

        <!-- 승인 클릭 시 여기로 스크롤 -->
        <div id="boardSection"></div>

        <h2>배포 예정/대기 현황</h2>

        <!-- ✅ 엑셀(보드로 이동): "배포된(승인)" 건만 기간 내 다운로드 -->
        <div class="exportBarBoard">
          <label>
            엑셀 기간 시작
            <input id="exportFrom" type="date">
          </label>
          <label>
            엑셀 기간 종료
            <input id="exportTo" type="date">
          </label>
          <button class="btn primary" id="btnExportXlsx" type="button">엑셀 내려받기</button>
          <span class="small">※ 기간 내 <b>배포 예정(승인)</b> 보도자료 목록만 내려받습니다.</span>
        </div>

        <!-- ✅ 검색: 버튼을 눌러 실행 -->
        <div class="searchBar">
          <label>
            검색(제목/작성자/상태/날짜)
            <input id="boardSearch" placeholder="예: 1월, 박윤혁, 배포 예정, 오픈그라운드..." />
          </label>
          <button class="btn primary" id="btnDoSearch" type="button">검색</button>
          <button class="btn" id="btnClearSearch" type="button">초기화</button>
        </div>

        <div class="note" style="margin-bottom:10px;">
          표는 <b>배포 예정(승인)</b>과 <b>대기중</b>만 표시됩니다. (반려는 내 신청 목록에서 확인)
        </div>

        <div style="overflow:auto;">
          <table>
            <thead>
              <tr>
                <th style="min-width:90px;">상태</th>
                <th style="min-width:260px;">제목</th>
                <th style="min-width:110px;">희망일</th>
                <th style="min-width:120px;">작성자</th>
                <th style="min-width:110px;">다운로드</th>
              </tr>
            </thead>
            <tbody id="boardTableBody">
              <tr><td colspan="5" class="muted">데이터가 없습니다.</td></tr>
            </tbody>
          </table>
        </div>
      </div>

      <!-- 승인 -->
      <div id="view_admin" class="hidden">
        <h2>관리자 승인/반려</h2>

        <!-- ✅ "내 관리자" 선택 + 패스코드 (관리자별 코드) -->
        <div class="two">
          <label class="required">
            내 관리자 이름 <span class="reqMark">*</span>
            <select id="adminWho"></select>
            <div class="errorText" id="err_adminWho"></div>
          </label>

          <label class="required">
            관리자 패스코드 <span class="reqMark">*</span>
            <input id="adminPass" type="password" placeholder="설정 탭에서 변경 가능" />
            <div class="errorText" id="err_adminPass"></div>
          </label>
        </div>

        <div class="divider"></div>

        <h2>승인 대기</h2>
        <div class="note" style="margin-bottom:10px;">
          <b>내가 ‘승인 관리자’로 지정된</b> 대기 건만 표시됩니다.<br/>
          대기 건에서 <b>‘첨삭/수정’</b>을 눌러 문구를 고친 뒤 승인할 수 있어요.
        </div>
        <div class="list" id="pendingList"></div>

        <div class="divider"></div>

        <h2>승인 완료</h2>
        <div class="note" style="margin-bottom:10px;">
          <b>내가 ‘승인 관리자’인</b> 승인 건만 표시됩니다.
        </div>
        <div class="list" id="approvedList"></div>

        <div class="divider"></div>

        <h2>카카오톡 안내문(복사해서 보내기)</h2>
        <div class="note">
          정적 사이트(HTML만)에서는 카카오톡 “자동 발송”이 어렵습니다.<br/>
          대신 승인/반려/첨삭 저장 시 자동 생성되는 문구를 <b>복사</b>해서 카톡으로 보내면 됩니다.
        </div>
        <div class="divider"></div>
        <textarea id="kakaoText" placeholder="승인/반려/첨삭 저장을 하면 여기에 문구가 생성됩니다."></textarea>
        <div class="row" style="margin-top:10px;">
          <button class="btn" id="btnCopyKakao">문구 복사</button>
          <span class="small" id="copyHint"></span>
        </div>

        <div class="divider"></div>

        <h2>데이터 관리</h2>
        <div class="note" style="margin-bottom:10px;">
          <b>전체 초기화</b>는 관리자 패스코드를 입력한 경우에만 가능합니다.<br/>
          (주의: 되돌릴 수 없음)
        </div>
        <button class="btn danger" id="btnResetAdmin">전체 초기화(관리자)</button>
      </div>

      <!-- 설정 -->
      <div id="view_settings" class="hidden">
        <h2>설정</h2>
        <div class="note">
          공휴일/관리자별 패스코드를 여기서 바꾸면 됩니다.<br/>
          (공휴일은 <b>YYYY-MM-DD</b> 형태로 한 줄에 하나씩 입력)
        </div>

        <div class="divider"></div>

        <h2 style="margin:0 0 8px;">관리자별 패스코드</h2>
        <div class="note" style="margin-bottom:10px;">
          아래 관리자별 패스코드를 설정하세요. (각각 다르게 설정 가능)
        </div>
        <div class="list" id="adminCodesBox"></div>

        <div class="divider"></div>

        <label>
          공휴일 목록(YYYY-MM-DD, 한 줄에 하나)
          <textarea id="setHolidays" placeholder="2026-01-01&#10;2026-02-09"></textarea>
        </label>

        <div class="row" style="margin-top:10px;">
          <button class="btn primary" id="btnSaveSettings">설정 저장</button>
          <span class="small" id="settingsHint"></span>
        </div>

        <div class="divider"></div>

        <h2>정적사이트 한계(짧게)</h2>
        <div class="note">
          이 HTML 버전은 데이터가 <b>각자 브라우저에만 저장</b>됩니다.<br/>
          “직원 모두가 같은 데이터를 공유”하려면 중앙 저장소(예: Google Sheet/Firebase)가 필요합니다.
        </div>
      </div>
    </div>

    <!-- ✅ 캘린더: 접기/펼치기 (기본 닫힘) -->
    <details class="card" id="calendarDetails">
      <summary class="summaryBtn">📅 배포 캘린더 열기/닫기</summary>

      <div style="margin-top:12px;">
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
    </details>
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
    <div class="note" style="margin-top:10px;">
      ※ 희망일이 <b>이미 승인된 날짜</b>와 겹치면 선택할 수 없습니다(자동으로 비워짐).<br/>
      ※ 사진은 이 MVP에서 “수정 시 재업로드”까지는 단순화했습니다.
    </div>
  </div>
  <div class="modalFoot">
    <button class="btn" id="uEditCancel">취소</button>
    <button class="btn primary" id="uEditSave">저장</button>
  </div>
</dialog>

<!-- ✅ 관리자 첨삭 모달 (요구사항 반영: 대용량 링크 삭제) -->
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
      <div class="note" style="min-height:84px; display:flex; align-items:center;">
        ※ ‘첨삭 저장’만 누르면 첨삭 상태로만 남고, 승인/반려는 별도 처리합니다.<br/>
        ※ 희망일은 “이미 승인된 날짜”와 겹치면 선택할 수 없습니다(자동으로 비워짐).
      </div>
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
const LS_KEY = "JCCEI_PRESS_MVP_DATA_V8";
const LS_SETTINGS = "JCCEI_PRESS_MVP_SETTINGS_V8";

/** 승인관리자 목록(공통) */
const APPROVER_LIST = [
  "이재형 본부장",
  "이경호 본부장",
  "김희정 본부장",
  "이한솔 팀장",
  "고덕훈 팀장",
  "이병선 대표"
];

/** ✅ 설정: 관리자별 패스코드 */
const DEFAULT_SETTINGS = {
  adminCodes: {
    "이재형 본부장": "admin1234",
    "이경호 본부장": "admin1234",
    "김희정 본부장": "admin1234",
    "이한솔 팀장": "admin1234",
    "고덕훈 팀장": "admin1234",
    "이병선 대표": "admin1234",
  },
  holidays: ["2026-01-01","2026-02-09","2026-02-10","2026-02-11"]
};

function loadSettings(){
  try{
    const s = JSON.parse(localStorage.getItem(LS_SETTINGS) || "null");
    if(!s) return structuredClone(DEFAULT_SETTINGS);

    const adminCodes = (s.adminCodes && typeof s.adminCodes === "object") ? s.adminCodes : {};
    const mergedCodes = {};
    APPROVER_LIST.forEach(name=>{
      mergedCodes[name] = (adminCodes[name] || DEFAULT_SETTINGS.adminCodes[name] || "admin1234");
    });

    return {
      adminCodes: mergedCodes,
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
    d.press.forEach(p=>{ if(!Array.isArray(p.editHistory)) p.editHistory = []; });
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

/** 영업일 계산(주말/공휴일 제외) */
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
  return addBusinessDays(ymd(new Date()), 3, settings);
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
const btnDoSearch = el("btnDoSearch");
const btnClearSearch = el("btnClearSearch");

const exportFrom = el("exportFrom");
const exportTo = el("exportTo");
const btnExportXlsx = el("btnExportXlsx");

const btnInsertTips = el("btnInsertTips");
const imgHelp = el("imgHelp");

const pendingList = el("pendingList");
const approvedList = el("approvedList");
const kakaoText = el("kakaoText");
const btnCopyKakao = el("btnCopyKakao");
const copyHint = el("copyHint");

const adminWho = el("adminWho");
const adminPass = el("adminPass");
const adminCodesBox = el("adminCodesBox");

const setHolidays = el("setHolidays");
const btnSaveSettings = el("btnSaveSettings");
const settingsHint = el("settingsHint");

const btnResetAdmin = el("btnResetAdmin");

const prevMonth = el("prevMonth");
const nextMonth = el("nextMonth");

const toast = el("toast");
const calendarDetails = el("calendarDetails");
const btnOpenCalendar = el("btnOpenCalendar");

const guideBody = el("guideBody");
const btnToggleGuide = el("btnToggleGuide");

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

/** ✅ 필수/에러 유틸 */
function setFieldError(inputEl, message){
  inputEl.classList.add("inputError");
  const errEl = document.getElementById("err_" + inputEl.id);
  if(errEl) errEl.textContent = message || "입력이 필요합니다.";
}
function clearFieldError(inputEl){
  inputEl.classList.remove("inputError");
  const errEl = document.getElementById("err_" + inputEl.id);
  if(errEl) errEl.textContent = "";
}
function requireValue(inputEl, message){
  const v = (inputEl.value || "").trim();
  if(!v){
    setFieldError(inputEl, message || "입력이 필요합니다.");
    return false;
  }
  clearFieldError(inputEl);
  return true;
}
function bindLiveValidation(){
  const requiredFields = [staffName, staffPhone, staffEmail, approver, title, body, desiredDate];
  requiredFields.forEach(f=>{
    f.addEventListener("blur", ()=> requireValue(f));
    f.addEventListener("input", ()=> { if((f.value||"").trim()) clearFieldError(f); });
    f.addEventListener("change", ()=> { if((f.value||"").trim()) clearFieldError(f); });
  });
}
bindLiveValidation();

/** 탭 전환 */
function activateTab(view){
  tabs.forEach(x=>x.classList.remove("active"));
  document.querySelector(`.tab[data-view="${view}"]`)?.classList.add("active");
  viewStaff.classList.toggle("hidden", view!=="staff");
  viewAdmin.classList.toggle("hidden", view!=="admin");
  viewSettings.classList.toggle("hidden", view!=="settings");
  // ✅ 승인 탭으로 이동 시, 리스트를 현재 관리자 기준으로 다시 그리기
  if(view==="admin") renderLists();
}
tabs.forEach(t=>{
  t.addEventListener("click", ()=>{
    const v = t.getAttribute("data-view");
    activateTab(v);
  });
});

/** ✅ 승인관리자 셀렉트 옵션 렌더 */
function renderApproverSelects(){
  approver.innerHTML = `<option value="">선택하세요</option>` + APPROVER_LIST.map(n=>`<option>${escapeHtml(n)}</option>`).join("");
  adminWho.innerHTML = `<option value="">선택하세요</option>` + APPROVER_LIST.map(n=>`<option>${escapeHtml(n)}</option>`).join("");
}
renderApproverSelects();

/** ✅ 선택한 관리자 힌트(테스트 편의) */
function setHints(){
  const who = (adminWho.value || "").trim();
  adminCodeHint.textContent = who ? (settings.adminCodes[who] || "-") : "-";
}
adminWho.addEventListener("change", setHints);
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

/** ✅ 희망일 즉시 검증 */
function validateDesiredDateImmediate(inputEl, ymdStr){
  if(!ymdStr) return true;

  const today = ymd(new Date());
  if(ymdStr < today){
    inputEl.value = "";
    showToast(`${ymdStr} : 선택 불가\n사유: 지난 날짜입니다.`);
    return false;
  }

  if(!validateDesiredDateBusinessRule(inputEl, ymdStr, settings)) return false;

  const chk = checkPublishable(ymdStr, data, settings);
  if(!chk.ok){
    inputEl.value = "";
    showToast(`${ymdStr} : 배포 불가\n사유: ${chk.reason}`);
    return false;
  }

  if(isDesiredDateBlockedByApproved(ymdStr, data)){
    inputEl.value = "";
    showToast(`${ymdStr} : 배포 불가\n사유: 이미 승인된 보도자료가 있는 날짜(1일 1개)`);
    return false;
  }
  return true;
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

  const todayStr = ymd(new Date());
  const minSubmitStr = earliestDesiredYmd(settings);

  cal.innerHTML = "";
  cells.forEach(c=>{
    const dstr = ymd(c.date);
    const isPast = dstr < todayStr;
    const isTooSoon = dstr < minSubmitStr;

    const approved = data.press.find(p=>p.status==="APPROVED" && p.approvedDate===dstr);
    const chk = checkPublishable(dstr, data, settings);

    let badgeClass = "ok";
    let badgeText = "가능";

    if(approved){
      badgeClass = "approved";
      badgeText = "승인";
    }else if(isPast){
      badgeClass = "bad";
      badgeText = "불가";
    }else if(isTooSoon){
      badgeClass = "bad";
      badgeText = "불가";
    }else if(!chk.ok){
      badgeClass = "bad";
      badgeText = "불가";
    }

    const out = !c.inMonth ? "out" : "";
    const dayDiv = document.createElement("div");
    dayDiv.className = `day ${out} ${isPast ? "disabled" : ""}`;
    dayDiv.innerHTML = `
      <div class="n">${c.date.getDate()}</div>
      <span class="badge ${badgeClass}" data-date="${dstr}" data-type="${badgeText}">[${badgeText}]</span>
    `;

    const badge = dayDiv.querySelector(".badge");

    badge.addEventListener("click", (e)=>{
      e.stopPropagation();
      const type = badge.getAttribute("data-type");
      const dateStr = badge.getAttribute("data-date");

      if(dateStr < minSubmitStr && !approved){
        showToast(`${dateStr} : 신청/배포 불가\n사유: 신청일(오늘) 기준 주말/공휴일 제외 3영업일 이전`);
        flash(badge, "red");
        renderApprovedTitlesForDate(dateStr);
        desiredDate.value = dateStr;
        validateDesiredDateImmediate(desiredDate, dateStr);
        return;
      }

      if(type === "가능"){
        showToast(`${dateStr} : 배포 가능합니다.`);
        flash(badge, "green");
        renderApprovedTitlesForDate(dateStr);
        desiredDate.value = dateStr;
        validateDesiredDateImmediate(desiredDate, dateStr);
        calendarDetails.open = false;
        return;
      }

      if(type === "불가"){
        if(dateStr < todayStr){
          showToast(`${dateStr} : 선택 불가\n사유: 지난 날짜입니다.`);
          flash(badge, "red");
          renderApprovedTitlesForDate(dateStr);
          return;
        }
        const r = checkPublishable(dateStr, data, settings);
        const baseReason = r.ok ? "신청 조건 미충족" : r.reason;
        showToast(`${dateStr} : 배포 불가\n사유: ${baseReason}`);
        flash(badge, "red");
        renderApprovedTitlesForDate(dateStr);
        desiredDate.value = dateStr;
        validateDesiredDateImmediate(desiredDate, dateStr);
        return;
      }

      if(type === "승인"){
        renderApprovedTitlesForDate(dateStr);
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

/** 상태 라벨 */
function statusKorean(status){
  if(status==="APPROVED") return {label:"배포 예정", cls:"approved"};
  if(status==="SUBMITTED") return {label:"대기중", cls:"pending"};
  if(status==="REJECTED") return {label:"반려", cls:"rejected"};
  return {label:"임시", cls:"pending"};
}

/** ✅ 현재 로그인 관리자(선택) */
function currentAdmin(){
  return (adminWho.value || "").trim();
}

/** 리스트/표 렌더 */
function renderLists(){
  const name = staffName.value.trim();
  const mine = name ? data.press.filter(p => p.authorName === name).sort((a,b)=>b.createdAt-a.createdAt) : [];
  myList.innerHTML = mine.length ? mine.map(p => pressCard(p, {admin:false, mine:true})).join("") : `<div class="muted">이름을 입력하면 내 신청 목록이 보입니다.</div>`;

  // ✅ 관리자: 본인을 승인관리자로 지정한 건만
  const who = currentAdmin();
  const adminScope = who ? (p)=> p.approver === who : ()=>false;

  const pending = data.press.filter(p => p.status==="SUBMITTED").filter(adminScope).sort((a,b)=>b.createdAt-a.createdAt);
  const approved = data.press.filter(p => p.status==="APPROVED").filter(adminScope).sort((a,b)=> (a.approvedDate||"").localeCompare(b.approvedDate||""));
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
    createdYmd,
    p.approvedDate || ""
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
    boardTableBody.innerHTML = `<tr><td colspan="5" class="muted">검색 결과가 없습니다.</td></tr>`;
    return;
  }

  boardTableBody.innerHTML = rows.map(p=>{
    const st = statusKorean(p.status);
    return `
      <tr>
        <td><span class="kstatus ${st.cls}">${st.label}</span></td>
        <td>${escapeHtml(p.title)}</td>
        <td>${escapeHtml(p.desiredDate || "-")}</td>
        <td>${escapeHtml(p.authorName || "-")}</td>
        <td>${p.status==="APPROVED" ? `<button class="btn small" type="button" data-act="downloadDoc" data-id="${p.id}">다운로드</button>` : `<span class="muted">-</span>`}</td>
      </tr>
    `;
  }).join("");

  bindBoardActions();
}

/** 다운로드: DOC(워드 호환) - 요구사항 반영
 * - 승인관리자 표시 제거
 * - 사진은 실제 이미지 대신 "보도용 사진 n장 별첨" 문구만 표기
 */
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

  const authorLine = `${p.authorName || "-"}${p.authorPhone ? `(${p.authorPhone})` : ""}`;
  const imgCount = (p.images && p.images.length) ? p.images.length : 0;
  const imgLine = imgCount > 0 ? `보도용 사진 ${imgCount}장 별첨` : "";

  const linkHtml = (p.bigFileLinks && String(p.bigFileLinks).trim())
    ? `<h3>첨부 링크</h3><div style="font-size:14px;line-height:1.6;">${nl2br(p.bigFileLinks)}</div>`
    : "";

  const html = `<!doctype html>
<html><head><meta charset="utf-8"><title>${escapeHtml(p.title)}</title></head>
<body style="font-family:'Noto Sans KR',Arial,sans-serif; line-height:1.6;">
  <div style="font-size:14px; margin-bottom:12px;">
    <div><b>발송기관</b> : 제주창조경제혁신센터</div>
    <div><b>작성자</b> : ${escapeHtml(authorLine)}</div>
  </div>
  <h1 style="margin:0 0 8px;">${escapeHtml(p.title)}</h1>
  ${p.subtitle ? `<h2 style="margin:0 0 14px;font-size:16px;color:#334155;">${escapeHtml(p.subtitle)}</h2>` : ""}
  <div style="font-size:12px;color:#64748b;margin-bottom:14px;">
    배포 희망일: ${escapeHtml(p.desiredDate||"-")}<br/>
    이메일: ${escapeHtml(p.authorEmail||"-")}<br/>
    ${imgLine ? `${escapeHtml(imgLine)}<br/>` : ``}
  </div>
  <hr style="border:none;border-top:1px solid #e2e8f0;margin:14px 0;"/>
  <div style="font-size:14px;">${nl2br(p.body)}</div>
  ${linkHtml}
</body></html>`;

  const blob = new Blob([html], {type: "application/msword;charset=utf-8"});
  const url = URL.createObjectURL(blob);
  const a = document.createElement("a");
  a.href = url;
  a.download = sanitizeFilename(`${p.title || "보도자료"}_${p.desiredDate || ""}.doc`);
  document.body.appendChild(a);
  a.click();
  a.remove();
  setTimeout(()=> URL.revokeObjectURL(url), 1000);
}

/** 배포 예정/대기 현황: 다운로드 버튼 */
function bindBoardActions(){
  document.querySelectorAll('[data-act="downloadDoc"]').forEach(btn=>{
    btn.onclick = ()=> downloadPressAsDoc(btn.getAttribute("data-id"));
  });
}

/** 변경 기록 */
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
function formatEditHistory(p){
  const h = Array.isArray(p.editHistory) ? p.editHistory : [];
  if(h.length===0) return `<div class="muted">변경 내역이 없습니다.</div>`;

  const items = h.slice().sort((a,b)=>(b.at||0)-(a.at||0)).slice(0,6);
  return items.map(e=>{
    const who = e.by === "admin" ? "관리자" : "신청자";
    const when = e.at ? new Date(e.at).toLocaleString("ko-KR") : "-";
    const changes = e.changes || {};
    const keys = Object.keys(changes);
    const fieldsKor = { title:"제목", subtitle:"부제목", body:"본문", desiredDate:"희망일", bigFileLinks:"대용량 링크" };

    const list = keys.map(k=>{
      const from = (changes[k]?.from ?? "");
      const to = (changes[k]?.to ?? "");
      if(k === "body"){
        const diff = highlightBodyDiff(String(from).slice(0,2000) || "", String(to).slice(0,2000) || "");
        return `
          <details style="margin-top:6px;">
            <summary class="summaryBtn">본문 변경(전/후)</summary>
            <div class="two" style="margin-top:10px;">
              <div>
                <div class="small" style="margin-bottom:6px;">변경 전</div>
                <div class="diffBox">${diff.beforeHtml}</div>
              </div>
              <div>
                <div class="small" style="margin-bottom:6px;">변경 후</div>
                <div class="diffBox">${diff.afterHtml}</div>
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

  // ✅ 관리자 첨삭 버튼: 대기중 + 본인 승인관리자만(렌더 단계에서 이미 필터링, 추가 안전)
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
            작성자: <b>${escapeHtml(p.authorName)}</b> ·
            희망: <b>${escapeHtml(desired)}</b> ·
            이메일: <b>${escapeHtml(p.authorEmail||"-")}</b> ·
            승인 관리자: <b>${escapeHtml(p.approver||"-")}</b>
          </div>
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

/** 카드 버튼 바인딩 */
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

/** 관리자 가드(관리자별 코드) */
function getAdminInput(id, act){
  const elx = document.querySelector(`[data-act="${act}"][data-id="${id}"]`);
  return elx ? elx.value : "";
}
function adminGuard(){
  // ✅ 필수 체크
  clearFieldError(adminWho);
  clearFieldError(adminPass);

  const who = (adminWho.value || "").trim();
  const pass = (adminPass.value || "").trim();
  let ok = true;
  ok = requireValue(adminWho, "내 관리자 이름을 선택해주세요.") && ok;
  ok = requireValue(adminPass, "패스코드를 입력해주세요.") && ok;
  if(!ok) return false;

  const expected = settings.adminCodes[who];
  if(pass !== expected){
    alert("관리자 패스코드가 올바르지 않습니다.");
    return false;
  }
  return true;
}
function isAdminScopePress(p){
  const who = currentAdmin();
  return who && p.approver === who;
}

/** 관리자 첨삭 모달 */
function openAdminEdit(id){
  if(!adminGuard()) return;
  const p = data.press.find(x=>x.id===id);
  if(!p){ alert("대상을 찾을 수 없습니다."); return; }
  if(!isAdminScopePress(p)){
    alert("본인이 '승인 관리자'로 지정된 건만 첨삭할 수 있습니다.");
    return;
  }
  if(p.status !== "SUBMITTED"){
    alert("대기중(접수) 상태에서만 첨삭할 수 있습니다.");
    return;
  }

  editingAdminId = id;
  aEditTitle.value = p.title || "";
  aEditSubtitle.value = p.subtitle || "";
  aEditBody.value = p.body || "";
  aEditDesiredDate.value = p.desiredDate || "";
  aLastDiff.textContent = "아직 변경 내역이 없습니다.";
  dlgEditAdmin.showModal();
}
function adminEditSave(){
  if(!adminGuard()) return;
  const id = editingAdminId;
  const p = data.press.find(x=>x.id===id);
  if(!p) return;
  if(!isAdminScopePress(p)){
    alert("본인이 '승인 관리자'로 지정된 건만 첨삭할 수 있습니다.");
    return;
  }

  const dd = aEditDesiredDate.value || "";
  if(dd && isDesiredDateBlockedByApproved(dd, data)){
    aEditDesiredDate.value = "";
    showToast(`${dd} : 배포 불가\n사유: 이미 승인된 보도자료가 있는 날짜(1일 1개)`);
    return;
  }

  // ✅ 관리자 첨삭에서 '대용량 링크'는 수정 불가(삭제 요구사항)
  const before = {
    title: p.title || "",
    subtitle: p.subtitle || "",
    body: p.body || "",
    desiredDate: p.desiredDate || ""
  };
  const after = {
    title: aEditTitle.value.trim(),
    subtitle: aEditSubtitle.value.trim(),
    body: aEditBody.value.trim(),
    desiredDate: aEditDesiredDate.value || ""
  };

  const changes = diffChanges(
    { ...before, bigFileLinks: p.bigFileLinks || "" },
    { ...after,  bigFileLinks: p.bigFileLinks || "" } // 링크 변경 없음
  );

  pushHistory(p, "admin", changes);

  p.title = after.title;
  p.subtitle = after.subtitle || null;
  p.body = after.body;
  p.desiredDate = after.desiredDate || null;

  saveData(data);
  renderCalendar();
  renderLists();

  const keys = Object.keys(changes);
  if(keys.length===0){
    aLastDiff.textContent = "변경된 내용이 없습니다.";
  }else{
    const lines = keys.map(k=>{
      if(k==="body") return `- 본문: (변경됨)`;
      const kor = ({title:"제목",subtitle:"부제목",desiredDate:"희망일"})[k] || k;
      return `- ${kor}: "${String(changes[k].from)}" → "${String(changes[k].to)}"`;
    });
    aLastDiff.textContent = `저장 완료!\n${lines.join("\n")}`;
  }

  kakaoText.value =
`[제주창조경제혁신센터] 보도자료 첨삭 완료 안내
- 제목: ${p.title}
- 상태: 대기중(접수)
※ ‘내 신청 목록’에서 “변경 내역 보기”를 누르면 수정된 부분(전/후)을 확인할 수 있습니다.`;

  // ✅ 요구사항: 첨삭 저장 시 "저장 완료" 안내 + 자동 닫기
  showToast("첨삭 저장 완료");
  dlgEditAdmin.close();
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
  dlgEditUser.showModal();
}
function userEditSave(){
  const id = editingUserId;
  const name = staffName.value.trim();
  const p = data.press.find(x=>x.id===id);
  if(!p || p.authorName !== name) return;

  const dd = uEditDesiredDate.value || "";
  if(dd && isDesiredDateBlockedByApproved(dd, data)){
    uEditDesiredDate.value = "";
    showToast(`${dd} : 배포 불가\n사유: 이미 승인된 보도자료가 있는 날짜(1일 1개)`);
    return;
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
  renderCalendar();
  renderLists();

  dlgEditUser.close();
  showToast("수정 저장 완료");
}

/** 승인/반려 */
function adminApprove(id){
  if(!adminGuard()) return;

  const pr = data.press.find(x=>x.id===id);
  if(!pr){ alert("대상을 찾을 수 없습니다."); return; }
  if(!isAdminScopePress(pr)){
    alert("본인이 '승인 관리자'로 지정된 건만 승인할 수 있습니다.");
    return;
  }

  const date = getAdminInput(id, "approveDate") || "";
  const target = date || pr.desiredDate;

  if(!target){
    alert("승인 배포일 또는 희망일이 필요합니다.");
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

  const pr = data.press.find(x=>x.id===id);
  if(!pr){ alert("대상을 찾을 수 없습니다."); return; }
  if(!isAdminScopePress(pr)){
    alert("본인이 '승인 관리자'로 지정된 건만 반려할 수 있습니다.");
    return;
  }

  const reason = getAdminInput(id, "rejectReason") || "반려";
  pr.status = "REJECTED";
  pr.rejectReason = reason;
  pr.approvedDate = null;
  pr.approvedAt = null;

  saveData(data);
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

/** 희망일 입력 즉시 검증 */
desiredDate.addEventListener("change", ()=>{
  const v = desiredDate.value || "";
  if(!v) return;
  const ok = validateDesiredDateImmediate(desiredDate, v);
  if(!ok) setFieldError(desiredDate, "선택한 날짜는 신청/배포가 불가합니다.");
});

/** 신청 제출 */
formSubmit.addEventListener("submit", (e)=>{
  e.preventDefault();

  [staffName, staffPhone, staffEmail, approver, title, body, desiredDate, bigFileLinks].forEach(clearFieldError);

  let ok = true;
  ok = requireValue(staffName, "이름을 입력해주세요.") && ok;
  ok = requireValue(staffPhone, "연락처를 입력해주세요.") && ok;
  ok = requireValue(staffEmail, "이메일을 입력해주세요.") && ok;
  ok = requireValue(approver, "승인 관리자를 선택해주세요.") && ok;
  ok = requireValue(title, "제목을 입력해주세요.") && ok;
  ok = requireValue(body, "본문을 입력해주세요.") && ok;
  ok = requireValue(desiredDate, "배포 희망일을 선택해주세요.") && ok;

  if(!ok){
    showToast("필수 입력사항을 확인해주세요.");
    return;
  }

  if(!validateDesiredDateImmediate(desiredDate, desiredDate.value)){
    setFieldError(desiredDate, "선택한 날짜는 신청/배포가 불가합니다.");
    return;
  }

  const linkText = (bigFileLinks.value || "").trim();
  if(selectedFiles.length === 0 && !linkText){
    setFieldError(bigFileLinks, "사진이 없으면 대용량 링크는 필수입니다.");
    showToast("사진 또는 대용량 링크가 필요합니다.");
    return;
  }

  const pr = {
    id: cryptoRandomId(),
    authorName: staffName.value.trim(),
    authorPhone: staffPhone.value.trim(),
    authorEmail: staffEmail.value.trim(),
    approver: approver.value.trim(),
    title: title.value.trim(),
    subtitle: subtitle.value.trim() || null,
    body: body.value.trim(),
    desiredDate: desiredDate.value,
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
  desiredDate.value = "";
  bigFileLinks.value = "";
  selectedFiles = [];
  renderPreview();

  staffMsg.textContent = "신청 완료! 관리자 승인 대기중입니다.";
  staffMsg.style.borderColor = "#bbf7d0";

  renderCalendar();
  renderLists();
  showToast("신청 완료");
});

/** 내 신청 목록 리렌더 */
staffName.addEventListener("input", ()=> renderLists());

/** 검색(버튼 클릭 시 실행) */
btnDoSearch.addEventListener("click", ()=> renderBoardTable());
btnClearSearch.addEventListener("click", ()=>{
  boardSearch.value = "";
  renderBoardTable();
});

/** 검색 Enter로 실행 */
boardSearch.addEventListener("keydown", (e)=>{
  if(e.key === "Enter"){
    e.preventDefault();
    renderBoardTable();
  }
});

/** 캘린더 이동 */
prevMonth.onclick = ()=>{ cursor = new Date(cursor.getFullYear(), cursor.getMonth()-1, 1); renderCalendar(); };
nextMonth.onclick = ()=>{ cursor = new Date(cursor.getFullYear(), cursor.getMonth()+1, 1); renderCalendar(); };

/** ✅ 설정 UI 렌더(관리자별 코드 입력) */
function renderAdminCodesUI(){
  adminCodesBox.innerHTML = APPROVER_LIST.map(name=>{
    const v = settings.adminCodes[name] || "";
    return `
      <div class="item">
        <div class="row" style="justify-content:space-between;">
          <div class="t">${escapeHtml(name)}</div>
          <span class="pill">관리자</span>
        </div>
        <div style="margin-top:10px;">
          <label>
            패스코드
            <input type="text" data-admin-code="${escapeHtml(name)}" value="${escapeHtml(v)}" placeholder="예: admin1234">
          </label>
          <div class="small">※ 승인 탭에서 “내 관리자 이름” 선택 후 해당 패스코드로 로그인합니다.</div>
        </div>
      </div>
    `;
  }).join("");
}
function renderSettingsUI(){
  renderAdminCodesUI();
  setHolidays.value = settings.holidays.join("\n");
}
renderSettingsUI();

btnSaveSettings.onclick = ()=>{
  const newCodes = {};
  document.querySelectorAll("[data-admin-code]").forEach(inp=>{
    const name = inp.getAttribute("data-admin-code");
    const code = (inp.value || "").trim() || (DEFAULT_SETTINGS.adminCodes[name] || "admin1234");
    newCodes[name] = code;
  });

  const hs = setHolidays.value.split("\n").map(s=>s.trim()).filter(Boolean);

  settings = { adminCodes: newCodes, holidays: hs };
  saveSettings(settings);
  renderSettingsUI();
  setHints();

  settingsHint.textContent = "저장 완료!";
  setTimeout(()=> settingsHint.textContent="", 1500);

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
  adminWho.value = "";
  setHints();

  renderSettingsUI();
  renderCalendar();
  renderLists();
  approvedTitles.innerHTML = `<div class="muted">아직 선택된 날짜가 없습니다.</div>`;
  selectedFiles = [];
  renderPreview();
  showToast("초기화 완료");
});

/** ✅ 엑셀 내보내기(보드로 이동): 기간 내 "배포 예정(승인)"만 */
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

  const rows = data.press
    .filter(p=> p.status === "APPROVED" && p.approvedDate)
    .filter(p=>{
      const ad = parseYMD(p.approvedDate);
      return ad >= from && ad <= to;
    })
    .slice()
    .sort((a,b)=> (a.approvedDate||"").localeCompare(b.approvedDate||""));

  if(rows.length === 0){
    alert("해당 기간에 배포 예정(승인) 보도자료가 없습니다.");
    return;
  }

  const aoa = [];
  aoa.push([
    "배포일", "제목", "부제목", "작성자", "연락처", "이메일", "승인관리자",
    "희망일", "사진장수", "대용량 링크", "수정기록(건수)"
  ]);

  rows.forEach(p=>{
    aoa.push([
      p.approvedDate || "",
      p.title || "",
      p.subtitle || "",
      p.authorName || "",
      p.authorPhone || "",
      p.authorEmail || "",
      p.approver || "",
      p.desiredDate || "",
      (p.images && p.images.length) ? p.images.length : 0,
      (p.bigFileLinks || "").replace(/\n/g, " "),
      (p.editHistory && p.editHistory.length) ? p.editHistory.length : 0
    ]);
  });

  const ws = XLSX.utils.aoa_to_sheet(aoa);
  ws["!cols"] = [
    {wch:12},{wch:50},{wch:32},{wch:12},{wch:16},{wch:22},{wch:14},
    {wch:12},{wch:10},{wch:40},{wch:14}
  ];
  const wb = XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb, ws, "배포(승인)");

  const filename = `배포예정_보도자료_${fromStr}_~_${toStr}.xlsx`;
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

uEditDesiredDate.addEventListener("change", ()=>{
  const v = uEditDesiredDate.value || "";
  if(!v) return;
  if(isDesiredDateBlockedByApproved(v, data)){
    uEditDesiredDate.value = "";
    showToast(`${v} : 배포 불가\n사유: 이미 승인된 보도자료가 있는 날짜(1일 1개)`);
  }
});
aEditDesiredDate.addEventListener("change", ()=>{
  const v = aEditDesiredDate.value || "";
  if(!v) return;
  if(isDesiredDateBlockedByApproved(v, data)){
    aEditDesiredDate.value = "";
    showToast(`${v} : 배포 불가\n사유: 이미 승인된 보도자료가 있는 날짜(1일 1개)`);
  }
});

/** ✅ 캘린더 열기 버튼 */
btnOpenCalendar?.addEventListener("click", ()=>{
  calendarDetails.open = true;
  setTimeout(()=> calendarDetails.scrollIntoView({behavior:"smooth", block:"start"}), 80);
});

/** ✅ 안내 접기/펼치기 */
btnToggleGuide.addEventListener("click", ()=>{
  const hidden = guideBody.style.display === "none";
  guideBody.style.display = hidden ? "block" : "none";
  btnToggleGuide.textContent = hidden ? "접기" : "펼치기";
});

/** 승인 탭에서 관리자/패스 변경 시 즉시 리스트 갱신 */
adminWho.addEventListener("change", ()=>{ setHints(); renderLists(); });
adminPass.addEventListener("input", ()=>{ /* 입력 중엔 굳이 렌더 X */ });

/** 초기 렌더 */
renderCalendar();
renderLists();

/** 도우미 */
function cryptoRandomId(){
  const s = Array.from(crypto.getRandomValues(new Uint8Array(12))).map(b=>b.toString(16).padStart(2,"0")).join("");
  return "pr_" + s;
}
function escapeHtml(str){
  return String(str ?? "")
    .replaceAll("&","&amp;")
    .replaceAll("<","&lt;")
    .replaceAll(">","&gt;")
    .replaceAll('"',"&quot;")
    .replaceAll("'","&#039;");
}
</script>

</body>
</html>

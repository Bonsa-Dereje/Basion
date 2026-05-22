<script>
  import './calendarPage.css';

  // ── Calendar state ────────────────────────────────────
  let currentMonth = 8;
  let currentYear  = 2024;

  const monthNames = ['January','February','March','April','May','June',
                      'July','August','September','October','November','December'];
  const dayLabels  = ['SUN','MON','TUE','WED','THU','FRI','SAT'];

  function daysInMonth(m, y) { return new Date(y, m + 1, 0).getDate(); }
  function firstDayOf(m, y)  { return new Date(y, m, 1).getDay(); }

  $: totalDays = daysInMonth(currentMonth, currentYear);
  $: startDay  = firstDayOf(currentMonth, currentYear);
  $: prevTotal = daysInMonth(currentMonth === 0 ? 11 : currentMonth - 1, currentMonth === 0 ? currentYear - 1 : currentYear);

  $: calCells = (() => {
    const cells = [];
    for (let i = 0; i < startDay; i++)
      cells.push({ day: prevTotal - startDay + 1 + i, overflow: true });
    for (let i = 1; i <= totalDays; i++)
      cells.push({ day: i, overflow: false });
    while (cells.length % 7 !== 0)
      cells.push({ day: cells.length - totalDays - startDay + 1, overflow: true });
    return cells;
  })();

  function prevMonth() {
    if (currentMonth === 0) { currentMonth = 11; currentYear--; }
    else currentMonth--;
  }
  function nextMonth() {
    if (currentMonth === 11) { currentMonth = 0; currentYear++; }
    else currentMonth++;
  }
  function goToday() { currentMonth = 8; currentYear = 2024; }

  // ── Calendar events ───────────────────────────────────
  let calEvents = [
    { day: 28, month: 7, id: '442', title: 'Content Shoot',   time: '10:00' },
    { day: 30, month: 7, id: '891', title: 'A-Roll Edit',      time: '14:00' },
    { day: 1,  month: 8, id: '212', title: 'B-Roll Selection', time: null    },
    { day: 5,  month: 8,             title: 'Client Approval',  time: null, tag: 'ST-V1' },
    { day: 6,  month: 8,             title: 'Scheduling',       time: '09:00', status: 'QUEUED', auto: false },
    { day: 7,  month: 8,             title: 'Post Live',        time: '12:00', status: 'AUTO',   auto: true  },
  ];

  function eventsFor(day, overflow) {
    if (overflow) return [];
    return calEvents.filter(e => e.day === day && (e.month === undefined || e.month === currentMonth));
  }

  // ── Client tabs ───────────────────────────────────────
  let activeClient = 'STARLIGHT MEDIA';
  const clients = ['STARLIGHT MEDIA', 'TECHNOVA', 'AURA LABS'];

  // ── View toggle ───────────────────────────────────────
  let activeView = 'MONTH';
  const views = ['YEAR', 'MONTH', 'WEEK', 'DAY'];

  // ── Pipeline groups ───────────────────────────────────
  // Each group is a self-contained enclosed node with subtasks inside.
  // Groups are draggable as a unit. Subtasks are draggable to the calendar.
  let groups = [
    {
      id: 'script',
      label: 'Script',
      phase: 'Phase 01',
      x: 30,
      y: 28,
      subtasks: [
        { id: 'brief',      label: 'Creative Brief',  detail: 'Lock concept',    done: false },
        { id: 'research',   label: 'Research',        detail: 'Gather refs',     done: true  },
        { id: 'firstdraft', label: 'First Draft',     detail: 'V1 script',       done: true  },
        { id: 'revisions',  label: 'Revisions',       detail: 'Client amends',   done: false },
      ],
    },
    {
      id: 'shoot',
      label: 'Shoot',
      phase: 'Phase 02',
      x: 320,
      y: 28,
      subtasks: [
        { id: 'rentcam',     label: 'Rent Camera',    detail: 'Book equipment',  done: false },
        { id: 'location',    label: 'Location Scout', detail: 'Confirm venue',   done: false },
        { id: 'lighting',    label: 'Lighting',       detail: 'Set & test rigs', done: false },
        { id: 'sendfootage', label: 'Send Footage',   detail: 'Upload to server',done: true  },
      ],
    },
    {
      id: 'edit',
      label: 'Edit',
      phase: 'Phase 03',
      x: 610,
      y: 28,
      active: true,
      subtasks: [
        { id: 'roughcut',    label: 'Rough Cut',      detail: 'First assembly',  done: true  },
        { id: 'sfx',         label: 'SFX',            detail: 'Score & effects', done: false },
        { id: 'soundcheck',  label: 'Sound Check',    detail: 'Mix & levels',    done: false },
        { id: 'colourgrade', label: 'Colour Grade',   detail: 'Grade pass',      done: false },
      ],
    },
    {
      id: 'post',
      label: 'Post',
      phase: 'Phase 04',
      x: 900,
      y: 28,
      subtasks: [
        { id: 'captions',   label: 'Captions',        detail: 'SRT + CC',        done: false },
        { id: 'thumbnail',  label: 'Thumbnail',       detail: 'Design & export', done: false },
        { id: 'subtitles',  label: 'Subtitles',       detail: 'Burn-in pass',    done: false },
        { id: 'schedule',   label: 'Schedule Post',   detail: 'Queue in planner',done: false },
      ],
    },
  ];

  // group dimensions
  const GW = 240; // group width
  const GH = 232; // group height (header 40 + 4 subtasks * 46 + 8 bottom pad)

  // ── Connection beziers between groups ─────────────────
  function groupRightX(g)  { return g.x + GW; }
  function groupCenterY(g) { return g.y + GH / 2; }
  function groupLeftX(g)   { return g.x; }

  function groupConnPath(a, b) {
    const ax = groupRightX(a), ay = groupCenterY(a);
    const bx = groupLeftX(b),  by = groupCenterY(b);
    const dx = (bx - ax) * 0.5;
    return `M ${ax} ${ay} C ${ax+dx} ${ay} ${bx-dx} ${by} ${bx} ${by}`;
  }

  $: groupConnections = groups.slice(0, -1).map((g, i) => [g, groups[i + 1]]);

  // ── Drag — group move ─────────────────────────────────
  let draggingGroup = null; // { id, offsetX, offsetY }

  function onGroupHeaderDown(e, gid) {
    if (e.target.closest('.subtask-chip')) return;
    e.preventDefault();
    const g = groups.find(g => g.id === gid);
    draggingGroup = { id: gid, offsetX: e.clientX - g.x, offsetY: e.clientY - g.y };
  }

  function onGraphMouseMove(e) {
    if (!draggingGroup) return;
    const rect = e.currentTarget.getBoundingClientRect();
    groups = groups.map(g => g.id !== draggingGroup.id ? g : {
      ...g,
      x: Math.max(0, Math.min(rect.width  - GW, e.clientX - draggingGroup.offsetX - rect.left)),
      y: Math.max(0, Math.min(rect.height - GH, e.clientY - draggingGroup.offsetY - rect.top)),
    });
  }

  function onGraphMouseUp() { draggingGroup = null; }

  // ── Drag — subtask to calendar ────────────────────────
  let draggingTask = null; // subtask id
  let dragOver     = null; // calendar day

  function onSubtaskDragStart(e, sid) {
    e.dataTransfer.setData('text/plain', sid);
    draggingTask = sid;
  }
  function onSubtaskDragEnd()          { draggingTask = null; }
  function onCalDragOver(e, day)       { e.preventDefault(); dragOver = day; }
  function onCalDragLeave()            { dragOver = null; }
  function onCalDrop(e, day) {
    e.preventDefault();
    if (!draggingTask) return;
    // find the subtask label
    let taskLabel = '';
    groups.forEach(g => {
      const s = g.subtasks.find(s => s.id === draggingTask);
      if (s) taskLabel = s.label;
    });
    // mark done + add to calendar
    groups = groups.map(g => ({
      ...g,
      subtasks: g.subtasks.map(s => s.id === draggingTask ? { ...s, done: true } : s),
    }));
    calEvents = [...calEvents, {
      day:   day,
      month: currentMonth,
      title: taskLabel,
      time:  null,
      tag:   null,
      dropped: true,
    }];
    draggingTask = null;
    dragOver = null;
  }

  function icFile() {
    return `<svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/><polyline points="14 2 14 8 20 8"/></svg>`;
  }
  function icInbox() {
    return `<svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"><polyline points="22 12 16 12 14 15 10 15 8 12 2 12"/><path d="M5.45 5.11L2 12v6a2 2 0 0 0 2 2h16a2 2 0 0 0 2-2v-6l-3.45-6.89A2 2 0 0 0 16.76 4H7.24a2 2 0 0 0-1.79 1.11z"/></svg>`;
  }
  function icCheck() {
    return `<svg width="9" height="9" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><polyline points="20 6 9 17 4 12"/></svg>`;
  }
  function icPlus() {
    return `<svg width="17" height="17" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round"><line x1="12" y1="5" x2="12" y2="19"/><line x1="5" y1="12" x2="19" y2="12"/></svg>`;
  }
</script>

<svelte:window on:mouseup={onGraphMouseUp} />

<div class="cal-page">

  <!-- ── Calendar grid ────────────────────────────────── -->
  <div class="calendar-area">
    <div class="day-labels">
      {#each dayLabels as d}
        <div class="day-label">{d}</div>
      {/each}
    </div>

    <div class="cal-grid">
      {#each calCells as cell}
        {@const isToday = !cell.overflow && cell.day === 1 && currentMonth === 8}
        {@const cellEvents = eventsFor(cell.day, cell.overflow)}
        <div
          class="cal-cell {cell.overflow ? 'overflow' : ''} {isToday ? 'today' : ''} {dragOver === cell.day && !cell.overflow ? 'drag-over' : ''}"
          on:dragover={e => !cell.overflow && onCalDragOver(e, cell.day)}
          on:dragleave={onCalDragLeave}
          on:drop={e => !cell.overflow && onCalDrop(e, cell.day)}
        >
          <span class="cell-num {isToday ? 'today-num' : ''}">{cell.day}</span>
          {#each cellEvents as ev}
            <div class="cal-event {ev.status ? (ev.auto ? 'ev-auto' : 'ev-queued') : ''} {ev.dropped ? 'ev-dropped' : ''}">
              {#if ev.id}<span class="ev-meta">ID: {ev.id}</span>{/if}
              {#if ev.tag}<span class="ev-meta">{ev.tag}</span>{/if}
              {#if ev.status}<span class="ev-status-label">{ev.status}</span>{/if}
              <span class="ev-title">{ev.title}</span>
              {#if ev.time}<span class="ev-time">{ev.time}</span>{/if}
            </div>
          {/each}
        </div>
      {/each}
    </div>
  </div>

  <!-- ── Bottom strip ──────────────────────────────────── -->
  <div class="bottom-strip">

    <!-- Toolbar -->
    <div class="strip-toolbar">
      <div class="client-tabs">
        <span class="clients-label">CLIENTS:</span>
        {#each clients as c}
          <button
            class="client-tab-btn {activeClient === c ? 'active' : ''}"
            on:click={() => activeClient = c}
          >{c}</button>
        {/each}
      </div>
      <div class="strip-actions">
        <button class="strip-btn">{@html icFile()} TEMPLATES</button>
        <button class="strip-btn active-btn">{@html icInbox()} UNSCHEDULED TASKS</button>
      </div>
    </div>

    <!-- Node graph canvas -->
    <div
      class="node-graph"
      on:mousemove={onGraphMouseMove}
      on:mouseup={onGraphMouseUp}
    >
      <!-- SVG layer: hatching defs + connector lines -->
      <svg class="graph-svg" xmlns="http://www.w3.org/2000/svg">
        <defs>
          <!-- fine diagonal hatch -->
          <pattern id="hatch-fine" x="0" y="0" width="10" height="10" patternUnits="userSpaceOnUse" patternTransform="rotate(45)">
            <line x1="0" y1="0" x2="0" y2="10" stroke="#2c2e2f" stroke-width="1"/>
          </pattern>
          <!-- coarser hatch for active group -->
          <pattern id="hatch-active" x="0" y="0" width="8" height="8" patternUnits="userSpaceOnUse" patternTransform="rotate(45)">
            <line x1="0" y1="0" x2="0" y2="8" stroke="#3a3c3e" stroke-width="1.2"/>
          </pattern>
          <!-- cross-hatch for connector zones -->
          <pattern id="hatch-cross" x="0" y="0" width="12" height="12" patternUnits="userSpaceOnUse" patternTransform="rotate(0)">
            <line x1="0" y1="0" x2="0" y2="12" stroke="#252728" stroke-width="0.8"/>
            <line x1="0" y1="0" x2="12" y2="0" stroke="#252728" stroke-width="0.8"/>
          </pattern>

          <!-- glow filter for active group -->
          <filter id="node-glow" x="-20%" y="-20%" width="140%" height="140%">
            <feGaussianBlur stdDeviation="4" result="blur"/>
            <feMerge><feMergeNode in="blur"/><feMergeNode in="SourceGraphic"/></feMerge>
          </filter>
        </defs>

        <!-- hatched background panels for each connector gap -->
        {#each groupConnections as [a, b]}
          <rect
            x={groupRightX(a) + 2}
            y={a.y + 10}
            width={groupLeftX(b) - groupRightX(a) - 4}
            height={GH - 20}
            fill="url(#hatch-cross)"
            rx="4"
          />
        {/each}

        <!-- group enclosure background (hatched) -->
        {#each groups as g}
          <rect
            x={g.x} y={g.y}
            width={GW} height={GH}
            fill={g.active ? 'url(#hatch-active)' : 'url(#hatch-fine)'}
            rx="10"
            opacity="1"
          />
        {/each}

        <!-- connector pipes -->
        {#each groupConnections as [a, b]}
          {@const ay = groupCenterY(a)}
          {@const by = groupCenterY(b)}
          <!-- pipe track (thick faint) -->
          <path
            d={groupConnPath(a, b)}
            fill="none"
            stroke="#2a2c2d"
            stroke-width="14"
            stroke-linecap="round"
          />
          <!-- pipe center line -->
          <path
            d={groupConnPath(a, b)}
            fill="none"
            stroke="#404345"
            stroke-width="1.5"
          />
          <!-- animated flow dot -->
          <circle r="3" fill="#606466">
            <animateMotion
              dur="2.8s"
              repeatCount="indefinite"
              path={groupConnPath(a, b)}
            />
          </circle>
        {/each}

        <!-- port circles on group edges -->
        {#each groups as g, i}
          {#if i < groups.length - 1}
            <!-- right port -->
            <circle cx={groupRightX(g)} cy={groupCenterY(g)} r="5" fill="#1c1b1b" stroke="#4a4c4d" stroke-width="1.2"/>
            <circle cx={groupRightX(g)} cy={groupCenterY(g)} r="2.5" fill="#606466"/>
          {/if}
          {#if i > 0}
            <!-- left port -->
            <circle cx={groupLeftX(g)} cy={groupCenterY(g)} r="5" fill="#1c1b1b" stroke="#4a4c4d" stroke-width="1.2"/>
            <circle cx={groupLeftX(g)} cy={groupCenterY(g)} r="2.5" fill="#606466"/>
          {/if}
        {/each}
      </svg>

      <!-- Group enclosures (HTML overlay) — each wrapped in a major-task enclosure -->
      {#each groups as g}
        {@const gDone = g.subtasks.filter(s => s.done).length}
        {@const gTotal = g.subtasks.length}
        <!-- Major task enclosure -->
        <div
          class="major-task-enclosure {g.active ? 'active' : ''}"
          style="left:{g.x}px; top:{g.y}px; width:{GW}px;"
        >
          <!-- Enclosure label bar -->
          <div class="enclosure-label-bar">
            <span class="enclosure-phase">{g.phase}</span>
            <span class="enclosure-title">{g.label}</span>
            <span class="enclosure-count">{gDone}/{gTotal}</span>
          </div>

          <!-- Inner group node -->
          <div
            class="group-node {g.active ? 'active' : ''} {draggingGroup && draggingGroup.id === g.id ? 'dragging' : ''}"
            style="position:relative; left:auto; top:auto; width:100%; height:{GH}px;"
          >
            <!-- Header — drag handle -->
            <div
              class="group-header"
              on:mousedown={e => onGroupHeaderDown(e, g.id)}
            >
              <div class="group-header-left">
                <span class="group-phase">{g.phase}</span>
                <span class="group-label">{g.label}</span>
              </div>
              <div class="group-progress">
                <span class="group-progress-text">{gDone}/{gTotal}</span>
                <div class="group-progress-bar">
                  <div class="group-progress-fill" style="width:{(gDone/gTotal)*100}%"></div>
                </div>
              </div>
            </div>

            <!-- Subtask blocks -->
            <div class="group-subtasks">
              {#each g.subtasks as s}
                <div
                  class="subtask-chip {s.done ? 'done' : ''}"
                  draggable="true"
                  on:dragstart={e => onSubtaskDragStart(e, s.id)}
                  on:dragend={onSubtaskDragEnd}
                  title="Drag to calendar"
                >
                  <div class="chip-check {s.done ? 'checked' : ''}">
                    {#if s.done}{@html icCheck()}{/if}
                  </div>
                  <div class="chip-text">
                    <span class="chip-label">{s.label}</span>
                    <span class="chip-detail">{s.detail}</span>
                  </div>
                  <div class="chip-drag-handle">
                    <svg width="10" height="10" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round"><circle cx="9" cy="6" r="1.2" fill="currentColor"/><circle cx="15" cy="6" r="1.2" fill="currentColor"/><circle cx="9" cy="12" r="1.2" fill="currentColor"/><circle cx="15" cy="12" r="1.2" fill="currentColor"/><circle cx="9" cy="18" r="1.2" fill="currentColor"/><circle cx="15" cy="18" r="1.2" fill="currentColor"/></svg>
                  </div>
                </div>
              {/each}
            </div>
          </div>
        </div>
      {/each}

    </div><!-- /node-graph -->
  </div><!-- /bottom-strip -->

  <!-- View switcher -->
  <div class="view-switcher">
    {#each views as v}
      <button
        class="view-btn {activeView === v ? 'active' : ''}"
        on:click={() => activeView = v}
      >{v}</button>
    {/each}
  </div>

  <!-- FAB -->
  <button class="fab">{@html icPlus()}</button>

</div>
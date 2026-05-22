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
  // Coordinates are fixed: Script, Shoot, Edit, Post horizontally spaced.
  let groups = [
    {
      id: 'script',
      label: 'Script',
      phase: 'Phase 01',
      x: 25,
      y: 25,
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
      x: 285,
      y: 25,
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
      x: 545,
      y: 25,
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
      x: 805,
      y: 25,
      subtasks: [
        { id: 'captions',   label: 'Captions',        detail: 'SRT + CC',        done: false },
        { id: 'thumbnail',  label: 'Thumbnail',       detail: 'Design & export', done: false },
        { id: 'subtitles',  label: 'Subtitles',       detail: 'Burn-in pass',    done: false },
        { id: 'schedule',   label: 'Schedule Post',   detail: 'Queue in planner',done: false },
      ],
    },
    {
      id: 'delegate',
      label: 'Delegate',
      phase: 'Phase 05',
      x: 1065,
      y: 25,
      subtasks: [
        { id: 'sarah',      label: 'Sarah (Lead Editor)',  detail: 'Internal Employee',  delegated: false },
        { id: 'apex',       label: 'Apex Studio',          detail: 'Outsource Partner',  delegated: false },
        { id: 'clara',      label: 'Clara (Copywriter)',   detail: 'Freelance Outsource',delegated: false },
        { id: 'basion_ai',  label: 'Basion AI Agent',      detail: 'Autonomous Agent',   delegated: false },
      ],
      isDelegateNode: true
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

  // ── Custom Pop-Out Subtask Drag and Drop ──────────────────
  let activeDragTask = null; // subtask object
  let dragGroupId     = null; // group ID containing the subtask
  let dragMouseX      = 0;
  let dragMouseY      = 0;
  let showDragCard    = false;
  let dragStartPos    = { x: 0, y: 0 };
  let dragOver        = null; // calendar day cell hovered

  function startTaskDrag(e, subtask, groupId) {
    if (e.button !== 0) return; // Left click only
    if (e.target.closest('.chip-check')) return; // Avoid drag on checkbox

    activeDragTask = subtask;
    dragGroupId = groupId;
    dragMouseX = e.clientX;
    dragMouseY = e.clientY;
    dragStartPos = { x: e.clientX, y: e.clientY };
    showDragCard = false;
  }

  function onWindowMouseMove(e) {
    if (!activeDragTask) return;
    dragMouseX = e.clientX;
    dragMouseY = e.clientY;

    if (!showDragCard) {
      const dx = e.clientX - dragStartPos.x;
      const dy = e.clientY - dragStartPos.y;
      if (Math.sqrt(dx*dx + dy*dy) > 5) {
        showDragCard = true;
      }
    }
  }

  function onWindowMouseUp(e) {
    if (!activeDragTask) return;

    if (showDragCard && dragOver !== null) {
      // Mark subtask as done
      groups = groups.map(g => {
        if (g.id !== dragGroupId) return g;
        return {
          ...g,
          subtasks: g.subtasks.map(s => s.id === activeDragTask.id ? { ...s, done: true } : s),
        };
      });

      // Add to calendar
      calEvents = [...calEvents, {
        day:   dragOver,
        month: currentMonth,
        title: activeDragTask.label,
        time:  null,
        tag:   null,
        dropped: true,
      }];
    }

    activeDragTask = null;
    dragGroupId = null;
    showDragCard = false;
    dragOver = null;
  }

  function toggleSubtask(groupId, subtaskId) {
    groups = groups.map(g => {
      if (g.id !== groupId) return g;
      return {
        ...g,
        subtasks: g.subtasks.map(s => s.id === subtaskId ? { ...s, done: !s.done } : s),
      };
    });
  }

  // ── Delegation & Automate Linking State ──────────────────────
  let isDelegatingAnim = false;

  function toggleDelegate(id) {
    groups = groups.map(g => {
      if (g.id !== 'delegate') return g;
      return {
        ...g,
        subtasks: g.subtasks.map(s => {
          if (s.id !== id) return s;
          const nextState = !s.delegated;
          
          if (nextState) {
            // Trigger animation
            isDelegatingAnim = true;
            setTimeout(() => {
              isDelegatingAnim = false;
            }, 1500);

            // Add corresponding task to Automate panel
            let newTask = null;
            if (id === 'sarah') {
              newTask = {
                id: 'auto-del-sarah',
                client: 'STARLIGHT MEDIA',
                title: "Review Sarah's video edits",
                detail: 'Provide draft feedback on rough cut revisions V2',
                type: 'Call',
                phone: '+1 (555) 234-5678'
              };
            } else if (id === 'apex') {
              newTask = {
                id: 'auto-del-apex',
                client: 'TECHNOVA',
                title: 'Audit Apex Studio assets',
                detail: 'Confirm outsource quality benchmarks for Shoot',
                type: 'Checkup',
                taskUrl: '#checkup'
              };
            } else if (id === 'clara') {
              newTask = {
                id: 'auto-del-clara',
                client: 'TECHNOVA',
                title: "Check Clara's copywriting",
                detail: 'Audit captions & review brief script description',
                type: 'Checkup',
                taskUrl: '#checkup'
              };
            } else {
              newTask = {
                id: 'auto-del-ai',
                client: 'AURA LABS',
                title: 'AI Agent mix diagnostics',
                detail: 'Check automated sound check alignment via SMS',
                type: 'Text',
                phone: '+1 (555) 876-5432'
              };
            }

            // Put it at the front of upcomingTasks
            upcomingTasks = [newTask, ...upcomingTasks.filter(t => t.id !== newTask.id)];
          } else {
            // Remove from upcoming tasks if undelegated
            const targetId = `auto-del-${id}`;
            upcomingTasks = upcomingTasks.filter(t => t.id !== targetId);
          }
          
          return { ...s, delegated: nextState };
        })
      };
    });
  }

  // ── Automate Panel State ────────────────────────────────────
  let upcomingTasks = [
    {
      id: 'auto-1',
      client: 'STARLIGHT MEDIA',
      title: 'Call Liam regarding brief edits',
      detail: 'Confirm changes for Creative Brief V2',
      type: 'Call',
      phone: '+1 (555) 234-5678'
    },
    {
      id: 'auto-2',
      client: 'TECHNOVA',
      title: 'Send Shoot details to director',
      detail: 'SMS location, rent logs & equipment checklist',
      type: 'Text',
      phone: '+1 (555) 987-6543'
    },
    {
      id: 'auto-3',
      client: 'AURA LABS',
      title: 'Run Audio Levels Checkup',
      detail: 'Audit mix levels and SFX alignment',
      type: 'Checkup',
      taskUrl: '#checkup'
    },
    {
      id: 'auto-4',
      client: 'STARLIGHT MEDIA',
      title: 'Review A-Roll edits with editor',
      detail: 'Provide draft revisions schedule',
      type: 'Call',
      phone: '+1 (555) 123-4567'
    },
    {
      id: 'auto-5',
      client: 'AURA LABS',
      title: 'Request thumbnail draft upload',
      detail: 'SMS design team for raw assets',
      type: 'Text',
      phone: '+1 (555) 876-5432'
    }
  ];

  // Modals / Overlays state
  let activeModal = null; // 'call' | 'text' | 'checkup' | null
  let activeModalTask = null; // task object
  
  // Call state
  let callTimer = '00:00';
  let callInterval = null;
  let callSeconds = 0;
  
  // Text state
  let textMessage = '';
  
  // Checkup state
  let checkupProgress = 0;
  let checkupStage = '';
  let checkupTimer = null;

  function triggerAction(task, actionType) {
    activeModalTask = task;
    activeModal = actionType;
    
    if (actionType === 'call') {
      callSeconds = 0;
      callTimer = '00:00';
      clearInterval(callInterval);
      callInterval = setInterval(() => {
        callSeconds++;
        const mins = String(Math.floor(callSeconds / 60)).padStart(2, '0');
        const secs = String(callSeconds % 60).padStart(2, '0');
        callTimer = `${mins}:${secs}`;
      }, 1000);
    } else if (actionType === 'text') {
      textMessage = `Hey! Regarding "${task.title}", the latest files look good, let's proceed.`;
    } else if (actionType === 'checkup') {
      checkupProgress = 0;
      checkupStage = 'Initializing Audit Diagnostic...';
      runCheckupAnimation();
    }
  }

  function runCheckupAnimation() {
    const stages = [
      { p: 20, s: 'Verifying files and project tags...' },
      { p: 55, s: 'Checking database queues...' },
      { p: 85, s: 'Running automation triggers...' },
      { p: 100, s: 'Audit diagnostics clear!' }
    ];
    let index = 0;
    clearInterval(checkupTimer);
    checkupTimer = setInterval(() => {
      if (index < stages.length) {
        checkupProgress = stages[index].p;
        checkupStage = stages[index].s;
        index++;
      } else {
        clearInterval(checkupTimer);
      }
    }, 800);
  }

  function completeModalAction() {
    clearInterval(callInterval);
    clearInterval(checkupTimer);
    
    if (activeModalTask) {
      dismissCard(activeModalTask.id);
    }
    
    activeModal = null;
    activeModalTask = null;
  }

  function cancelModalAction() {
    clearInterval(callInterval);
    clearInterval(checkupTimer);
    activeModal = null;
    activeModalTask = null;
  }

  let dismissedCardIds = new Set();
  
  function dismissCard(id) {
    dismissedCardIds.add(id);
    dismissedCardIds = dismissedCardIds;
    
    setTimeout(() => {
      const taskIndex = upcomingTasks.findIndex(t => t.id === id);
      if (taskIndex !== -1) {
        const [removed] = upcomingTasks.splice(taskIndex, 1);
        upcomingTasks = [...upcomingTasks, removed]; // recycle card to bottom
      }
      dismissedCardIds.delete(id);
      dismissedCardIds = dismissedCardIds;
    }, 450);
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

<svelte:window on:mousemove={onWindowMouseMove} on:mouseup={onWindowMouseUp} />

<div class="cal-page">

  <!-- ── Top bar ──────────────────────────────────────── -->
  <header class="cal-topbar">
    <div class="topbar-left">
      <h1 class="month-title">{monthNames[currentMonth]}</h1>
      <div class="cal-nav">
        <button class="nav-btn" on:click={prevMonth}>
          <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"><polyline points="15 18 9 12 15 6"/></svg>
        </button>
        <button class="today-btn" on:click={goToday}>TODAY</button>
        <button class="nav-btn" on:click={nextMonth}>
          <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"><polyline points="9 18 15 12 9 6"/></svg>
        </button>
      </div>
    </div>

    <div class="topbar-right">
      <div class="search-bar">
        <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"><circle cx="11" cy="11" r="8"/><line x1="21" y1="21" x2="16.65" y2="16.65"/></svg>
        <input type="text" placeholder="Search tasks…" />
        <span class="search-kbd">⌘K</span>
      </div>
      <button class="icon-btn">
        <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"><path d="M18 8A6 6 0 0 0 6 8c0 7-3 9-3 9h18s-3-2-3-9"/><path d="M13.73 21a2 2 0 0 1-3.46 0"/></svg>
      </button>
      <button class="icon-btn">
        <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="3"/><path d="M19.4 15a1.65 1.65 0 0 0 .33 1.82l.06.06a2 2 0 0 1-2.83 2.83l-.06-.06a1.65 1.65 0 0 0-1.82-.33 1.65 1.65 0 0 0-1 1.51V21a2 2 0 0 1-4 0v-.09A1.65 1.65 0 0 0 9 19.4a1.65 1.65 0 0 0-1.82.33l-.06.06a2 2 0 0 1-2.83-2.83l.06.06A1.65 1.65 0 0 0 4.68 15a1.65 1.65 0 0 0-1.51-1H3a2 2 0 0 1 0-4h.09A1.65 1.65 0 0 0 4.6 9a1.65 1.65 0 0 0-.33-1.82l-.06-.06a2 2 0 0 1 2.83-2.83l.06.06A1.65 1.65 0 0 0 9 4.68a1.65 1.65 0 0 0 1-1.51V3a2 2 0 0 1 4 0v.09a1.65 1.65 0 0 0 1 1.51 1.65 1.65 0 0 0 1.82-.33l.06-.06a2 2 0 0 1 2.83 2.83l-.06.06A1.65 1.65 0 0 0 19.4 9a1.65 1.65 0 0 0 1.51 1H21a2 2 0 0 1 0 4h-.09a1.65 1.65 0 0 0-1.51 1z"/></svg>
      </button>
      <div class="avatar-ring">CF</div>
    </div>
  </header>

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
          on:mouseenter={() => { if (activeDragTask && !cell.overflow) dragOver = cell.day; }}
          on:mouseleave={() => { if (dragOver === cell.day) dragOver = null; }}
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

    <!-- Content Split: Groups Node Graph on Left, Automate panel on Right -->
    <div class="bottom-content-split">
      <div class="node-graph-scroll">
        <div class="node-graph">
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

            <!-- connector pipe from Delegate to Automate (right edge) -->
            {@const delNode = groups.find(g => g.id === 'delegate')}
            {#if delNode}
              {@const ax = groupRightX(delNode)}
              {@const ay = groupCenterY(delNode)}
              {@const bx = 1380}
              {@const by = ay}
              {@const dx = (bx - ax) * 0.5}
              <path
                d="M {ax} {ay} C {ax+dx} {ay} {bx-dx} {by} {bx} {by}"
                fill="none"
                stroke={isDelegatingAnim ? 'rgba(16, 185, 129, 0.25)' : '#2a2c2d'}
                stroke-width="14"
                stroke-linecap="round"
                class="delegate-to-auto-track"
              />
              <path
                d="M {ax} {ay} C {ax+dx} {ay} {bx-dx} {by} {bx} {by}"
                fill="none"
                stroke={isDelegatingAnim ? '#10b981' : '#404345'}
                stroke-width="1.5"
                class="delegate-to-auto-line"
              />
              <!-- animated flow dot -->
              <circle r="4" fill={isDelegatingAnim ? '#10b981' : '#606466'}>
                <animateMotion
                  dur={isDelegatingAnim ? '0.8s' : '2.8s'}
                  repeatCount="indefinite"
                  path="M {ax} {ay} C {ax+dx} {ay} {bx-dx} {by} {bx} {by}"
                />
              </circle>
              <!-- right port for delegate node -->
              <circle cx={ax} cy={ay} r="5" fill="#1c1b1b" stroke="#4a4c4d" stroke-width="1.2"/>
              <circle cx={ax} cy={ay} r="2.5" fill={isDelegatingAnim ? '#10b981' : '#606466'}/>
            {/if}

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

          <!-- Group enclosures (HTML overlay) -->
          {#each groups as g}
            {@const done = g.isDelegateNode ? g.subtasks.filter(s => s.delegated).length : g.subtasks.filter(s => s.done).length}
            {@const total = g.subtasks.length}
            <div
              class="group-node {g.active ? 'active' : ''}"
              style="left:{g.x}px; top:{g.y}px; width:{GW}px; height:{GH}px;"
            >
              <!-- Header — Fixed -->
              <div class="group-header">
                <div class="group-header-left">
                  <span class="group-phase">{g.phase}</span>
                  <span class="group-label">{g.label}</span>
                </div>
                <div class="group-progress">
                  <span class="group-progress-text">{done}/{total}</span>
                  <div class="group-progress-bar">
                    <div class="group-progress-fill" style="width:{(done/total)*100}%"></div>
                  </div>
                </div>
              </div>

              <!-- Subtask chips / Delegate list -->
              <div class="group-subtasks">
                {#if g.isDelegateNode}
                  {#each g.subtasks as s}
                    <div class="delegate-chip {s.delegated ? 'active' : ''}">
                      <div class="delegate-avatar">
                        {#if s.id === 'sarah'}
                          👩‍💻
                        {:else if s.id === 'apex'}
                          🏢
                        {:else if s.id === 'clara'}
                          ✍️
                        {:else}
                          🤖
                        {/if}
                      </div>
                      <div class="chip-text">
                        <span class="chip-label">{s.label}</span>
                        <span class="chip-detail">{s.detail}</span>
                      </div>
                      <button
                        class="delegate-action-btn {s.delegated ? 'delegated' : ''}"
                        on:click|stopPropagation={() => toggleDelegate(s.id)}
                        type="button"
                      >
                        {s.delegated ? 'ACTIVE' : 'DELEGATE'}
                      </button>
                    </div>
                  {/each}
                {#else}
                  {#each g.subtasks as s}
                    <div
                      class="subtask-chip {s.done ? 'done' : ''} {activeDragTask?.id === s.id ? 'is-dragging-source' : ''}"
                      on:mousedown={e => startTaskDrag(e, s, g.id)}
                      title="Drag to calendar or click checkbox"
                    >
                      <!-- Clickable checkbox to toggle completion -->
                      <button
                        class="chip-check {s.done ? 'checked' : ''}"
                        on:click|stopPropagation={() => toggleSubtask(g.id, s.id)}
                        type="button"
                      >
                        {#if s.done}{@html icCheck()}{/if}
                      </button>
                      <div class="chip-text">
                        <span class="chip-label">{s.label}</span>
                        <span class="chip-detail">{s.detail}</span>
                      </div>
                      <div class="chip-drag-handle">
                        <svg width="10" height="10" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round"><circle cx="9" cy="6" r="1.2" fill="currentColor"/><circle cx="15" cy="6" r="1.2" fill="currentColor"/><circle cx="9" cy="12" r="1.2" fill="currentColor"/><circle cx="15" cy="12" r="1.2" fill="currentColor"/><circle cx="9" cy="18" r="1.2" fill="currentColor"/><circle cx="15" cy="18" r="1.2" fill="currentColor"/></svg>
                      </div>
                    </div>
                  {/each}
                {/if}
              </div>
            </div>
          {/each}
        </div>
      </div>

      <!-- Automate panel on the right -->
      <div class="automate-panel">
        <div class="automate-header">
          <div class="automate-pulse-dot"></div>
          <span class="automate-title">AUTOMATE</span>
        </div>
        
        <div class="stacked-cards-container">
          {#each upcomingTasks as task}
            <!-- Stack calculations for top 3 cards -->
            {@const visualIdx = upcomingTasks.filter(t => !dismissedCardIds.has(t.id)).indexOf(task)}
            {@const isDismissed = dismissedCardIds.has(task.id)}
            {#if visualIdx >= 0 && visualIdx < 3}
              <div
                class="automate-card {isDismissed ? 'dismissed' : ''}"
                style="
                  transform: translateY({visualIdx * 8}px) scale({1 - visualIdx * 0.045}) translateZ({-visualIdx * 10}px);
                  z-index: {10 - visualIdx};
                  opacity: {1 - visualIdx * 0.25};
                  pointer-events: {visualIdx === 0 ? 'auto' : 'none'};
                "
              >
                <div class="auto-card-top">
                  <div class="auto-card-meta">
                    <span class="auto-card-client">{task.client}</span>
                    <span class="auto-card-badge {task.type.toLowerCase()}">{task.type}</span>
                  </div>
                  <h3 class="auto-card-title">{task.title}</h3>
                  <p class="auto-card-detail">{task.detail}</p>
                </div>
                
                <div class="auto-card-actions">
                  <button class="action-btn call" on:click={() => triggerAction(task, 'call')}>
                    <svg width="11" height="11" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M22 16.92v3a2 2 0 0 1-2.18 2 19.79 19.79 0 0 1-8.63-3.07 19.5 19.5 0 0 1-6-6 19.79 19.79 0 0 1-3.07-8.67A2 2 0 0 1 4.11 2h3a2 2 0 0 1 2 1.72 12.84 12.84 0 0 0 .7 2.81 2 2 0 0 1-.45 2.11L8.09 9.91a16 16 0 0 0 6 6l1.27-1.27a2 2 0 0 1 2.11-.45 12.84 12.84 0 0 0 2.81.7A2 2 0 0 1 22 16.92z"/></svg>
                    CALL
                  </button>
                  <button class="action-btn text" on:click={() => triggerAction(task, 'text')}>
                    <svg width="11" height="11" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M21 15a2 2 0 0 1-2 2H7l-4 4V5a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2z"/></svg>
                    TEXT
                  </button>
                  <button class="action-btn checkup" on:click={() => triggerAction(task, 'checkup')}>
                    <svg width="11" height="11" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="10"/><path d="m9 12 2 2 4-4"/></svg>
                    CHECKUP
                  </button>
                </div>
              </div>
            {/if}
          {/each}
        </div>

        <!-- Simulated Action Overlays (Local to Automate Panel) -->
        {#if activeModal === 'call' && activeModalTask}
          <div class="automate-overlay-modal call-mode">
            <div class="calling-container">
              <div class="calling-avatar">
                {activeModalTask.client.substring(0, 2)}
              </div>
              <div class="calling-status">CONNECTING...</div>
              <div class="calling-client">{activeModalTask.client}</div>
              <div class="calling-phone">{activeModalTask.phone}</div>
              
              <!-- Simple waveform visualizer -->
              <div class="waveform">
                <div class="bar"></div>
                <div class="bar"></div>
                <div class="bar"></div>
                <div class="bar"></div>
                <div class="bar"></div>
              </div>

              <div class="call-duration">{callTimer}</div>

              <div class="calling-actions">
                <button class="call-btn mute-btn">
                  <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M12 1a3 3 0 0 0-3 3v8a3 3 0 0 0 6 0V4a3 3 0 0 0-3-3z"/><path d="M19 10v2a7 7 0 0 1-14 0v-2"/><line x1="12" y1="19" x2="12" y2="23"/></svg>
                </button>
                <button class="call-btn end-call" on:click={completeModalAction}>
                  <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M10.68 13.31a16 16 0 0 0 3.41 2.6l1.27-1.27a2 2 0 0 1 2.11-.45 12.84 12.84 0 0 0 2.81.7A2 2 0 0 1 22 16.92v3a2 2 0 0 1-2.18 2 19.79 19.79 0 0 1-8.63-3.07 19.5 19.5 0 0 1-6-6 19.79 19.79 0 0 1-3.07-8.67A2 2 0 0 1 4.11 2h3a2 2 0 0 1 2 1.72 12.84 12.84 0 0 0 .7 2.81 2 2 0 0 1-.45 2.11L8.09 9.91" stroke-linecap="round" stroke-linejoin="round"/></svg>
                </button>
              </div>
            </div>
          </div>
        {:else if activeModal === 'text' && activeModalTask}
          <div class="automate-overlay-modal text-mode">
            <div class="texting-header">
              <span class="texting-to">To: {activeModalTask.client}</span>
              <button class="texting-close" on:click={cancelModalAction}>&times;</button>
            </div>
            
            <div class="texting-history">
              <div class="msg received">
                <span class="msg-text">Hey Basion Team, is scheduling confirmed?</span>
              </div>
              <div class="msg sent animate-send">
                <span class="msg-text">{textMessage}</span>
              </div>
            </div>

            <div class="texting-input-area">
              <input type="text" bind:value={textMessage} placeholder="Type a message..." />
              <button class="texting-send-btn" on:click={completeModalAction}>
                <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="22" y1="2" x2="11" y2="13"/><polygon points="22 2 15 22 11 13 2 9 22 2"/></svg>
              </button>
            </div>
          </div>
        {:else if activeModal === 'checkup' && activeModalTask}
          <div class="automate-overlay-modal checkup-mode">
            <div class="checkup-header">
              <span class="checkup-title">AUDIT LOGS</span>
              <button class="checkup-close" on:click={cancelModalAction}>&times;</button>
            </div>
            
            <div class="checkup-body">
              <div class="audit-icon-wrapper">
                <svg class="audit-spinner {checkupProgress === 100 ? 'done' : ''}" viewBox="0 0 36 36">
                  <path class="circle-bg" d="M18 2.0845 a 15.9155 15.9155 0 0 1 0 31.831 a 15.9155 15.9155 0 0 1 0 -31.831" />
                  <path class="circle" stroke-dasharray="{checkupProgress}, 100" d="M18 2.0845 a 15.9155 15.9155 0 0 1 0 31.831 a 15.9155 15.9155 0 0 1 0 -31.831" />
                </svg>
                <div class="audit-percent">{checkupProgress}%</div>
              </div>

              <div class="checkup-stage">{checkupStage}</div>
              
              {#if checkupProgress === 100}
                <button class="checkup-complete-btn" on:click={completeModalAction}>
                  RESOLVE TASK
                </button>
              {/if}
            </div>
          </div>
        {/if}
      </div>
    </div>
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

  <!-- ── Custom pop-out floating drag card ──────────────── -->
  {#if showDragCard && activeDragTask}
    <div
      class="floating-drag-card"
      style="left: {dragMouseX}px; top: {dragMouseY}px;"
    >
      <div class="drag-card-badge">DROP IN CALENDAR</div>
      <div class="drag-card-title">{activeDragTask.label}</div>
      <div class="drag-card-detail">{activeDragTask.detail}</div>
    </div>
  {/if}

</div>

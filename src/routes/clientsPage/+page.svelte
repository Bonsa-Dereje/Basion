<script>
  import './clientsPage.css';

  let selectedClient = 'Starlight Media';

  const clients = [
    { id: null,    name: 'Starlight Media',  label: 'Selected' },
    { id: '992',   name: 'Vanguard Labs',    label: null },
    { id: '841',   name: 'Nexus Group',      label: null },
    { id: '211',   name: 'Aero Dynamics',    label: null },
  ];

  const actions = [
    {
      priority: true,
      icon: 'clipboard',
      title: 'Approve Brand Assets',
      sub: 'Deadline: Today, 5:00 PM',
    },
    {
      priority: false,
      icon: 'upload',
      title: 'Upload Raw Footage',
      sub: 'Starlight Media HQ Server',
    },
    {
      priority: false,
      icon: 'message-square',
      title: 'Review Editor Feedback',
      sub: '2 new comments on V1',
    },
  ];

  const weekDays = [
    { label: 'MON', num: 11, event: null },
    { label: 'TUE', num: 12, event: null },
    { label: 'WED', num: 12, event: { bar: true, text: 'REEL #03\nLAUNCH' }, today: true },
    { label: 'THU', num: 13, event: null },
    { label: 'FRI', num: 14, event: null, fri: true },
    { label: 'SAT', num: 15, event: null },
    { label: 'SUN', num: 16, event: null },
  ];

  const phases = [
    { num: '01', name: 'Scripting',  status: '100% Complete', statusClass: 'complete', filled: 5, partial: 0 },
    { num: '02', name: 'Production', status: '60% Complete',  statusClass: 'complete', filled: 3, partial: 1 },
    { num: '03', name: 'Post-Edit',  status: 'Pending',       statusClass: 'pending',  filled: 0, partial: 0 },
  ];

  const icons = {
    clipboard:      `<path d="M16 4h2a2 2 0 0 1 2 2v14a2 2 0 0 1-2 2H6a2 2 0 0 1-2-2V6a2 2 0 0 1 2-2h2"/><rect x="8" y="2" width="8" height="4" rx="1"/>`,
    upload:         `<path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/><polyline points="17 8 12 3 7 8"/><line x1="12" y1="3" x2="12" y2="15"/>`,
    'message-square': `<path d="M21 15a2 2 0 0 1-2 2H7l-4 4V5a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2z"/>`,
    'chevron-left':  `<polyline points="15 18 9 12 15 6"/>`,
    'chevron-right': `<polyline points="9 18 15 12 9 6"/>`,
    'external-link': `<path d="M18 13v6a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h6"/><polyline points="15 3 21 3 21 9"/><line x1="10" y1="14" x2="21" y2="3"/>`,
  };

  function ic(name, size = 14) {
    return `<svg width="${size}" height="${size}" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round">${icons[name] || ''}</svg>`;
  }
</script>

<div class="clients-page">

  <!-- ── Client Selector Strip ──────────────────────── -->
  <div class="client-strip">
    {#each clients as c}
      <div
        class="client-tab {selectedClient === c.name ? 'selected' : ''}"
        role="button" tabindex="0"
        on:click={() => selectedClient = c.name}
        on:keydown={e => e.key === 'Enter' && (selectedClient = c.name)}
      >
        <div class="client-tab-meta">
          {c.label ?? `ID: ${c.id}`}
        </div>
        <div class="client-tab-name">{c.name}</div>
      </div>
    {/each}
  </div>

  <!-- ── Body ───────────────────────────────────────── -->
  <div class="clients-body">

    <!-- Left Column -->
    <div class="left-col">

      <!-- Actions Required -->
      <div class="actions-section">
        <div class="actions-header">
          <span class="actions-title">Actions Required</span>
          <span class="urgent-badge">3 Urgent</span>
        </div>

        {#each actions as a}
          {#if a.priority}
            <div class="action-primary">
              <span class="action-icon">{@html ic(a.icon, 15)}</span>
              <div>
                <div class="action-title">{a.title}</div>
                <div class="action-sub">{a.sub}</div>
              </div>
            </div>
          {:else}
            <div class="action-card">
              <span class="action-icon">{@html ic(a.icon, 15)}</span>
              <div>
                <div class="action-title">{a.title}</div>
                <div class="action-sub">{a.sub}</div>
              </div>
            </div>
          {/if}
        {/each}
      </div>

      <!-- Financial Status -->
      <div class="financial-section">
        <div class="financial-card">
          <div class="fin-label">Financial Status</div>
          <div class="fin-amount">$4,200.00</div>
          <div class="fin-invoice">Invoice #SL-2024-004</div>
          <div class="fin-due-row">
            <div>
              <div class="fin-due-label">Payment Due Date</div>
              <div class="fin-due-value">Due in 3 days</div>
            </div>
            <button class="pay-btn">Pay Now</button>
          </div>
        </div>
      </div>

      <!-- Lead Found -->
      <div class="lead-section">
        <div class="lead-card">
          <div>
            <div class="lead-badge">Lead Found</div>
            <div class="lead-user">
              <div class="lead-avatar">DS</div>
              <span class="lead-handle">@design_studio_x</span>
            </div>
          </div>
          <span class="lead-external">{@html ic('external-link', 15)}</span>
        </div>
      </div>

      <!-- New Project -->
      <button class="new-project-btn">New Project</button>

    </div>

    <!-- Right Column -->
    <div class="right-col">

      <!-- Content Schedule -->
      <div class="schedule-section">
        <div class="schedule-header">
          <span class="schedule-title">Content Schedule</span>
          <div class="schedule-nav">
            <button class="schedule-nav-btn">{@html ic('chevron-left', 14)}</button>
            <button class="schedule-nav-btn">{@html ic('chevron-right', 14)}</button>
          </div>
        </div>

        <div class="week-grid">
          {#each weekDays as d}
            <div class="day-col">
              <div class="day-header">{d.label}</div>
              <div class="day-cell {d.today ? 'today' : ''}">
                <div class="day-number">{d.num}</div>

                {#if d.fri}
                  <div class="fri-block"></div>
                {/if}

                {#if d.event}
                  <div class="day-event">
                    {#if d.event.bar}
                      <div class="event-bar"></div>
                    {/if}
                    <div class="event-label">
                      {#each d.event.text.split('\n') as line}
                        {line}<br/>
                      {/each}
                    </div>
                  </div>
                {/if}
              </div>
            </div>
          {/each}
        </div>
      </div>

      <!-- Phase Progression -->
      <div class="phase-section">
        <div class="phase-title">Phase Progression</div>

        {#each phases as p}
          <div class="phase-row">
            <div class="phase-row-header">
              <span class="phase-name">{p.num} &nbsp; {p.name}</span>
              <span class="phase-status {p.statusClass}">{p.status}</span>
            </div>
            <div class="phase-track">
              {#each Array(5) as _, i}
                <div
                  class="phase-seg
                    {i < p.filled ? 'filled' : ''}
                    {i === p.filled && p.partial ? 'partial' : ''}"
                ></div>
              {/each}
            </div>
          </div>
        {/each}
      </div>

    </div>
  </div>
</div>
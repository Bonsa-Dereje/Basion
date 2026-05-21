<script>
  import './clientsPage.css';

  let selectedClient = 'Starlight Media';

  const clients = [
    { id: null,  name: 'Starlight Media', label: 'Selected' },
    { id: '992', name: 'Vanguard Labs',   label: null },
    { id: '841', name: 'Nexus Group',     label: null },
    { id: '211', name: 'Aero Dynamics',   label: null },
  ];

  const actions = [
    { priority: true,  icon: 'clipboard',       title: 'Approve Brand Assets',    sub: 'Deadline: Today, 5:00 PM' },
    { priority: false, icon: 'upload',           title: 'Upload Raw Footage',      sub: 'Starlight Media HQ Server' },
    { priority: false, icon: 'message-square',   title: 'Review Editor Feedback',  sub: '2 new comments on V1' },
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

  // Project workflow nodes — node style with approve/revise actions
  // positions: { left, top } in px relative to canvas (560×260)
  const wfNodes = [
    {
      id: 'brief',
      label: 'Input',
      title: 'Creative Brief',
      sub: 'Locked · Apr 8',
      state: 'done',
      pos: { left: 0, top: 80 },
      actions: null,
    },
    {
      id: 'script',
      label: 'Phase 01',
      title: 'Script Draft',
      sub: 'Delivered · Apr 10',
      state: 'done',
      pos: { left: 150, top: 20 },
      actions: null,
    },
    {
      id: 'roughcut',
      label: 'Phase 02',
      title: 'Rough Cut',
      sub: 'Awaiting review',
      state: 'active',
      pos: { left: 150, top: 140 },
      actions: { approve: 'Approve', revise: 'Revise' },
    },
    {
      id: 'vfx',
      label: 'Phase 03',
      title: 'VFX & Grade',
      sub: 'Blocked — pending cut',
      state: 'pending',
      pos: { left: 310, top: 80 },
      actions: null,
    },
    {
      id: 'final',
      label: 'Delivery',
      title: 'Final Export',
      sub: 'Not started',
      state: 'pending',
      pos: { left: 430, top: 80 },
      actions: { approve: 'Sign off', revise: 'Hold' },
    },
  ];

  // SVG connector paths between nodes (from right edge to left edge)
  // Each node is 130px wide, 20px taller due to header
  // We define center-right and center-left of each node
  function nodeCenter(node) {
    const h = node.actions ? 112 : 78;
    return { x: node.pos.left + 130, y: node.pos.top + h / 2 };
  }
  function nodeLeft(node) {
    const h = node.actions ? 112 : 78;
    return { x: node.pos.left, y: node.pos.top + h / 2 };
  }

  const connections = [
    ['brief',    'script'],
    ['brief',    'roughcut'],
    ['script',   'vfx'],
    ['roughcut', 'vfx'],
    ['vfx',      'final'],
  ];

  function bezier(from, to) {
    const dx = (to.x - from.x) * 0.5;
    return `M ${from.x} ${from.y} C ${from.x + dx} ${from.y} ${to.x - dx} ${to.y} ${to.x} ${to.y}`;
  }

  // Follow-up tracker messages
  const followups = [
    {
      avatar: 'ED',
      name: 'Editor',
      role: 'Post',
      time: '2 min ago',
      msg: 'Will be done by 8 — just finishing colour pass on scene 3.',
      tag: 'urgent',
      tagLabel: 'Awaiting',
    },
    {
      avatar: 'DI',
      name: 'Director',
      role: 'Creative',
      time: '1 hr ago',
      msg: 'Script V2 approved. Can we push rough cut review to Thursday?',
      tag: null,
      tagLabel: null,
    },
    {
      avatar: 'SM',
      name: 'Starlight Media',
      role: 'Client',
      time: '3 hr ago',
      msg: 'Loved the brand deck — please add one more slide on deliverables.',
      tag: 'urgent',
      tagLabel: 'Action needed',
    },
    {
      avatar: 'VX',
      name: 'VFX Team',
      role: 'Post',
      time: 'Yesterday',
      msg: 'Assets uploaded to /delivery/vfx-exports. Ready for review.',
      tag: 'done',
      tagLabel: 'Done',
    },
    {
      avatar: 'PM',
      name: 'Producer',
      role: 'Ops',
      time: 'Yesterday',
      msg: 'Invoice sent — chasing payment. Due in 3 days.',
      tag: null,
      tagLabel: null,
    },
  ];

  // Icon map
  const icons = {
    clipboard:        `<path d="M16 4h2a2 2 0 0 1 2 2v14a2 2 0 0 1-2 2H6a2 2 0 0 1-2-2V6a2 2 0 0 1 2-2h2"/><rect x="8" y="2" width="8" height="4" rx="1"/>`,
    upload:           `<path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/><polyline points="17 8 12 3 7 8"/><line x1="12" y1="3" x2="12" y2="15"/>`,
    'message-square': `<path d="M21 15a2 2 0 0 1-2 2H7l-4 4V5a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2z"/>`,
    'chevron-left':   `<polyline points="15 18 9 12 15 6"/>`,
    'chevron-right':  `<polyline points="9 18 15 12 9 6"/>`,
    'external-link':  `<path d="M18 13v6a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h6"/><polyline points="15 3 21 3 21 9"/><line x1="10" y1="14" x2="21" y2="3"/>`,
    check:            `<polyline points="20 6 9 17 4 12"/>`,
    plus:             `<line x1="12" y1="5" x2="12" y2="19"/><line x1="5" y1="12" x2="19" y2="12"/>`,
    instagram:        `<rect x="2" y="2" width="20" height="20" rx="5"/><circle cx="12" cy="12" r="4"/><circle cx="17.5" cy="6.5" r="1" fill="currentColor" stroke="none"/>`,
    users:            `<path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"/><circle cx="9" cy="7" r="4"/><path d="M23 21v-2a4 4 0 0 0-3-3.87"/><path d="M16 3.13a4 4 0 0 1 0 7.75"/>`,
    'trending-up':    `<polyline points="23 6 13.5 15.5 8.5 10.5 1 18"/><polyline points="17 6 23 6 23 12"/>`,
  };

  function ic(name, size = 14) {
    return `<svg width="${size}" height="${size}" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round">${icons[name] || ''}</svg>`;
  }

  // Build node lookup for connections
  const nodeMap = Object.fromEntries(wfNodes.map(n => [n.id, n]));
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
        <div class="client-tab-meta">{c.label ?? `ID: ${c.id}`}</div>
        <div class="client-tab-name">{c.name}</div>
      </div>
    {/each}
  </div>

  <!-- ── Body ───────────────────────────────────────── -->
  <div class="clients-body">

    <!-- ── Left Column ──────────────────────────────── -->
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

      <!-- Lead Found — expanded with social info -->
      <div class="lead-section">
        <div class="lead-card">

          <!-- Top: handle + external -->
          <div class="lead-card-top">
            <div>
              <div class="lead-badge">Lead Found</div>
              <div class="lead-user">
                <div class="lead-avatar">DS</div>
                <div>
                  <div class="lead-handle">@design_studio_x</div>
                  <div style="font-family:var(--font-mono);font-size:9px;color:var(--outline);margin-top:2px;">Instagram · Design</div>
                </div>
              </div>
            </div>
            <span class="lead-external">{@html ic('external-link', 14)}</span>
          </div>

          <!-- Social stats 2×2 grid -->
          <div class="lead-stats">
            <div class="lead-stat">
              <div class="lead-stat-label">Followers</div>
              <div class="lead-stat-value">84.2k</div>
              <div class="lead-stat-sub">{@html ic('trending-up', 10)} +2.1k this week</div>
            </div>
            <div class="lead-stat">
              <div class="lead-stat-label">Avg. Views</div>
              <div class="lead-stat-value">12.8k</div>
              <div class="lead-stat-sub">per reel</div>
            </div>
            <div class="lead-stat">
              <div class="lead-stat-label">Engagement</div>
              <div class="lead-stat-value">6.4%</div>
              <div class="lead-stat-sub">above avg</div>
            </div>
            <div class="lead-stat">
              <div class="lead-stat-label">Posts / mo</div>
              <div class="lead-stat-value">18</div>
              <div class="lead-stat-sub">consistent</div>
            </div>
          </div>



          <!-- Niche tags -->
          <div class="lead-tags">
            <span class="lead-tag">Branding</span>
            <span class="lead-tag">Motion</span>
            <span class="lead-tag">Reels</span>
            <span class="lead-tag">B2B</span>
            <span class="lead-tag">Design</span>
          </div>

          <!-- CTA buttons -->
          <div class="lead-actions">
            <button class="lead-btn-primary">Reach Out</button>
            <button class="lead-btn-secondary">Save Lead</button>
          </div>

        </div>
      </div>

      <!-- New Project -->
      <button class="new-project-btn">New Project</button>

    </div>

    <!-- ── Right Column ──────────────────────────────── -->
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
                {#if d.fri}<div class="fri-block"></div>{/if}
                {#if d.event}
                  <div class="day-event">
                    {#if d.event.bar}<div class="event-bar"></div>{/if}
                    <div class="event-label">
                      {#each d.event.text.split('\n') as line}{line}<br/>{/each}
                    </div>
                  </div>
                {/if}
              </div>
            </div>
          {/each}
        </div>
      </div>

      <!-- Bottom: Workflow + Follow-up side by side -->
      <div class="right-bottom">

        <!-- ── Project Workflow (node canvas) ───────── -->
        <div class="workflow-section">
          <div class="workflow-title">Project Workflow</div>

          <div class="node-canvas">

            <!-- SVG connector lines -->
            <svg class="node-svg" xmlns="http://www.w3.org/2000/svg">
              {#each connections as [fromId, toId]}
                {@const fromNode = nodeMap[fromId]}
                {@const toNode   = nodeMap[toId]}
                {@const from = nodeCenter(fromNode)}
                {@const to   = nodeLeft(toNode)}
                <path
                  d="{bezier(from, to)}"
                  fill="none"
                  stroke={fromNode.state === 'done' ? '#8e9192' : '#444748'}
                  stroke-width="1.2"
                  stroke-dasharray={fromNode.state === 'pending' ? '4,4' : 'none'}
                />
              {/each}
            </svg>

            <!-- Nodes -->
            {#each wfNodes as node}
              <div
                class="wf-node {node.state}"
                style="left:{node.pos.left}px; top:{node.pos.top}px;"
              >
                <div class="wf-node-header">
                  <span class="wf-node-label">{node.label}</span>
                  <span class="wf-status-dot {node.state}"></span>
                </div>
                <div class="wf-node-body">
                  <div class="wf-node-title">{node.title}</div>
                  <div class="wf-node-sub">{node.sub}</div>

                  {#if node.actions}
                    <div class="wf-node-actions">
                      <button class="wf-btn-approve">{node.actions.approve}</button>
                      <button class="wf-btn-revise">{node.actions.revise}</button>
                    </div>
                  {:else if node.state === 'done'}
                    <div class="wf-done-badge">
                      {@html ic('check', 11)}
                      Approved
                    </div>
                  {/if}
                </div>

                <!-- connector dots -->
                {#if node.id !== 'brief'}
                  <div class="wf-connector left"></div>
                {/if}
                {#if node.id !== 'final'}
                  <div class="wf-connector"></div>
                {/if}
              </div>
            {/each}

          </div>
        </div>

        <!-- ── Follow-up Tracker ─────────────────────── -->
        <div class="followup-section">
          <div class="followup-header">
            <span class="followup-title">Follow-ups</span>
            <span class="followup-count">5 open</span>
          </div>

          <div class="followup-list">
            {#each followups as f}
              <div class="followup-item">
                <div class="followup-row1">
                  <div class="followup-who">
                    <div class="followup-avatar">{f.avatar}</div>
                    <div>
                      <div class="followup-name">{f.name}</div>
                      <div class="followup-role">{f.role}</div>
                    </div>
                  </div>
                  <span class="followup-time">{f.time}</span>
                </div>

                <div class="followup-msg">{f.msg}</div>

                {#if f.tag}
                  <div class="followup-tag {f.tag}">{f.tagLabel}</div>
                {/if}
              </div>
            {/each}
          </div>

          <button class="followup-add">
            + Add Follow-up
          </button>
        </div>

      </div>
    </div>
  </div>
</div>
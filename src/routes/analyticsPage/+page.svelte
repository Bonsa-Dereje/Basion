<script>
  import './analyticsPage.css';

  let activeDay = 'Tue';

  const days = ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'];

  const pipelineSteps = [
    { label: 'Script',   state: 'done' },
    { label: 'Shoot',    state: 'done' },
    { label: 'Edit',     state: 'active' },
    { label: 'Review',   state: 'pending' },
    { label: 'Approval', state: 'pending' },
    { label: 'Post',     state: 'pending' },
  ];

  const stats = [
    { label: 'Likes',    value: '24.8k', dotClass: 'filled' },
    { label: 'Comments', value: '1.2k',  dotClass: 'dim' },
    { label: 'Views',    value: '142.5k', dotClass: 'dimmer' },
  ];

  const tasks = [
    { text: 'Draft script for AI Automation post', done: true },
    { text: 'Shoot B-roll in main office',          done: true },
    { text: 'Edit video intro (VFX layers)',         done: false },
    { text: 'Final post review & scheduling',        done: false },
  ];

  const bars = [38, 52, 44, 60, 48, 70, 95];

  const campaigns = [
    { name: 'Q2 Tech Security Series',  views: '124.5k', shares: '1.2k', status: 'active' },
    { name: 'Office Transparency Vlog', views: '89.2k',  shares: '842',  status: 'closed' },
    { name: 'AI Integration Features',  views: '--',      shares: '--',   status: 'pending' },
  ];

  function dashArray(pct, r) {
    const c = 2 * Math.PI * r;
    return `${(pct / 100) * c} ${c}`;
  }

  function linePath(points, w, h) {
    const maxV = Math.max(...points);
    const xs = points.map((_, i) => (i / (points.length - 1)) * w);
    const ys = points.map(v => h - (v / maxV) * h * 0.85);
    return xs.map((x, i) => `${i === 0 ? 'M' : 'L'} ${x} ${ys[i]}`).join(' ');
  }

  const tiktokPts = [20, 35, 50, 40, 60, 80, 95, 120, 145, 170, 185, 200];
  const igPts     = [60, 80, 95, 70, 55, 60, 75,  80,  70,  60,  50,  65];
  const fbPts     = [10, 12, 18, 20, 15, 18, 22,  25,  20,  18,  22,  28];

  const icons = {
    search:      `<circle cx="11" cy="11" r="8"/><line x1="21" y1="21" x2="16.65" y2="16.65"/>`,
    bell:        `<path d="M18 8A6 6 0 0 0 6 8c0 7-3 9-3 9h18s-3-2-3-9"/><path d="M13.73 21a2 2 0 0 1-3.46 0"/>`,
    cog2:        `<circle cx="12" cy="12" r="3"/><path d="M19.4 15a1.65 1.65 0 0 0 .33 1.82l.06.06a2 2 0 1 1-2.83 2.83l-.06-.06a1.65 1.65 0 0 0-1.82-.33 1.65 1.65 0 0 0-1 1.51V21a2 2 0 0 1-4 0v-.09A1.65 1.65 0 0 0 9 19.4a1.65 1.65 0 0 0-1.82.33l-.06-.06a2 2 0 1 1-2.83-2.83l.06-.06A1.65 1.65 0 0 0 4.68 15a1.65 1.65 0 0 0-1.51-1H3a2 2 0 0 1 0-4h.09A1.65 1.65 0 0 0 4.6 9a1.65 1.65 0 0 0-.33-1.82l-.06-.06a2 2 0 1 1 2.83-2.83l.06.06A1.65 1.65 0 0 0 9 4.68a1.65 1.65 0 0 0 1-1.51V3a2 2 0 0 1 4 0v.09a1.65 1.65 0 0 0 1 1.51 1.65 1.65 0 0 0 1.82-.33l.06-.06a2 2 0 1 1 2.83 2.83l-.06.06A1.65 1.65 0 0 0 19.4 9a1.65 1.65 0 0 0 1.51 1H21a2 2 0 0 1 0 4h-.09a1.65 1.65 0 0 0-1.51 1z"/>`,
    calendar2:   `<rect x="3" y="4" width="18" height="18" rx="2"/><line x1="16" y1="2" x2="16" y2="6"/><line x1="8" y1="2" x2="8" y2="6"/><line x1="3" y1="10" x2="21" y2="10"/>`,
    check:       `<polyline points="20 6 9 17 4 12"/>`,
  };

  function ic(name, size = 14) {
    return `<svg width="${size}" height="${size}" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round">${icons[name] || ''}</svg>`;
  }
</script>

<div class="page-header">
  <div>
    <h1 class="page-title">Analytics Overview</h1>
    <p class="page-subtitle">Real-time performance metrics and production tracking.</p>
  </div>

  <div class="week-nav">
    <div class="week-label">
      {@html ic('calendar2', 12)}
      April – Week 3
    </div>

    {#each days as day}
      <button
        class="day-btn {activeDay === day ? 'active' : ''}"
        on:click={() => activeDay = day}
      >{day}</button>
    {/each}
  </div>
</div>

<div class="dashboard-scroll">
  <div class="dashboard-grid">

    <div class="card engagement-card">
      <div class="card-label">Engagement Profile</div>

      <div class="donut-area">
        <div class="donut-wrap">
          <svg viewBox="0 0 180 180" width="180" height="180">

            <circle cx="90" cy="90" r="72"
              fill="none"
              stroke="#353434"
              stroke-width="14"
            />

            <circle cx="90" cy="90" r="72"
              fill="none"
              stroke="#ffffff"
              stroke-width="14"
              stroke-dasharray="{dashArray(72, 72)}"
              stroke-linecap="round"
            />

            <circle cx="90" cy="90" r="54"
              fill="none"
              stroke="#353434"
              stroke-width="12"
            />

            <circle cx="90" cy="90" r="54"
              fill="none"
              stroke="#8e9192"
              stroke-width="12"
              stroke-dasharray="{dashArray(55, 54)}"
              stroke-linecap="round"
            />

            <circle cx="90" cy="90" r="37"
              fill="none"
              stroke="#353434"
              stroke-width="10"
            />

            <circle cx="90" cy="90" r="37"
              fill="none"
              stroke="#474746"
              stroke-width="10"
              stroke-dasharray="{dashArray(38, 37)}"
              stroke-linecap="round"
            />
          </svg>

          <div class="donut-center">
            <span class="donut-label">Overall</span>
            <span class="donut-value">72%</span>
          </div>
        </div>
      </div>

      <div class="stats-list">
        {#each stats as s}
          <div class="stat-row">
            <div class="stat-left">
              <span class="stat-dot {s.dotClass}"></span>
              {s.label}
            </div>

            <span class="stat-val">{s.value}</span>
          </div>
        {/each}
      </div>

      <div class="tasks-section">
        <div class="tasks-header">
          <span class="tasks-title">Tasks for Thursday</span>
          <span class="tasks-count">03/04</span>
        </div>

        {#each tasks as t}
          <div class="task-item {t.done ? 'done' : ''}">
            <div class="task-check {t.done ? 'done' : ''}">
              {#if t.done}
                {@html ic('check', 10)}
              {/if}
            </div>

            <span class="task-text">{t.text}</span>
          </div>
        {/each}
      </div>
    </div>

    <div class="card progress-card">
      <div class="card-label">Current Video Production's Progress</div>

      <div class="pipeline-track">
        {#each pipelineSteps as step}
          <div class="pipeline-step {step.state}">
            <div class="step-dot {step.state}"></div>
            <span class="step-label">{step.label}</span>
          </div>
        {/each}
      </div>
    </div>

    <div class="stats-row">

      <div class="card views-card">
        <div class="card-label">Last Video Views</div>
        <div class="views-number">14.2k</div>

        <div class="bar-chart">
          {#each bars as h, i}
            <div
              class="bar {i === bars.length - 2 ? 'highlight' : ''}"
              style="height: {h}%"
            ></div>
          {/each}
        </div>
      </div>

      <div class="card platform-card">

        <div style="display:flex; align-items:center; justify-content:space-between; padding: 10px 16px 0; flex-shrink:0;">
          <div class="card-label" style="border:none; padding:0;">
            Total View Across Platforms
          </div>

          <div class="platform-legend">
            <div class="legend-item">
              <span class="legend-dot filled"></span> TikTok
            </div>

            <div class="legend-item">
              <span class="legend-dot outline-dot"></span> IG
            </div>

            <div class="legend-item">
              <span class="legend-dot dim"></span> FB
            </div>
          </div>
        </div>

        <div class="line-chart-wrap">
          <svg viewBox="0 0 380 90" width="100%" height="90" preserveAspectRatio="none">

            <line x1="0" y1="88" x2="380" y2="88" stroke="#444748" stroke-width="0.5"/>
            <line x1="0" y1="60" x2="380" y2="60" stroke="#333" stroke-width="0.5" stroke-dasharray="3,4"/>
            <line x1="0" y1="30" x2="380" y2="30" stroke="#333" stroke-width="0.5" stroke-dasharray="3,4"/>

            <path
              d="{linePath(fbPts, 380, 88)}"
              fill="none"
              stroke="#8e9192"
              stroke-width="1.2"
              stroke-dasharray="4,4"
            />

            <path
              d="{linePath(igPts, 380, 88)}"
              fill="none"
              stroke="#c4c7c8"
              stroke-width="1.5"
              stroke-dasharray="6,3"
            />

            <path
              d="{linePath(tiktokPts, 380, 88)}"
              fill="none"
              stroke="#ffffff"
              stroke-width="2"
            />
          </svg>
        </div>

        <div class="axis-labels">
          {#each ['JAN','MAR','MAY','JUL','SEP','NOV'] as m}
            <span class="axis-label">{m}</span>
          {/each}
        </div>

      </div>

    </div>

    <div class="card performance-card">
      <div class="card-label">Content Performance Breakdown</div>

      <table class="perf-table">
        <thead>
          <tr>
            <th>Campaign</th>
            <th>Views</th>
            <th>Shares</th>
            <th>Status</th>
          </tr>
        </thead>

        <tbody>
          {#each campaigns as c}
            <tr>
              <td>{c.name}</td>

              <td class="{c.views === '--' ? 'muted' : ''}">
                {c.views}
              </td>

              <td class="{c.shares === '--' ? 'muted' : ''}">
                {c.shares}
              </td>

              <td>
                <span class="status-badge {c.status}">
                  {c.status}
                </span>
              </td>
            </tr>
          {/each}
        </tbody>
      </table>
    </div>

  </div>
</div>
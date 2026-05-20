<script>
  import { onMount } from 'svelte';
    import './homePage.css';

  let activeNav = 'Dashboard';

  const navItems = [
    { label: 'Dashboard', icon: 'grid' },
    { label: 'Clients',   icon: 'users' },
    { label: 'Calendar',  icon: 'calendar' },
    { label: 'Automation',icon: 'cpu' },
    { label: 'Analytics', icon: 'bar-chart' },
  ];

  const footerItems = [
    { label: 'Settings', icon: 'settings' },
    { label: 'Logout',   icon: 'log-out' },
  ];

  const projects = [
    { name: 'Starlight Media',   phase: 'Shoot Phase', pct: 75 },
    { name: 'Nordic Design',     phase: 'Editing',     pct: 40 },
    { name: 'Urban Outfitters',  phase: 'Review',      pct: 15 },
    { name: 'Luxe Hospitality',  phase: 'Delivery',    pct: 95 },
  ];

  const flowSteps = [
    { label: 'Trigger', icon: 'mail' },
    { label: 'Logic Gate', icon: 'user' },
    { label: 'Payment', icon: 'credit-card' },
  ];

  // SVG icon helper
  const icons = {
    grid: `<rect x="3" y="3" width="7" height="7" rx="1"/><rect x="14" y="3" width="7" height="7" rx="1"/><rect x="3" y="14" width="7" height="7" rx="1"/><rect x="14" y="14" width="7" height="7" rx="1"/>`,
    users: `<path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"/><circle cx="9" cy="7" r="4"/><path d="M23 21v-2a4 4 0 0 0-3-3.87"/><path d="M16 3.13a4 4 0 0 1 0 7.75"/>`,
    calendar: `<rect x="3" y="4" width="18" height="18" rx="2"/><line x1="16" y1="2" x2="16" y2="6"/><line x1="8" y1="2" x2="8" y2="6"/><line x1="3" y1="10" x2="21" y2="10"/>`,
    cpu: `<rect x="4" y="4" width="16" height="16" rx="2"/><rect x="9" y="9" width="6" height="6"/><line x1="9" y1="1" x2="9" y2="4"/><line x1="15" y1="1" x2="15" y2="4"/><line x1="9" y1="20" x2="9" y2="23"/><line x1="15" y1="20" x2="15" y2="23"/><line x1="20" y1="9" x2="23" y2="9"/><line x1="20" y1="14" x2="23" y2="14"/><line x1="1" y1="9" x2="4" y2="9"/><line x1="1" y1="14" x2="4" y2="14"/>`,
    'bar-chart': `<line x1="18" y1="20" x2="18" y2="10"/><line x1="12" y1="20" x2="12" y2="4"/><line x1="6" y1="20" x2="6" y2="14"/><line x1="2" y1="20" x2="22" y2="20"/>`,
    settings: `<circle cx="12" cy="12" r="3"/><path d="M19.4 15a1.65 1.65 0 0 0 .33 1.82l.06.06a2 2 0 1 1-2.83 2.83l-.06-.06a1.65 1.65 0 0 0-1.82-.33 1.65 1.65 0 0 0-1 1.51V21a2 2 0 0 1-4 0v-.09A1.65 1.65 0 0 0 9 19.4a1.65 1.65 0 0 0-1.82.33l-.06.06a2 2 0 1 1-2.83-2.83l.06-.06A1.65 1.65 0 0 0 4.68 15a1.65 1.65 0 0 0-1.51-1H3a2 2 0 0 1 0-4h.09A1.65 1.65 0 0 0 4.6 9a1.65 1.65 0 0 0-.33-1.82l-.06-.06a2 2 0 1 1 2.83-2.83l.06.06A1.65 1.65 0 0 0 9 4.68a1.65 1.65 0 0 0 1-1.51V3a2 2 0 0 1 4 0v.09a1.65 1.65 0 0 0 1 1.51 1.65 1.65 0 0 0 1.82-.33l.06-.06a2 2 0 1 1 2.83 2.83l-.06.06A1.65 1.65 0 0 0 19.4 9a1.65 1.65 0 0 0 1.51 1H21a2 2 0 0 1 0 4h-.09a1.65 1.65 0 0 0-1.51 1z"/>`,
    'log-out': `<path d="M9 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h4"/><polyline points="16 17 21 12 16 7"/><line x1="21" y1="12" x2="9" y2="12"/>`,
    search: `<circle cx="11" cy="11" r="8"/><line x1="21" y1="21" x2="16.65" y2="16.65"/>`,
    bell: `<path d="M18 8A6 6 0 0 0 6 8c0 7-3 9-3 9h18s-3-2-3-9"/><path d="M13.73 21a2 2 0 0 1-3.46 0"/>`,
    'more-vertical': `<circle cx="12" cy="5" r="1"/><circle cx="12" cy="12" r="1"/><circle cx="12" cy="19" r="1"/>`,
    plus: `<line x1="12" y1="5" x2="12" y2="19"/><line x1="5" y1="12" x2="19" y2="12"/>`,
    mail: `<path d="M4 4h16c1.1 0 2 .9 2 2v12c0 1.1-.9 2-2 2H4c-1.1 0-2-.9-2-2V6c0-1.1.9-2 2-2z"/><polyline points="22,6 12,13 2,6"/>`,
    user: `<path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"/><circle cx="12" cy="7" r="4"/>`,
    'credit-card': `<rect x="1" y="4" width="22" height="16" rx="2"/><line x1="1" y1="10" x2="23" y2="10"/>`,
    send: `<line x1="22" y1="2" x2="11" y2="13"/><polygon points="22 2 15 22 11 13 2 9 22 2"/>`,
    'shopping-cart': `<circle cx="9" cy="21" r="1"/><circle cx="20" cy="21" r="1"/><path d="M1 1h4l2.68 13.39a2 2 0 0 0 2 1.61h9.72a2 2 0 0 0 2-1.61L23 6H6"/>`,
    'git-branch': `<line x1="6" y1="3" x2="6" y2="15"/><circle cx="18" cy="6" r="3"/><circle cx="6" cy="18" r="3"/><path d="M18 9a9 9 0 0 1-9 9"/>`,
    film: `<rect x="2" y="2" width="20" height="20" rx="2.18"/><line x1="7" y1="2" x2="7" y2="22"/><line x1="17" y1="2" x2="17" y2="22"/><line x1="2" y1="12" x2="22" y2="12"/><line x1="2" y1="7" x2="7" y2="7"/><line x1="17" y1="7" x2="22" y2="7"/><line x1="17" y1="17" x2="22" y2="17"/><line x1="2" y1="17" x2="7" y2="17"/>`,
    sparkles: `<path d="M12 3l1.5 4.5L18 9l-4.5 1.5L12 15l-1.5-4.5L6 9l4.5-1.5L12 3z"/><path d="M5 17l.75 2.25L8 20l-2.25.75L5 23l-.75-2.25L2 20l2.25-.75L5 17z"/>`,
    arrow: `<line x1="4" y1="12" x2="20" y2="12"/><polyline points="14 6 20 12 14 18"/>`,
  };

  function icon(name, size = 14) {
    return `<svg width="${size}" height="${size}" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round">${icons[name] || ''}</svg>`;
  }
</script>

<svelte:head>
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  
</svelte:head>

<div class="app-shell">

  <!-- ── Sidebar ──────────────────────────────────────── -->
  <aside class="sidebar">
    <div class="sidebar-brand">
      <div class="brand-name">ClientFlow</div>
      <div class="brand-sub">Admin Console</div>
    </div>

    <nav class="sidebar-nav">
      {#each navItems as item}
        <div
          class="nav-item {activeNav === item.label ? 'active' : ''}"
          on:click={() => activeNav = item.label}
          role="button"
          tabindex="0"
          on:keydown={(e) => e.key === 'Enter' && (activeNav = item.label)}
        >
          <span class="nav-icon">{@html icon(item.icon, 15)}</span>
          {item.label}
        </div>
      {/each}
    </nav>

    <div class="sidebar-footer">
      {#each footerItems as item}
        <div class="nav-item" role="button" tabindex="0">
          <span class="nav-icon">{@html icon(item.icon, 15)}</span>
          {item.label}
        </div>
      {/each}

      <div style="padding: 10px 0 0;">
        <button class="add-client-btn">
          {@html icon('plus', 12)}
          Add Client
        </button>
      </div>
    </div>
  </aside>

  <!-- ── Main Content ────────────────────────────────── -->
  <main class="main-content">

    <!-- Topbar -->
    <header class="topbar">
      <h1 class="page-title">Overview</h1>
      <div class="topbar-actions">
        <button class="icon-btn">{@html icon('search', 16)}</button>
        <button class="icon-btn">{@html icon('bell', 16)}</button>
        <div class="avatar">AD</div>
      </div>
    </header>

    <!-- Dashboard Body -->
    <div class="dashboard-body">

      <!-- Projects Panel -->
      <section class="projects-panel">
        <div class="panel-header">
          <span class="panel-label">Active Projects</span>
          <span class="panel-count">4 Total</span>
        </div>

        <div class="project-list">
          {#each projects as p}
            <div class="project-card">
              <div class="project-card-header">
                <span class="project-name">{p.name}</span>
                <button class="project-menu-btn">{@html icon('more-vertical', 14)}</button>
              </div>
              <div class="project-meta">
                <span class="project-phase">{p.phase}</span>
                <span class="project-pct">{p.pct}%</span>
              </div>
              <div class="progress-bar">
                <div
                  class="progress-fill {p.pct < 20 ? 'low' : ''}"
                  style="width: {p.pct}%"
                ></div>
              </div>
            </div>
          {/each}
        </div>
      </section>

      <!-- Right: Graph + Operations stacked -->
      <div style="display: flex; flex-direction: column; overflow: hidden;">

        <!-- Project Graph -->
        <section class="project-graph" style="flex: 1; overflow: auto;">
          <div class="section-title">Project Graph</div>
          <div class="section-sub">Workspace / Visual Pipeline</div>

          <div class="graph-canvas">
            <!-- SVG Connectors -->
            <svg class="connectors" xmlns="http://www.w3.org/2000/svg">
              <!-- Source 1 → Shoot Phase -->
              <path
                d="M 178 72 C 210 72 218 72 228 72"
                stroke="#2a2a2a" stroke-width="1.5" fill="none"
              />
              <!-- Shoot Phase → Post Production -->
              <path
                d="M 408 90 C 440 90 450 180 458 180"
                stroke="#2a2a2a" stroke-width="1.5" fill="none"
              />
              <!-- Source 2 → somewhere (faded) -->
              <path
                d="M 178 260 C 210 260 215 220 228 200"
                stroke="#222" stroke-width="1.5" fill="none"
              />
            </svg>

            <!-- Source Node 1: Starlight Media -->
            <div class="graph-node node-source-1">
              <div class="node-header">
                <span class="node-header-label">Project Source</span>
              </div>
              <div class="node-body">
                <div class="node-title">Starlight Media</div>
                <div class="node-id">ID: PRJ-0822</div>
              </div>
              <div class="node-connector"></div>
            </div>

            <!-- Source Node 2: Nordic Design -->
            <div class="graph-node node-source-2">
              <div class="node-header">
                <span class="node-header-label">Project Source</span>
              </div>
              <div class="node-body">
                <div class="node-title">Nordic Design</div>
                <div class="node-id">ID: PRJ-0911</div>
              </div>
              <div class="node-connector"></div>
            </div>

            <!-- Shoot Phase Node -->
            <div class="graph-node node-shoot">
              <div class="node-header">
                <span class="node-header-label">Shoot Phase</span>
              </div>
              <div class="node-body">
                <div class="node-tag">
                  {@html icon('film', 11)}
                  4:2:2 10-bit
                </div>
                <div class="node-progress">
                  <div class="node-progress-fill" style="width: 65%;"></div>
                </div>
              </div>
              <div class="node-connector left"></div>
              <div class="node-connector"></div>
            </div>

            <!-- Post-Production Node -->
            <div class="graph-node node-post">
              <div class="node-header">
                <span class="node-header-label">Post-Production</span>
              </div>
              <div class="node-body">
                <div class="node-tag">
                  {@html icon('sparkles', 11)}
                  VFX Pass
                </div>
              </div>
              <div class="node-connector left"></div>
            </div>

          </div>
        </section>

        <!-- Operations Center -->
        <section class="operations-center">
          <div class="ops-header">
            <div class="ops-title-block">
              <div class="ops-title">Operations<br/>Center</div>
              <div class="ops-desc">Manage multi-channel automation and team communications</div>
            </div>
            <div class="ops-badge">Active<br/>System</div>
          </div>

          <div class="ops-actions">
            <button class="ops-action-btn">
              {@html icon('send', 12)}
              Mass Text on Telegram
            </button>
            <button class="ops-action-btn secondary">
              {@html icon('shopping-cart', 12)}
              Restock Assets
            </button>
          </div>

          <div class="ops-bottom-row">
            <!-- Automation flow -->
            <div class="auto-card">
              <div class="card-label">Automation Logic: Onboarding</div>
              <div class="auto-flow">
                {#each flowSteps as step, i}
                  <div class="flow-step">
                    <div class="flow-icon">{@html icon(step.icon, 16)}</div>
                    <div class="flow-label">{step.label}</div>
                  </div>
                  {#if i < flowSteps.length - 1}
                    <div class="flow-arrow">{@html icon('arrow', 14)}</div>
                  {/if}
                {/each}
              </div>
            </div>

            <!-- Employee Conversations -->
            <div class="convo-card">
              <div class="card-label">Employee Conversations</div>
              <div class="convo-item">
                <div class="convo-avatar">JD</div>
                <div class="convo-info">
                  <div class="convo-name">John Doe</div>
                  <div class="convo-preview">"The color grading on the..."</div>
                </div>
              </div>
              <button class="view-all-btn">View All Channels</button>
            </div>
          </div>
        </section>

      </div>
    </div>
  </main>
</div>
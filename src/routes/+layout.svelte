<script>
  import { goto } from '$app/navigation';
  import { page } from '$app/state';
  import './homePage/homePage.css';

  let activeNav = 'Dashboard';

const navItems = [
  { label: 'Dashboard', icon: 'grid', route: '/homePage' },
  { label: 'Clients', icon: 'users', route: '/clientsPage' },
  { label: 'Calendar', icon: 'calendar', route: '/calendarPage' },
  { label: 'Automation', icon: 'cpu' },
  { label: 'Analytics', icon: 'bar-chart', route: '/analyticsPage' },
];

  const footerItems = [
    { label: 'Settings', icon: 'settings' },
    { label: 'Logout',   icon: 'log-out' },
  ];

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
  };

  function icon(name, size = 14) {
    return `<svg width="${size}" height="${size}" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round">${icons[name] || ''}</svg>`;
  }

  function handleNav(item) {
    activeNav = item.label;

    if (item.route) {
      goto(item.route);
    }
  }
</script>

<div class="app-shell">

  <aside class="sidebar">
    <div class="sidebar-brand">
      <div class="brand-name">ClientFlow</div>
      <div class="brand-sub">Admin Console</div>
    </div>

    <nav class="sidebar-nav">
      {#each navItems as item}
        <div
          class="nav-item {activeNav === item.label ? 'active' : ''}"
          on:click={() => handleNav(item)}
          role="button"
          tabindex="0"
          on:keydown={(e) => e.key === 'Enter' && handleNav(item)}
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
          +
          Add Client
        </button>
      </div>
    </div>
  </aside>

  <main class="main-content">

    <header class="topbar">
      <h1 class="page-title">Overview</h1>

      <div class="topbar-actions">
        <button class="icon-btn">{@html icon('search', 16)}</button>
        <button class="icon-btn">{@html icon('bell', 16)}</button>
        <div class="avatar">AD</div>
      </div>
    </header>

    <slot />
  </main>
</div>
<script>
  import './automationPage.css';
  import { onMount } from 'svelte';

  let activeTab = 'Timeline';
  const tabs = ['Timeline', 'Post_Production'];

  /* ── Node graph state ── */
  let nodes = [
    {
      id: 'system',
      type: 'SYSTEM',
      title: 'SmartCache',
      sub: 'Location: /SSD_CACHE/01',
      status: 'Active Adjustment',
      statusDot: 'active',
      x: 30, y: 30,
    },
    {
      id: 'source',
      type: 'SOURCE',
      title: 'Master_Footage.r3d',
      sub: '4K_LOG_RAW',
      status: null,
      statusDot: 'pending',
      x: 210, y: 160,
    },
  ];

  const outputNodes = [
    { id: 'storage',       icon: 'server',        label: 'Local Storage',   sub: '4TB NVMe Raid',   x: 370, y: 280 },
    { id: 'ai',            icon: 'brain',          label: 'AI Scripting',    sub: 'GPT-4o Analysis', x: 510, y: 280 },
    { id: 'transcription', icon: 'language',       label: 'Transcription',   sub: 'Whisper v3 Large',x: 650, y: 280 },
    { id: 'slack',         icon: 'brand-slack',    label: 'Slack Sync',      sub: 'Webhooks/v2',     x: 790, y: 280 },
  ];

  /* Dragging */
  let dragging = null;
  let dragOffset = { x: 0, y: 0 };
  let canvasEl;

  function startDrag(e, id) {
    const node = nodes.find(n => n.id === id);
    if (!node) return;
    dragging = id;
    const rect = canvasEl.getBoundingClientRect();
    dragOffset.x = e.clientX - rect.left - node.x;
    dragOffset.y = e.clientY - rect.top  - node.y;
    window.addEventListener('mousemove', onDrag);
    window.addEventListener('mouseup', stopDrag);
  }

  function onDrag(e) {
    if (!dragging) return;
    const rect = canvasEl.getBoundingClientRect();
    const x = Math.max(0, e.clientX - rect.left - dragOffset.x);
    const y = Math.max(0, e.clientY - rect.top  - dragOffset.y);
    nodes = nodes.map(n => n.id === dragging ? { ...n, x, y } : n);
  }

  function stopDrag() {
    dragging = null;
    window.removeEventListener('mousemove', onDrag);
    window.removeEventListener('mouseup', stopDrag);
  }

  /* SVG connector paths */
  function edgePath(x1, y1, x2, y2) {
    const cx = (x1 + x2) / 2;
    return `M ${x1} ${y1} C ${cx} ${y1}, ${cx} ${y2}, ${x2} ${y2}`;
  }

  const NODE_W = 160;
  const NODE_H_SYS = 90;
  const NODE_H_SRC = 70;
  const ICON_W  = 72;
  const ICON_H  = 72;

  $: systemNode = nodes.find(n => n.id === 'system');
  $: sourceNode = nodes.find(n => n.id === 'source');

  /* Sidebar sequences */
  const sequences = [
    { id: 'VIDEO_PROCESSING_V1', label: 'VIDEO_PROCESSING_V1', active: true  },
    { id: 'AUDIO_SYNC_V2',       label: 'AUDIO_SYNC_V2',       active: false },
    { id: 'COLOR_GRADE_V3',      label: 'COLOR_GRADE_V3',      active: false },
  ];
</script>

<div class="auto-page">

  <!-- ── Sidebar ── -->
  <aside class="auto-sidebar">
    <div class="auto-sidebar-logo">
      <span class="auto-logo-text">AUTOMA_CORE</span>
      <span class="auto-logo-sub">WORKFLOWS</span>
    </div>

    <nav class="auto-nav">
      {#each [
        { label: 'DASHBOARD',       icon: 'layout-grid' },
        { label: 'NODES',           icon: 'circle-dot' },
        { label: 'GRAPH_EDITOR',    icon: 'vector-bezier-2', active: true },
        { label: 'PROCESSING_LOG',  icon: 'terminal-2' },
      ] as item}
        <div class="auto-nav-item {item.active ? 'active' : ''}">
          <i class="ti ti-{item.icon}" aria-hidden="true"></i>
          <span>{item.label}</span>
        </div>
      {/each}
    </nav>

    <div class="auto-nav-footer">
      <div class="auto-nav-item">
        <i class="ti ti-settings" aria-hidden="true"></i>
        <span>CONFIGURATION</span>
      </div>
    </div>

    <button class="new-sequence-btn">NEW_SEQUENCE</button>
  </aside>

  <!-- ── Main ── -->
  <div class="auto-main">

    <!-- Topbar -->
    <header class="auto-topbar">
      <div class="auto-breadcrumb">
        <span class="breadcrumb-root">WORKFLOWS</span>
        <span class="breadcrumb-sep">›</span>
        <span class="breadcrumb-current">VIDEO_PROCESSING_V1</span>
      </div>

      <div class="auto-search">
        <i class="ti ti-search" aria-hidden="true"></i>
        <input type="text" placeholder="Find processing nodes…" />
      </div>

      <div class="auto-topbar-right">
        {#each tabs as tab}
          <button
            class="tab-btn {activeTab === tab ? 'active' : ''}"
            on:click={() => activeTab = tab}
          >{tab}</button>
        {/each}
        <button class="icon-btn-top"><i class="ti ti-bell" aria-hidden="true"></i></button>
        <button class="icon-btn-top"><i class="ti ti-adjustments-horizontal" aria-hidden="true"></i></button>
        <div class="auto-avatar">AD</div>
      </div>
    </header>

    <!-- Canvas -->
    <div class="canvas-wrap" bind:this={canvasEl}>

      <!-- SVG edges -->
      <svg class="edges-svg" style="position:absolute;inset:0;width:100%;height:100%;pointer-events:none;overflow:visible;">
        <!-- system → source -->
        <path
          d={edgePath(
            systemNode.x + NODE_W,     systemNode.y + NODE_H_SYS / 2,
            sourceNode.x,              sourceNode.y + NODE_H_SRC / 2
          )}
          fill="none" stroke="rgba(255,255,255,0.15)" stroke-width="1" stroke-dasharray="4 4"
        />
        <!-- source → each output node -->
        {#each outputNodes as out}
          <path
            d={edgePath(
              sourceNode.x + NODE_W,   sourceNode.y + NODE_H_SRC / 2,
              out.x + ICON_W / 2,      out.y
            )}
            fill="none" stroke="rgba(255,255,255,0.1)" stroke-width="1" stroke-dasharray="4 4"
          />
        {/each}
      </svg>

      <!-- Draggable nodes -->
      {#each nodes as node (node.id)}
        <div
          class="graph-node {node.id} {dragging === node.id ? 'dragging' : ''}"
          style="left:{node.x}px; top:{node.y}px;"
          on:mousedown={(e) => startDrag(e, node.id)}
          role="button"
          tabindex="0"
          aria-label="Node {node.title}"
        >
          <div class="gn-header">
            <i class="ti ti-{node.id === 'system' ? 'cpu' : 'device-desktop-analytics'}" aria-hidden="true"></i>
            <span class="gn-type">{node.type}</span>
            <div class="gn-dot {node.statusDot}"></div>
          </div>
          <div class="gn-body">
            <div class="gn-title">{node.title}</div>
            <div class="gn-sub">{node.sub}</div>
            {#if node.status}
              <div class="gn-status">
                <span class="status-pulse"></span>
                {node.status}
              </div>
            {/if}
          </div>
          <!-- connector handle right -->
          <div class="cn-handle right"></div>
          <!-- connector handle left -->
          <div class="cn-handle left"></div>
        </div>
      {/each}

      <!-- Static output nodes -->
      {#each outputNodes as out}
        <div class="output-node" style="left:{out.x}px; top:{out.y}px;">
          <div class="on-icon">
            <i class="ti ti-{out.icon}" aria-hidden="true"></i>
          </div>
          <div class="on-label">{out.label}</div>
          <div class="on-sub">{out.sub}</div>
        </div>
      {/each}

    </div>

    <!-- Toolbar -->
    <div class="canvas-toolbar">
      <button class="toolbar-btn"><i class="ti ti-zoom-out" aria-hidden="true"></i></button>
      <button class="toolbar-btn"><i class="ti ti-zoom-in" aria-hidden="true"></i></button>
      <button class="toolbar-btn"><i class="ti ti-maximize" aria-hidden="true"></i></button>

      <button class="execute-btn">
        <i class="ti ti-player-play" aria-hidden="true"></i>
        EXECUTE_FLOW
      </button>

      <button class="toolbar-btn"><i class="ti ti-plus" aria-hidden="true"></i></button>
    </div>

    <!-- Status bar -->
    <div class="status-bar">
      <div class="status-left">
        <i class="ti ti-terminal-2" aria-hidden="true"></i>
        <span>KERNEL_TERMINAL</span>
      </div>
      <div class="status-center">
        <span class="status-dot-green"></span>
        <span>STATUS: <strong>IDLE_LISTENING</strong></span>
      </div>
      <div class="status-right">
        MEM: 12.4GB / 64GB
        <i class="ti ti-chevron-up" aria-hidden="true"></i>
      </div>
    </div>

  </div>
</div>
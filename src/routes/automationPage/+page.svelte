<script>
  import './automationPage.css';

  /* ── layout constants ── */
  const TG_X  = 40;
  const TG_W  = 195;
  const TG_H  = 74;
  const NODE_W = 178;
  const NODE_H = 82;

  /* ── triggers (fixed, non-draggable) ── */
  const triggers = [
    { id: 'tg_telegram', icon: 'brand-telegram',     label: 'Telegram Message', sub: 'On new message',      y: 30  },
    { id: 'tg_social',   icon: 'message-dots',        label: 'Socials Comment',  sub: 'Instagram · TikTok',  y: 120 },
    { id: 'tg_deadline', icon: 'clock-exclamation',   label: 'Deadline Trigger', sub: '48 h before due',     y: 210 },
    { id: 'tg_calendar', icon: 'calendar-event',      label: 'Calendar Event',   sub: 'Scheduled trigger',   y: 300 },
    { id: 'tg_add',      icon: 'plus',                label: 'Add Trigger',      sub: null,                  y: 402 },
  ];

  /* ── flow nodes (fixed positions, not draggable) ── */
  const nodes = [
    { id: 'tg_resp',    type: 'ACTION',  icon: 'send',             title: 'Telegram Response', sub: 'Auto-reply template',  x: 310,  y: 18  },
    { id: 'shoot',      type: 'ACTION',  icon: 'camera',           title: 'Shoot',             sub: 'Production trigger',   x: 310,  y: 200 },
    { id: 'cam',        type: 'SERVICE', icon: 'device-camera',    title: 'Camera Rental',     sub: 'BookingKit API',       x: 560,  y: 130 },
    { id: 'txt',        type: 'SERVICE', icon: 'file-description', title: 'Text Editor',       sub: 'Script & captions',    x: 560,  y: 270 },
    { id: 'approve',    type: 'REVIEW',  icon: 'circle-check',     title: 'Approve Footage',   sub: 'Manual review gate',   x: 810,  y: 200 },
    { id: 'post',       type: 'OUTPUT',  icon: 'rocket',           title: 'Post',              sub: 'Publish to channels',  x: 1060, y: 200 },
  ];

  /* ── bezier helper ── */
  function bez(x1, y1, x2, y2) {
    const cx = (x1 + x2) / 2;
    return `M${x1} ${y1} C${cx} ${y1},${cx} ${y2},${x2} ${y2}`;
  }

  /* shorthand edge coords */
  const r  = (nd) => [nd.x + NODE_W,      nd.y + NODE_H / 2];
  const l  = (nd) => [nd.x,               nd.y + NODE_H / 2];
  const tr = (tg) => [TG_X + TG_W,        tg.y + TG_H / 2];

  const nd = Object.fromEntries(nodes.map(n => [n.id, n]));
  const tg = Object.fromEntries(triggers.map(t => [t.id, t]));

  /* ── bottom drawer ── */
  const automations = [
    { id: 1, name: 'Telegram → Post Flow',  status: 'running',  nodes: 6, lastRun: '2 min ago',  needs: false },
    { id: 2, name: 'Deadline Shoot Trigger', status: 'approval', nodes: 4, lastRun: '14 min ago', needs: true  },
    { id: 3, name: 'Calendar Sync',          status: 'idle',     nodes: 2, lastRun: '1 h ago',    needs: false },
    { id: 4, name: 'Social Auto-Reply',      status: 'approval', nodes: 3, lastRun: '28 min ago', needs: true  },
  ];

  /* ── right panel ── */
  let panelOpen = true;
  let panelTab  = 'notes'; // 'notes' | 'reply' | 'call'
  let noteText  = '';
  let replyText = '';
  let calling   = false;
</script>

<div class="auto-page">

  <!-- ══════════════════════════════════════════════
       TOP STRIP: breadcrumb + toolbar
  ═══════════════════════════════════════════════ -->
  <div class="top-strip">
    <div class="breadcrumb">
      <span class="bc-root">WORKFLOWS</span>
      <span class="bc-sep">›</span>
      <span class="bc-cur">VIDEO_PROCESSING_V1</span>
    </div>

    <div class="strip-center">
      <span class="strip-badge">
        <span class="strip-dot"></span>
        IDLE_LISTENING
      </span>
      <span class="strip-stat">6 NODES</span>
      <span class="strip-stat">8 EDGES</span>
    </div>

    <div class="strip-right">
      <button class="tb-btn"><i class="ti ti-zoom-out" aria-hidden="true"></i></button>
      <button class="tb-btn"><i class="ti ti-zoom-in"  aria-hidden="true"></i></button>
      <button class="tb-btn"><i class="ti ti-maximize" aria-hidden="true"></i></button>
      <div class="tb-sep"></div>
      <button class="execute-btn">
        <i class="ti ti-player-play" aria-hidden="true"></i>
        EXECUTE_FLOW
      </button>
      <button class="tb-btn tb-btn--templates" on:click={() => panelOpen = true}>
        <i class="ti ti-layout-grid" aria-hidden="true"></i>
        Browse Templates
      </button>
    </div>
  </div>

  <!-- ══════════════════════════════════════════════
       BODY: canvas + right panel
  ═══════════════════════════════════════════════ -->
  <div class="body-row">

    <!-- ── CANVAS ── -->
    <div class="canvas-wrap">
      <div class="canvas-inner">

        <!-- SVG EDGES -->
        <svg class="edges-svg">
          <!-- telegram → tg_resp -->
          <path d={bez(...tr(tg.tg_telegram), ...l(nd.tg_resp))}    class="edge edge-solid" />
          <!-- deadline → shoot -->
          <path d={bez(...tr(tg.tg_deadline), ...l(nd.shoot))}      class="edge edge-dash"  />
          <!-- shoot → cam -->
          <path d={bez(...r(nd.shoot), ...l(nd.cam))}               class="edge edge-dash"  />
          <!-- shoot → txt -->
          <path d={bez(...r(nd.shoot), ...l(nd.txt))}               class="edge edge-dash"  />
          <!-- cam → approve -->
          <path d={bez(...r(nd.cam),   ...l(nd.approve))}           class="edge edge-dim"   />
          <!-- txt → approve -->
          <path d={bez(...r(nd.txt),   ...l(nd.approve))}           class="edge edge-dim"   />
          <!-- approve → post -->
          <path d={bez(...r(nd.approve), ...l(nd.post))}            class="edge edge-solid" />
          <!-- tg_resp → approve (dashed secondary) -->
          <path d={bez(nd.tg_resp.x + NODE_W, nd.tg_resp.y + NODE_H / 2,
                       nd.approve.x, nd.approve.y + 18)}            class="edge edge-dim"   />

          <!-- feedback arc: approve → telegram trigger -->
          <path
            d={`M${r(nd.approve)[0]} ${r(nd.approve)[1] - 8}
                C${r(nd.approve)[0] + 55} ${r(nd.approve)[1] - 8},
                 ${r(nd.approve)[0] + 55} ${r(nd.approve)[1] - 100},
                 ${r(nd.approve)[0]}      ${r(nd.approve)[1] - 100}
                L${tr(tg.tg_telegram)[0] + 12} ${r(nd.approve)[1] - 100}
                C${tr(tg.tg_telegram)[0] - 30} ${r(nd.approve)[1] - 100},
                 ${tr(tg.tg_telegram)[0] - 30} ${tr(tg.tg_telegram)[1]},
                 ${tr(tg.tg_telegram)[0]}       ${tr(tg.tg_telegram)[1]}`}
            class="edge edge-feedback" />
          <text
            x={(r(nd.approve)[0] + tr(tg.tg_telegram)[0]) / 2 + 20}
            y={r(nd.approve)[1] - 107}
            class="edge-lbl">REVISE</text>

          <!-- branch labels as SVG text -->
          <text x={nd.shoot.x + NODE_W + 16} y={nd.shoot.y + 14}  class="branch-lbl">BRANCH A</text>
          <text x={nd.shoot.x + NODE_W + 16} y={nd.shoot.y + NODE_H + 4} class="branch-lbl">BRANCH B</text>
        </svg>

        <!-- TRIGGER COLUMN -->
        <div class="trigger-col" style="left:{TG_X}px; top:14px;">
          <div class="col-label">
            <span class="pulse-dot"></span>
            TRIGGERS
          </div>

          {#each triggers as trig}
            <div
              class="trigger-node {trig.sub === null ? 'trig-ghost' : ''}"
              style="top:{trig.y}px;"
            >
              {#if trig.sub !== null}
                <div class="tn-icon">
                  <i class="ti ti-{trig.icon}" aria-hidden="true"></i>
                </div>
                <div class="tn-body">
                  <div class="tn-label">{trig.label}</div>
                  <div class="tn-sub">{trig.sub}</div>
                </div>
                <div class="tn-dot"></div>
                <div class="cn-handle right"></div>
              {:else}
                <i class="ti ti-plus" aria-hidden="true"></i>
                <span class="tn-add-label">Add Trigger</span>
              {/if}
            </div>
          {/each}
        </div>

        <!-- FLOW NODES -->
        {#each nodes as node (node.id)}
          <div
            class="flow-node {node.id === 'approve' ? 'fn-gate' : ''} {node.id === 'post' ? 'fn-live' : ''}"
            style="left:{node.x}px; top:{node.y}px; width:{NODE_W}px;"
          >
            <div class="fn-head">
              <div class="fn-icon">
                <i class="ti ti-{node.icon}" aria-hidden="true"></i>
              </div>
              <div class="fn-titles">
                <div class="fn-type">{node.type}</div>
                <div class="fn-name">{node.title}</div>
              </div>
              {#if node.id === 'approve'}
                <div class="fn-pill">GATE</div>
              {:else if node.id === 'post'}
                <div class="fn-pill fn-pill--live">LIVE</div>
              {:else}
                <div class="fn-status-dot"></div>
              {/if}
            </div>
            <div class="fn-sub">{node.sub}</div>
            <div class="cn-handle right"></div>
            <div class="cn-handle left"></div>
          </div>
        {/each}

      </div><!-- /canvas-inner -->
    </div><!-- /canvas-wrap -->

    <!-- ── RIGHT PANEL ── -->
    {#if panelOpen}
      <div class="right-panel">
        <div class="rp-header">
          <div class="rp-tabs">
            <button class="rp-tab {panelTab === 'notes' ? 'active' : ''}" on:click={() => panelTab = 'notes'}>
              <i class="ti ti-notes" aria-hidden="true"></i> Notes
            </button>
            <button class="rp-tab {panelTab === 'reply' ? 'active' : ''}" on:click={() => panelTab = 'reply'}>
              <i class="ti ti-message-reply" aria-hidden="true"></i> Reply
            </button>
            <button class="rp-tab {panelTab === 'call' ? 'active' : ''}" on:click={() => panelTab = 'call'}>
              <i class="ti ti-phone" aria-hidden="true"></i> Call
            </button>
          </div>
          <button class="rp-close" on:click={() => panelOpen = false}>
            <i class="ti ti-x" aria-hidden="true"></i>
          </button>
        </div>

        <!-- NOTES -->
        {#if panelTab === 'notes'}
          <div class="rp-body">
            <div class="rp-section-label">INTERNAL NOTES</div>
            <div class="rp-note-entry">
              <div class="rp-note-avatar">AD</div>
              <div class="rp-note-content">
                <div class="rp-note-who">You · <span>3 min ago</span></div>
                <div class="rp-note-text">Camera rental confirmed for Friday. Awaiting footage upload from Alex.</div>
              </div>
            </div>
            <div class="rp-note-entry">
              <div class="rp-note-avatar">JR</div>
              <div class="rp-note-content">
                <div class="rp-note-who">J. Reeves · <span>1 h ago</span></div>
                <div class="rp-note-text">Script draft is in text editor. Needs review before deadline trigger fires.</div>
              </div>
            </div>
            <textarea
              class="rp-textarea"
              placeholder="Add a note…"
              bind:value={noteText}
              rows="3"
            ></textarea>
            <button class="rp-send-btn" disabled={!noteText.trim()}>
              <i class="ti ti-send" aria-hidden="true"></i> Post Note
            </button>
          </div>
        {/if}

        <!-- REPLY -->
        {#if panelTab === 'reply'}
          <div class="rp-body">
            <div class="rp-section-label">RECENT MESSAGE</div>
            <div class="rp-msg-bubble">
              <div class="rp-msg-meta">
                <span class="rp-msg-from">@client_handle</span>
                <span class="rp-msg-time">via Telegram · 8 min ago</span>
              </div>
              <div class="rp-msg-text">"Hey, is the footage from yesterday ready for review?"</div>
            </div>
            <div class="rp-section-label" style="margin-top:14px;">REPLY</div>
            <textarea
              class="rp-textarea"
              placeholder="Type a reply…"
              bind:value={replyText}
              rows="3"
            ></textarea>
            <div class="rp-reply-actions">
              <button class="rp-send-btn" disabled={!replyText.trim()}>
                <i class="ti ti-send" aria-hidden="true"></i> Send Reply
              </button>
            </div>
          </div>
        {/if}

        <!-- CALL -->
        {#if panelTab === 'call'}
          <div class="rp-body rp-body--call">
            <div class="rp-section-label">CONTACT</div>
            <div class="rp-contact-row">
              <div class="rp-contact-avatar">CL</div>
              <div>
                <div class="rp-contact-name">Client Handle</div>
                <div class="rp-contact-sub">+1 (555) 012-3456</div>
              </div>
            </div>
            {#if calling}
              <div class="rp-calling">
                <div class="rp-calling-ring">
                  <i class="ti ti-phone" aria-hidden="true"></i>
                </div>
                <div class="rp-calling-label">Calling…</div>
                <button class="rp-end-btn" on:click={() => calling = false}>
                  <i class="ti ti-phone-off" aria-hidden="true"></i> End Call
                </button>
              </div>
            {:else}
              <button class="rp-call-btn" on:click={() => calling = true}>
                <i class="ti ti-phone" aria-hidden="true"></i>
                Start Call
              </button>
              <div class="rp-call-note">Uses system dialer or VoIP integration</div>
            {/if}
          </div>
        {/if}

        <!-- TEMPLATES BUTTON -->
        <div class="rp-footer">
          <button class="rp-templates-btn">
            <i class="ti ti-layout-grid" aria-hidden="true"></i>
            Browse Templates
          </button>
        </div>
      </div>
    {:else}
      <!-- Collapsed tab -->
      <button class="panel-toggle" on:click={() => panelOpen = true}>
        <i class="ti ti-message-circle" aria-hidden="true"></i>
      </button>
    {/if}

  </div><!-- /body-row -->

  <!-- ══════════════════════════════════════════════
       BOTTOM DRAWER
  ═══════════════════════════════════════════════ -->
  <div class="bottom-drawer">
    <div class="bd-header">
      <div class="bd-title">
        <i class="ti ti-list-check" aria-hidden="true"></i>
        Ongoing Automations
      </div>
      <div class="bd-badges">
        <span class="bd-badge bd-badge--approval">2 Need Approval</span>
        <span class="bd-badge">4 Active</span>
      </div>
    </div>

    <div class="bd-list">
      {#each automations as a}
        <div class="bd-row {a.needs ? 'bd-row--urgent' : ''}">
          <div class="bd-status-dot {a.status}"></div>
          <div class="bd-name">{a.name}</div>
          <div class="bd-meta">{a.nodes} nodes</div>
          <div class="bd-meta">{a.lastRun}</div>
          <div class="bd-status-tag {a.status}">{a.status}</div>
          {#if a.needs}
            <button class="bd-approve-btn">Approve</button>
          {:else}
            <div class="bd-spacer"></div>
          {/if}
        </div>
      {/each}
    </div>
  </div>

</div>
<script>
  import Search from "@lucide/svelte/icons/search";
  import ArrowRight from "@lucide/svelte/icons/arrow-right";
  import Command from "@lucide/svelte/icons/command";
  import Globe2 from "@lucide/svelte/icons/globe-2";
  import ShieldCheck from "@lucide/svelte/icons/shield-check";
  import Zap from "@lucide/svelte/icons/zap";

  import Select from "./Select.svelte";
  import { BACKEND_OPTIONS, LOCATION_OPTIONS } from "../lib/backends.js";
  import { settings } from "../lib/settings.svelte.js";
  import { activeTab, activeUrl, open } from "../lib/tabs.svelte.js";
  import { resolve } from "../lib/url.js";

  const GOOD_MS = 100;
  const FAIR_MS = 300;

  let query = $state("");
  let latency = $state(null);

  const visible = $derived(activeTab()?.kind === "proxy" && !activeUrl());
  const label = $derived(
    latency === null ? "Checking connection…" : latency === "error" ? "Connection unavailable" : `${latency} ms`,
  );
  const level = $derived.by(() => {
    if (typeof latency !== "number") return latency === "error" ? "poor" : "";
    return latency < GOOD_MS ? "good" : latency < FAIR_MS ? "fair" : "poor";
  });

  $effect(() => {
    if (visible) measure();
  });

  async function measure() {
    latency = null;
    const started = performance.now();
    try {
      const response = await fetch("/ping", { cache: "no-store" });
      if (!response.ok) throw new Error(response.status);
      latency = Math.round(performance.now() - started);
    } catch {
      latency = "error";
    }
  }

  function navigate(value = query) {
    const target = resolve(value, settings.search);
    if (target) open(target);
    query = "";
  }

  function keydown(event) {
    if (event.key === "Enter") navigate();
  }

  function useExample(value) {
    query = value;
    navigate(value);
  }
</script>

<section class="page" id="startPage" class:active={visible}>
  <div class="startGrid"></div>
  <div class="startOrb orbOne"></div>
  <div class="startOrb orbTwo"></div>

  <div class="inner">
    <div class="heroBadge"><span></span> GAME TFM • READY</div>

    <div class="brandMark">
      <img src="/gametfm-logo.jpg" alt="GameTFM" />
    </div>

    <div class="brandBlock">
      <h1 class="brandTitle"><span>Game</span>TFM</h1>
      <p class="brandSubtitle">A faster, cleaner way to browse.</p>
    </div>

    <div id="startBar" class="heroSearch">
      <Search />
      <input
        id="startSearch"
        spellcheck="false"
        autocomplete="off"
        placeholder="Search or enter a web address"
        bind:value={query}
        onkeydown={keydown}
      />
      <button class="searchGo" aria-label="Go" onclick={() => navigate()}>
        <ArrowRight />
      </button>
    </div>

    <div class="quickRow" aria-label="Quick launch">
      <button class="quickChip" onclick={() => useExample("https://example.com")}>
        <Globe2 /> Example
      </button>
      <button class="quickChip" onclick={() => useExample("https://www.google.com")}>
        <Search /> Search
      </button>
      <div class="shortcutChip"><Command /> Enter to go</div>
    </div>

    <div id="startControls">
      <div class="controlCard">
        <div class="controlIcon"><Zap /></div>
        <div class="controlMeta">
          <span class="controlLabel">Backend</span>
          <span class="controlHint">Connection engine</span>
        </div>
        <Select
          id="setStartBackend"
          options={BACKEND_OPTIONS}
          value={settings.backend}
          onchange={(val) => (settings.backend = val)}
        />
      </div>
      <div class="controlCard">
        <div class="controlIcon"><ShieldCheck /></div>
        <div class="controlMeta">
          <span class="controlLabel">Location</span>
          <span class="controlHint">Routing preference</span>
        </div>
        <Select
          id="setStartLocation"
          options={LOCATION_OPTIONS}
          value={settings.location}
          onchange={(val) => (settings.location = val)}
        />
      </div>
    </div>

    <div class="featureRow">
      <div class="featureCard"><span class="featureDot"></span><b>Fast</b><span>startup</span></div>
      <div class="featureCard"><span class="featureDot"></span><b>Clean</b><span>interface</span></div>
      <div class="featureCard"><span class="featureDot"></span><b>Focused</b><span>workspace</span></div>
    </div>

    <div class="startFooter">GameTFM <span>•</span> built for a simple browsing workflow</div>
  </div>

  <div id="latency" title="Connection status">
    <span class="dot {level}"></span>
    {label}
  </div>
</section>

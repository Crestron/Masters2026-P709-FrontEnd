<svelte:options runes />

<script lang="ts">
  import { untrack } from "svelte";
  import Card from "../ui/Card.svelte";
  import {
    conciergeConfigState,
    conciergeState,
  } from "../../stores/concierge.svelte";
  import {
    sendConciergeRequest,
    startConcierge,
  } from "../../transport/concierge";

  const STATUS_TEXT: Record<string, string> = {
    idle: "Tap to request",
    sending: "Sending request…",
    confirmed: "Request confirmed ✓",
    failed: "Request failed — try again",
  };

  $effect(() => {
    const config = conciergeConfigState.current;
    return untrack(() =>
      startConcierge(
        config ? { ...config } : { apiUrl: "", roomId: "", authToken: "" },
      ),
    );
  });
</script>

<section class="page-body" aria-labelledby="concierge-title">
  <header class="page__header">
    <h1 id="concierge-title">Concierge</h1>
  </header>

  <h2 id="concierge-requests-title">Service requests</h2>
  <section class="concierge" aria-labelledby="concierge-requests-title">
    <Card class="concierge__card">
      {#if conciergeState.status === "loading"}
        <p class="concierge__hint" role="status">Loading concierge services…</p>
      {:else if conciergeState.status !== "online"}
        <p class="concierge__hint" role="alert">{conciergeState.message}</p>
      {:else if conciergeState.requests.length === 0}
        {#if conciergeState.notice}<p role="alert">
            {conciergeState.notice}
          </p>{/if}
        <p class="concierge__hint">
          No concierge services are available for this room.
        </p>
      {:else}
        {#if conciergeState.notice}<p role="alert">
            {conciergeState.notice}
          </p>{/if}
        <p class="concierge__hint">Select a service to send a request.</p>
        <div class="requests" role="group" aria-label="Service requests">
          {#each conciergeState.requests as request (request.uid)}
            <!-- TODO: UI Lab 3.2 - Add the request button -->
            <!-- Hint: The possible request statuses are 'idle', 'sending', 'confirmed', and 'failed' -->
            <!-- UI Lab 3.2 - End -->

            <!-- REF: UI Lab 3.2 -->
            <button
              class="request-button"
              class:request-button--sending={request.status === "sending"}
              class:request-button--confirmed={request.status === "confirmed"}
              class:request-button--failed={request.status === "failed"}
              onclick={() => sendConciergeRequest(request)}
              disabled={request.status !== "idle"}
            >
              {#if request.icon}
                <img
                  class="request-icon"
                  src="data:image/svg+xml;base64,{request.icon}"
                  alt=""
                />
              {/if}
              <span class="request-label">{request.name}</span>
              <span class="request-description">{request.description}</span>
              <span class="request-status" role="status">
                {#if request.status === "sending"}
                  <span class="request-spinner" aria-hidden="true"></span>
                {/if}
                {request.message || STATUS_TEXT[request.status]}
              </span>
            </button>
            <!-- REF: UI Lab 3.2 - End -->
          {/each}
        </div>
      {/if}
    </Card>
  </section>
</section>

<style>
  /* Fill the page (the .page wrapper in PageOutlet is absolutely positioned
     over <main>) as a flex column: the header stays anchored top-left in
     normal flow, and the controls section below grows to take the remaining
     space and centres the card within it. */
  .page-body {
    box-sizing: border-box;
    min-height: 100%;
    max-width: 1200px;
    width: 100%;
    margin: 0 auto;
    padding: 1rem clamp(1rem, 5vw, 2rem);
    display: flex;
    flex-direction: column;
  }

  .page__header {
    flex-shrink: 0;
    margin-bottom: 1rem;
  }
  .page__header h1 {
    margin: 0;
    font-size: 1.5rem;
    letter-spacing: 0.02em;
  }

  h2 {
    margin: 0 0 0.75rem;
    font-size: 0.75rem;
    font-weight: 600;
    text-transform: uppercase;
    letter-spacing: 0.12em;
    opacity: 0.45;
  }

  .concierge {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  /* The Card renders with this forwarded class; :global() is needed because
     a page's scoped CSS can't otherwise reach inside the component. */
  .concierge > :global(.concierge__card) {
    width: 100%;
    max-width: 1100px;
  }

  .concierge__hint {
    margin: 0 0 1rem;
    font-size: 0.95rem;
    opacity: 0.7;
  }

  .requests {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(min(280px, 100%), 1fr));
    gap: 0.75rem;
  }

  .request-button {
    appearance: none;
    font: inherit;
    text-align: left;
    color: inherit;
    background: rgb(var(--surface-hi) / 0.5);
    border: 1px solid rgb(var(--ink) / 0.15);
    border-radius: 10px;
    padding: 0.85rem 1rem;
    cursor: pointer;
    display: flex;
    flex-direction: column;
    gap: 0.3rem;
    transition:
      border-color 200ms,
      background 200ms,
      opacity 200ms,
      transform 80ms;
  }
  .request-button:active:not(:disabled) {
    transform: scale(0.98);
  }
  .request-button:not(:disabled):hover {
    border-color: rgb(var(--accent-rgb) / 0.7);
  }

  /* A slot showing its own progress/result is disabled but NOT greyed out —
     its state colouring below is the feedback. */
  .request-button:disabled {
    cursor: default;
  }

  .request-button--sending {
    border-color: rgb(var(--accent-rgb));
    background: rgb(var(--accent-rgb) / 0.12);
  }

  .request-button--confirmed {
    border-color: #4ade80;
    background: rgb(74 222 128 / 0.1);
  }

  .request-button--failed {
    border-color: #f87171;
    background: rgb(248 113 113 / 0.1);
  }

  .request-label {
    font-weight: 600;
    letter-spacing: 0.01em;
  }

  .request-icon {
    width: 2rem;
    height: 2rem;
    object-fit: contain;
  }
  .request-description {
    font-size: 0.85rem;
    opacity: 0.75;
  }

  .request-status {
    display: flex;
    align-items: center;
    gap: 0.45rem;
    font-size: 0.85rem;
    opacity: 0.65;
  }

  .request-button--confirmed .request-status {
    color: #4ade80;
    opacity: 1;
  }
  .request-button--failed .request-status {
    color: #f87171;
    opacity: 1;
  }

  .request-spinner {
    flex-shrink: 0;
    width: 0.8rem;
    height: 0.8rem;
    border: 2px solid rgb(var(--accent-rgb) / 0.3);
    border-top-color: rgb(var(--accent-rgb));
    border-radius: 50%;
    animation: spin 0.8s linear infinite;
  }

  @keyframes spin {
    to {
      transform: rotate(360deg);
    }
  }
</style>

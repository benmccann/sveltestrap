<script>
  import {
    autoUpdate,
    computePosition,
    flip,
    limitShift,
    shift
  } from '@floating-ui/dom';
  import { onDestroy, onMount } from 'svelte';
  import classnames, { uuid } from './utils';
  import InlineContainer from './InlineContainer.svelte';
  import Portal from './Portal.svelte';

  let className = '';
  export { className as class };
  export let animation = true;
  export let children = undefined;
  export let container = undefined;
  export let id = `tooltip_${uuid()}`;
  export let isOpen = false;
  export let placement = 'top';
  export let target = '';
  let bsPlacement;
  let popperPlacement = placement;
  let referenceEl;
  let floatingEl;
  let cleanup;

  $: {
    if (isOpen && floatingEl) {
      cleanup = autoUpdate(referenceEl, floatingEl, () => {
        computePosition(referenceEl, floatingEl, {
          placement: popperPlacement,
          middleware: [flip(), shift({ limiter: limitShift() })],
        }).then(({x, y}) => {
          Object.assign(floatingEl.style, {
            left: `${x}px`,
            top: `${y}px`,
          });
        })
      });
    } else if (cleanup) {
      cleanup();
      cleanup = undefined;
    }
  }

  const open = () => (isOpen = true);
  const close = () => (isOpen = false);

  onMount(registerEventListeners);
  onDestroy(unregisterEventListeners);

  $: if (target) {
    unregisterEventListeners();
    registerEventListeners();
  }

  function registerEventListeners() {
    if (target == null || target.length == 0) {
      referenceEl = null;
      return;
    }

    // Check if target is HTMLElement
    try {
      if (target instanceof HTMLElement) {
        referenceEl = target;
      }
    } catch (e) {
      // fails on SSR
    }

    // If referenceEl has not been found yet
    if (referenceEl == null) {
      // Check if target can be found via querySelector
      try {
        referenceEl = document.querySelector(`#${target}`);
      } catch (e) {
        // fails on SSR
      }
    }

    // If we've found referenceEl
    if (referenceEl) {
      referenceEl.addEventListener('mouseover', open);
      referenceEl.addEventListener('mouseleave', close);
      referenceEl.addEventListener('focus', open);
      referenceEl.addEventListener('blur', close);
    }
  }

  function unregisterEventListeners() {
    if (referenceEl) {
      referenceEl.removeEventListener('mouseover', open);
      referenceEl.removeEventListener('mouseleave', close);
      referenceEl.removeEventListener('focus', open);
      referenceEl.removeEventListener('blur', close);
      referenceEl.removeAttribute('aria-describedby');
    }
  }

  $: if (referenceEl) {
    if (isOpen) referenceEl.setAttribute('aria-describedby', id);
    else referenceEl.removeAttribute('aria-describedby');
  }

  $: {
    if (popperPlacement === 'left') bsPlacement = 'start';
    else if (popperPlacement === 'right') bsPlacement = 'end';
    else bsPlacement = popperPlacement;
  }

  $: classes = classnames(
    className,
    'tooltip',
    animation ? 'fade' : false,
    `bs-tooltip-${bsPlacement}`,
    isOpen ? 'show' : false
  );

  $: outer = container === 'inline' ? InlineContainer : Portal;
</script>

{#if isOpen}
  <svelte:component this={outer}>
    <div
      bind:this={floatingEl}
      {...$$restProps}
      class={classes}
      {id}
      role="tooltip"
      x-placement={popperPlacement}
    >
      <div class="tooltip-arrow" data-popper-arrow />
      <div class="tooltip-inner">
        {#if children}
          {children}
        {:else}
          <slot />
        {/if}
      </div>
    </div>
  </svelte:component>
{/if}

<style>
  :global(.floating) {
    width: max-content;
    position: absolute;
    top: 0;
    left: 0;
  }
</style>

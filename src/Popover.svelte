<script>
  import {
    autoUpdate,
    computePosition,
    flip,
    limitShift,
    offset,
    shift
  } from '@floating-ui/dom';
  import { onMount } from 'svelte';
  import classnames from './utils';
  import InlineContainer from './InlineContainer.svelte';
  import Portal from './Portal.svelte';

  let className = '';
  export { className as class };
  export let animation = true;
  export let children = undefined;
  export let container = undefined;
  export let dismissible = false;
  export let isOpen = false;
  export let placement = 'top';
  export let target = '';
  export let title = '';
  export let trigger = 'click';
  let referenceEl;
  let floatingEl;
  let bsPlacement;
  let popperPlacement = placement;
  let cleanup;

  $: {
    if (isOpen && floatingEl) {
      cleanup = autoUpdate(referenceEl, floatingEl, () => {
        floatingEl.style = {
          width: 'max-content',
          position: 'absolute',
          top: 0,
          left: 0
        };
        computePosition(referenceEl, floatingEl, {
          placement: popperPlacement,
          middleware: [offset(8), flip(), shift({ limiter: limitShift() })],
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
  const toggle = () => (isOpen = !isOpen);

  onMount(() => {
    referenceEl = document.querySelector(`#${target}`);
    switch (trigger) {
      case 'hover':
        referenceEl.addEventListener('mouseover', open);
        referenceEl.addEventListener('mouseleave', close);
        break;
      case 'focus':
        referenceEl.addEventListener('focus', open);
        referenceEl.addEventListener('blur', close);
        break;
      default:
        referenceEl.addEventListener('click', toggle);
        if (dismissible) referenceEl.addEventListener('blur', close);
        break;
    }
    return () => {
      switch (trigger) {
        case 'hover':
          referenceEl.removeEventListener('mouseover', open);
          referenceEl.removeEventListener('mouseleave', close);
          break;
        case 'focus':
          referenceEl.removeEventListener('focus', open);
          referenceEl.removeEventListener('blur', close);
          break;
        default:
          referenceEl.removeEventListener('click', toggle);
          if (dismissible) referenceEl.removeEventListener('blur', close);
          break;
      }
    };
  });

  $: if (!target) {
    throw new Error('Need target!');
  }

  $: {
    if (popperPlacement === 'left') bsPlacement = 'start';
    else if (popperPlacement === 'right') bsPlacement = 'end';
    else bsPlacement = popperPlacement;
  }

  $: classes = classnames(
    className,
    'popover',
    animation ? 'fade' : false,
    `bs-popover-${bsPlacement}`,
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
      role="tooltip"
      x-placement={popperPlacement}
    >
      <div class="popover-arrow" data-popper-arrow />
      <h3 class="popover-header">
        <slot name="title">{title}</slot>
      </h3>
      <div class="popover-body">
        {#if children}
          {children}
        {:else}
          <slot />
        {/if}
      </div>
    </div>
  </svelte:component>
{/if}

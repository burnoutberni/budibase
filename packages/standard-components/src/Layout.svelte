<script>
  import { getContext } from "svelte"
  import { Heading, Icon } from "@budibase/bbui"

  const { styleable, linkable, builderStore } = getContext("sdk")
  const component = getContext("component")

  export let title
  export let hideTitle = false
  export let logoUrl
  export let hideLogo = false
  export let navigation = "Top"
  export let sticky = false
  export let links
  export let width = "Large"

  const navigationClasses = {
    Top: "top",
    Left: "left",
    None: "none",
  }
  const widthClasses = {
    Large: "l",
    Medium: "m",
    Small: "s",
  }

  $: validLinks = links?.filter(link => link.text && link.url) || []
  $: typeClass = navigationClasses[navigation] || "none"
  $: widthClass = widthClasses[width] || "l"
  let mobileOpen = false

  const isInternal = url => {
    return url.startsWith("/")
  }

  const ensureExternal = url => {
    return !url.startsWith("http") ? `http://${url}` : url
  }

  const close = () => {
    mobileOpen = false
  }

  const navigateToPortal = () => {
    if ($builderStore.inBuilder) return
    window.location.href = "/builder/apps"
  }
</script>

<div class="layout layout--{typeClass}" use:styleable={$component.styles}>
  {#if typeClass !== "none"}
    <div class="nav-wrapper" class:sticky>
      <div class="nav nav--{typeClass} size--{widthClass}">
        <div class="nav-header">
          {#if validLinks?.length}
            <div class="burger">
              <Icon
                hoverable
                name="ShowMenu"
                on:click={() => (mobileOpen = !mobileOpen)}
              />
            </div>
          {/if}
          <div class="logo">
            {#if !hideLogo}
              <img
                src={logoUrl || "https://i.imgur.com/Xhdt1YP.png"}
                alt={title}
              />
            {/if}
            {#if !hideTitle && title}
              <Heading size="S">{title}</Heading>
            {/if}
          </div>
          <div class="portal">
            <Icon hoverable name="Apps" on:click={navigateToPortal} />
          </div>
        </div>
        <div
          class="mobile-click-handler"
          class:visible={mobileOpen}
          on:click={() => (mobileOpen = false)}
        />
        {#if validLinks?.length}
          <div class="links" class:visible={mobileOpen}>
            {#each validLinks as { text, url }}
              {#if isInternal(url)}
                <a class="link" href={url} use:linkable on:click={close}>
                  {text}
                </a>
              {:else}
                <a class="link" href={ensureExternal(url)} on:click={close}>
                  {text}
                </a>
              {/if}
            {/each}
            <div class="close">
              <Icon
                hoverable
                name="Close"
                on:click={() => (mobileOpen = false)}
              />
            </div>
          </div>
        {/if}
      </div>
    </div>
  {/if}
  <div class="main-wrapper">
    <div class="main size--{widthClass}">
      <slot />
    </div>
  </div>
</div>

<style>
  /*  Main components */
  .layout {
    display: flex;
    flex-direction: column;
    justify-content: flex-start;
    align-items: stretch;
    height: 100%;
    overflow: auto;
  }

  .nav-wrapper {
    display: flex;
    flex-direction: row;
    justify-content: center;
    align-items: stretch;
    background: white;
    z-index: 1;
    box-shadow: 0 0 8px -1px rgba(0, 0, 0, 0.075);
  }
  .layout--top .nav-wrapper.sticky {
    position: sticky;
    top: 0;
    left: 0;
  }

  .nav {
    display: flex;
    flex-direction: column;
    justify-content: flex-start;
    align-items: stretch;
    padding: var(--spacing-xl) 32px;
    max-width: 100%;
    gap: var(--spacing-xl);
  }
  .nav-header {
    flex: 0 0 auto;
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
    gap: var(--spacing-xl);
  }
  .main-wrapper {
    display: flex;
    flex-direction: row;
    justify-content: center;
    align-items: stretch;
    flex: 1 1 auto;
  }
  .main {
    display: flex;
    flex-direction: column;
    justify-content: flex-start;
    align-items: stretch;
    max-width: 100%;
    position: relative;
    padding: 32px;
  }
  .size--s {
    width: 800px;
  }
  .size--m {
    width: 1100px;
  }
  .size--l {
    width: 1400px;
  }

  /*  Nav components */
  .burger {
    display: none;
  }
  .logo {
    display: flex;
    flex-direction: row;
    justify-content: flex-start;
    align-items: center;
    gap: var(--spacing-m);
    flex: 1 1 auto;
  }
  .logo img {
    height: 36px;
  }
  .logo :global(h1) {
    font-weight: 600;
    flex: 1 1 auto;
    width: 0;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }
  .portal {
    display: grid;
    place-items: center;
  }
  .links {
    display: flex;
    flex-direction: row;
    justify-content: flex-start;
    align-items: center;
    gap: var(--spacing-xl);
    margin-top: var(--spacing-xl);
  }
  .link {
    color: var(--spectrum-alias-text-color);
    font-size: var(--spectrum-global-dimension-font-size-200);
    font-weight: 600;
    transition: color 130ms ease-out;
  }
  .link:hover {
    color: var(--spectrum-global-color-blue-600);
  }
  .close {
    display: none;
    position: absolute;
    top: var(--spacing-xl);
    right: var(--spacing-xl);
  }
  .mobile-click-handler {
    display: none;
  }

  /* Desktop nav overrides */
  @media (min-width: 600px) {
    .layout--left {
      flex-direction: row;
      overflow: hidden;
    }
    .layout--left .main-wrapper {
      height: 100%;
      overflow: auto;
    }

    .nav--left {
      width: 250px;
      padding: var(--spacing-xl);
    }

    .nav--left .links {
      margin-top: var(--spacing-m);
      flex-direction: column;
      justify-content: flex-start;
      align-items: stretch;
    }
    .nav--left .link {
      font-size: var(--spectrum-global-dimension-font-size-150);
    }
  }

  /* Mobile nav overrides */
  @media (max-width: 600px) {
    .nav-wrapper {
      position: sticky;
      top: 0;
      left: 0;
      box-shadow: 0 0 8px -1px rgba(0, 0, 0, 0.075);
    }

    /* Show close button in drawer */
    .close {
      display: block;
    }

    /* Force standard top bar */
    .nav {
      padding: var(--spacing-m) 16px;
    }
    .burger {
      display: grid;
      place-items: center;
    }
    .logo {
      flex: 0 0 auto;
    }
    .logo :global(h1) {
      display: none;
    }

    /* Reduce padding */
    .main {
      padding: 16px;
    }

    /* Transform links into drawer */
    .links {
      margin-top: 0;
      position: fixed;
      top: 0;
      left: -250px;
      transform: translateX(0);
      width: 250px;
      transition: transform 0.26s ease-in-out, opacity 0.26s ease-in-out;
      height: 100vh;
      opacity: 0;
      background: white;
      z-index: 999;
      flex-direction: column;
      justify-content: flex-start;
      align-items: stretch;
      padding: var(--spacing-xl);
    }
    .link {
      width: calc(100% - 30px);
      font-size: 120%;
    }
    .links.visible {
      opacity: 1;
      transform: translateX(250px);
      box-shadow: 0 0 40px 20px rgba(0, 0, 0, 0.1);
    }
    .mobile-click-handler.visible {
      position: fixed;
      display: block;
      top: 0;
      left: 0;
      width: 100vw;
      height: 100vh;
      z-index: 998;
    }
  }
</style>

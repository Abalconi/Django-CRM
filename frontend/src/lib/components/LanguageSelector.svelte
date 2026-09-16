<script>
  import { LANGUAGES, getLanguage, setLanguage } from '$lib/i18n.js';

  let language = $state('es');

  $effect(() => {
    language = getLanguage();
  });

  function changeLanguage(nextLanguage) {
    language = nextLanguage;
    setLanguage(nextLanguage);
  }
</script>

<div class="language-picker">
  <span class="language-label">Idioma</span>
  <div class="language-options" role="group" aria-label="Idioma">
    {#each LANGUAGES as item}
      <button
        type="button"
        class:active={language === item.code}
        aria-pressed={language === item.code}
        aria-label={item.label}
        onclick={() => changeLanguage(item.code)}
      >{item.code.toUpperCase()}</button>
    {/each}
  </div>
</div>

<style>
  .language-picker {
    display: flex;
    flex-direction: column;
    align-items: stretch;
    gap: 5px;
    width: 100%;
    padding: 0 8px 12px;
    border-bottom: 1px solid var(--v2-line-soft, #eee8e2);
    color: var(--v2-muted, #6b625c);
    font-size: 10px;
    font-weight: 700;
    letter-spacing: 0.08em;
    text-transform: uppercase;
  }

  .language-options {
    display: flex;
    gap: 16px;
  }

  button {
    flex: 0 0 auto;
    border: 0;
    background: transparent;
    color: var(--v2-slate, #6b625c);
    border-bottom: 2px solid transparent;
    padding: 3px 0 5px;
    font: inherit;
    font-size: 11px;
    letter-spacing: 0;
    cursor: pointer;
  }

  button:hover,
  button.active {
    color: var(--v2-ink, #24201d);
    border-bottom-color: var(--v2-ember, #b8542d);
  }
</style>

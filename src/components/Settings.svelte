<script lang="ts">
  import Footer from './Footer.svelte';
  import { favorites } from '../lib/player';
  import type { FavoriteStore } from 'src/types/FavoritesStore';
  import ActionButton from '../lib/ActionButton.svelte';
  import { fade } from 'svelte/transition';

  import Modal from './Modal.svelte';

  const favoritesVersion = '1.0.0';
  interface FavoritesExport {
    __app__: 'musicale';
    version: string;
    favorites: FavoriteStore[];
  }
  let importMessage = '';
  let displayImportWarning = false;

  let noFavorites = true;

  $: noFavorites = $favorites.length === 0 ? true : false

  function exportFavorites() {
    const data = JSON.stringify({
      __app__: 'musicale',
      version: favoritesVersion,
      favorites: $favorites,
    } as FavoritesExport);
    const blob = new Blob([data], { type: 'application/json' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.download = 'favorites.musicale.json';
    a.href = url;
    a.click();
    setTimeout(() => URL.revokeObjectURL(url), 5000);
  }
  function importFavorites(force) {
    if (!noFavorites && force !== true) {
      displayImportWarning = true;
      return;
    }
    const input = document.createElement('input');
    input.type = 'file';
    input.accept = 'application/json';
    input.onchange = () => {
      importMessage = '';
      const file = input.files[0];
      const reader = new FileReader();
      reader.onload = () => {
        try {
          const data: FavoritesExport = JSON.parse(reader.result as string);
          if (
            data.__app__ !== 'musicale' ||
            data.version !== favoritesVersion
          ) {
            importMessage = 'Import failed. Invalid file.';
            return;
          }
          favorites.set(data.favorites);
          importMessage = 'Import successful.';
        } catch (err) {
          importMessage = 'Import failed. Invalid file.';
        }
      };
      reader.readAsText(file);
    };
    input.click();
  }
</script>

<main>
  <!-- modal displayed to alert favorites import will remove the present ones -->
  <Modal closed={!displayImportWarning} closable={false}>
    <div slot="title">Favorites already present</div>
    <p>
      Seems there are some favorites already saved on Musicale.<br />By importing
      new ones, all the olds will be lost.
    </p>
    <p>
      If you want to keep the olds, press "Cancel" to abort.
    </p>
    <div slot="content__bottom">
      <div class="flex-buttons">
        <ActionButton
        title="Import"
        backgroundColor="var(--back-color)"
        scale="0.8"
        on:click={() => {
          displayImportWarning = false;
          importFavorites(true);
        }}
      />
      <ActionButton
        title="Cancel"
        backgroundColor="var(--back-color)"
        scale="0.8"
        on:click={() => {
          displayImportWarning = false;
        }}
      />
      </div>
    </div>
  </Modal>
  <div class="settings">
    <div class="settings__title">Settings</div>
    <div class="settings__content">
      <div class="content__favorites-title">Manage favorites</div>
      <div class="content__message">
        Since Musicale saves all your favorites locally, you probably want to
        export them if you're planning to use a new device.
      </div>
      <div class="content__buttons">
        <ActionButton
          title="Export Favorites"
          on:click={() => noFavorites ? null : exportFavorites()}
          color="#fff"
          fitContent={false}
          scale="0.9"
          disabled={noFavorites}
          hoverTitle={noFavorites ? 'No favorites to export' : null}
        />
        <ActionButton
          title="Import Favorites"
          on:click={importFavorites}
          color="#fff"
          fitContent={false}
          scale="0.9"
        />
      </div>
      {#if importMessage}
        <div class="import-message" in:fade>{importMessage}</div>
      {/if}
    </div>
  </div>
  <Footer size="small" />
</main>

<style>
  main {
    display: grid;
    padding: 2em;
    min-height: calc(100vh - var(--bars-height) * 2);
    align-content: space-between;
  }
  .settings__title {
    font-size: var(--fs-big);
    font-weight: 700;
  }
  .settings__content {
    max-width: 500px;
    max-width: 50ch;
  }
  .content__favorites-title {
    font-size: var(--fs-medium);
    font-weight: 700;
    margin-bottom: 1rem;
  }
  .content__message {
    margin-bottom: 1rem;
  }
  .content__buttons {
    display: grid;
    grid-template-columns: 1fr 1fr;
    padding-inline: 5em;
    gap: 1em;
  }
  .import-message {
    text-align: center;
  }
  * + * {
    margin-top: 0.7em;
  }
  .flex-buttons {
    display: flex;
    flex-direction: row;
    gap: 1em;
  }
</style>

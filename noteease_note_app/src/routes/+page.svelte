<script>
  // PUBLIC_INTERFACE
  /**
   * The main container for the NoteEase application.
   * Features: Create, edit, delete, categorize, and search notes.
   * Theme: Light, with color scheme - primary: #4A90E2, secondary: #FFFFFF, accent: #F5A623
   */
  import { onMount } from 'svelte';

  // Note model example: { id, title, content, categories: [string], created, updated }
  let notes = [];
  let searchTerm = '';
  let selectedNote = null; // null or note object
  let isEditing = false;
  let newNoteMode = false;

  let availableCategories = [
    { name: 'Personal', color: '#F5A623' },
    { name: 'Work', color: '#4A90E2' },
    { name: 'Ideas', color: '#7ED957' }
  ];
  let noteCategories = [];

  // Basic localstorage persistence (optional)
  onMount(() => {
    const stored = localStorage.getItem('noteease-notes');
    if (stored) {
      notes = JSON.parse(stored);
    }
  });

  function persist() {
    localStorage.setItem('noteease-notes', JSON.stringify(notes));
  }

  // PUBLIC_INTERFACE
  function openNewNote() {
    selectedNote = { id: Date.now(), title: '', content: '', categories: [], created: new Date(), updated: new Date() };
    isEditing = true;
    newNoteMode = true;
    noteCategories = [];
  }

  // PUBLIC_INTERFACE
  function selectNote(note) {
    selectedNote = { ...note };
    noteCategories = [...(note.categories || [])];
    isEditing = false;
    newNoteMode = false;
  }

  // PUBLIC_INTERFACE
  function saveNote() {
    selectedNote.categories = [...noteCategories];
    if (newNoteMode) {
      notes = [{ ...selectedNote }, ...notes];
    } else {
      notes = notes.map((n) => n.id === selectedNote.id ? { ...selectedNote, updated: new Date() } : n);
    }
    persist();
    isEditing = false;
    newNoteMode = false;
    selectedNote = null;
    noteCategories = [];
  }

  // PUBLIC_INTERFACE
  function editNote() {
    isEditing = true;
    newNoteMode = false;
  }

  // PUBLIC_INTERFACE
  function deleteNote(id) {
    notes = notes.filter((n) => n.id !== id);
    persist();
    selectedNote = null;
    isEditing = false;
    newNoteMode = false;
    noteCategories = [];
  }

  // PUBLIC_INTERFACE
  function closeNote() {
    selectedNote = null;
    isEditing = false;
    newNoteMode = false;
    noteCategories = [];
  }

  // PUBLIC_INTERFACE
  function toggleCategory(cat) {
    if (noteCategories.includes(cat.name)) {
      noteCategories = noteCategories.filter((c) => c !== cat.name);
    } else {
      noteCategories = [...noteCategories, cat.name];
    }
  }

  // Derived: Filtered notes list
  $: filteredNotes = notes.filter(note =>
    note.title.toLowerCase().includes(searchTerm.toLowerCase()) ||
    note.content.toLowerCase().includes(searchTerm.toLowerCase())
  );
</script>

<style>
  :global(body) {
    background: #f6f8fb;
    margin: 0;
    font-family: 'Inter', system-ui, sans-serif;
    color: #222;
  }
  .container-main {
    max-width: 800px;
    margin: 40px auto 0;
    background: #fff;
    border-radius: 18px;
    box-shadow: 0 6px 32px 0 rgba(50,80,130,0.090);
    padding: 32px 24px 56px 24px;
    min-height: 70vh;
    position: relative;
  }
  .header-bar {
    display: flex;
    align-items: center;
    gap: 12px;
    margin-bottom: 28px;
  }
  .app-title {
    font-size: 2rem;
    font-weight: 800;
    color: #4A90E2;
    flex: 1;
  }
  .search-bar {
    flex: 2;
    border: 1px solid #d1d6dd;
    padding: 8px 16px;
    border-radius: 24px;
    font-size: 1rem;
    outline: none;
    background: #f7fafc;
    transition: border .2s;
  }
  .notes-list {
    margin-bottom: 60px;
  }
  .note-card {
    background: #f8fafc;
    border-radius: 12px;
    padding: 20px 18px;
    margin-bottom: 18px;
    box-shadow: 0 2px 14px 0 rgba(74,144,226,0.07);
    cursor: pointer;
    border-left: 4px solid #4A90E2;
    position: relative;
    transition: box-shadow 0.18s;
  }
  .note-card:hover {
    box-shadow: 0 3px 22px 0 rgba(74,144,226,0.13);
  }
  .note-card-title {
    font-size: 1.07rem;
    font-weight: 600;
    margin-bottom: 5px;
  }
  .note-card-snippet {
    font-size: 1rem;
    color: #53576a;
    margin-bottom: 8px;
  }
  .note-card-categories {
    display: flex;
    gap: 7px;
    flex-wrap: wrap;
  }
  .category-label {
    font-size: 0.88em;
    padding: 3px 10px;
    border-radius: 14px;
    color: #fff;
    font-weight: 500;
    background: var(--tag-color, #aaa);
    display: inline-block;
  }
  .fab-btn {
    position: fixed;
    right: 42px;
    bottom: 42px;
    width: 64px;
    height: 64px;
    border-radius: 50%;
    background: #4A90E2;
    color: #fff;
    font-size: 2.2rem;
    display: flex;
    justify-content: center;
    align-items: center;
    box-shadow: 0 5px 30px 0 rgba(74,144,226,0.23);
    border: none;
    cursor: pointer;
    z-index: 10;
    transition: background .18s;
  }
  .fab-btn:hover {
    background: #3169b3;
  }

  /* Modal/Editor styling */
  .modal-bg {
    position: fixed;
    top: 0; left: 0; width: 100vw; height: 100vh;
    background: rgba(0,0,0,0.09);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 11;
  }
  .modal-contents {
    background: #fff;
    border-radius: 18px;
    box-shadow: 0 10px 40px 0 rgba(50,80,130,0.17);
    padding: 30px 32px 24px 32px;
    min-width: 350px;
    max-width: 480px;
    min-height: 180px;
  }
  .input-title {
    font-size: 1.13rem;
    padding: 8px 10px 6px 10px;
    margin-bottom: 12px;
    border: 1px solid #e1e6ed;
    border-radius: 8px;
    outline: none;
    width: 100%;
    font-weight: 600;
    background: #f6fafd;
  }
  .input-content {
    font-size: 1.02rem;
    padding: 10px 10px 36px 10px;
    border: 1px solid #e1e6ed;
    border-radius: 10px;
    outline: none;
    width: 100%;
    min-height: 95px;
    margin-bottom: 17px;
    resize: vertical;
    background: #f6fafd;
  }
  .category-selectors {
    display: flex;
    gap: 10px;
    flex-wrap: wrap;
    margin-bottom: 18px;
  }
  .category-select {
    border: none;
    background: none;
    outline: none;
    cursor: pointer;
  }
  .category-label.selected {
    filter: brightness(1.15) saturate(2.5);
    border: 2px solid #d5e6fd;
  }
  .editor-actions {
    display: flex;
    justify-content: flex-end;
    gap: 18px;
    margin-top: 15px;
  }
  .btn {
    padding: 8px 18px;
    border-radius: 19px;
    font-weight: 600;
    font-size: 1rem;
    border: none;
    cursor: pointer;
    background: #4A90E2;
    color: #fff;
    margin-left: 8px;
    transition: background .19s;
  }
  .btn.secondary {
    background: #bfc5ce;
    color: #444;
  }
  .btn.danger {
    background: #F5A623 !important;
    color: #fff;
  }
</style>

<div class="container-main">
  <div class="header-bar">
    <span class="app-title">NoteEase</span>
    <input
      class="search-bar"
      type="text"
      placeholder="Search notes..."
      bind:value={searchTerm}
      aria-label="Search notes"
    />
  </div>

  <div class="notes-list">
    {#if filteredNotes.length > 0}
      {#each filteredNotes as note (note.id)}
        <div class="note-card" on:click={() => selectNote(note)}>
          <div class="note-card-title">{note.title}</div>
          <div class="note-card-snippet">{note.content.slice(0, 72)}{note.content.length > 72 ? '…' : ''}</div>
          <div class="note-card-categories">
            {#each note.categories ?? [] as cat}
              <span
                class="category-label"
                style="--tag-color: {availableCategories.find(c=>c.name===cat)?.color || '#aaa'}"
                >{cat}</span>
            {/each}
          </div>
        </div>
      {/each}
    {:else}
      <div style="opacity:0.64;padding:46px 0;text-align:center;">No notes found.</div>
    {/if}
  </div>
</div>

<button class="fab-btn" aria-label="Add new note" on:click={openNewNote}>
  +
</button>

{#if selectedNote}
  <div class="modal-bg" on:click|self={closeNote}>
    <div class="modal-contents" on:click|stopPropagation>
      {#if isEditing}
        <input
          class="input-title"
          placeholder="Title"
          bind:value={selectedNote.title}
          maxlength="64"
        />
        <textarea
          class="input-content"
          placeholder="Write your note…"
          bind:value={selectedNote.content}
        />
        <div class="category-selectors">
          {#each availableCategories as cat}
            <button
              class="category-select"
              type="button"
              aria-label={"Toggle category " + cat.name}
              on:click={() => toggleCategory(cat)}
            >
              <span
                class="category-label {noteCategories.includes(cat.name) ? 'selected' : ''}"
                style="--tag-color: {cat.color}"
              >{cat.name}</span>
            </button>
          {/each}
        </div>
        <div class="editor-actions">
          <button class="btn secondary" on:click={closeNote}>Cancel</button>
          {#if !newNoteMode}
            <button class="btn danger" on:click={() => deleteNote(selectedNote.id)}>Delete</button>
          {/if}
          <button class="btn" on:click={saveNote}>Save</button>
        </div>
      {:else}
        <h2>{selectedNote.title}</h2>
        <div style="margin-bottom:10px;font-size:1.03rem;color:#555;">
          {selectedNote.content}
        </div>
        <div class="note-card-categories" style="margin-bottom:20px;">
          {#each selectedNote.categories ?? [] as cat}
            <span
              class="category-label"
              style="--tag-color: {availableCategories.find(c=>c.name===cat)?.color || '#aaa'}"
              >{cat}</span>
          {/each}
        </div>
        <div class="editor-actions">
          <button class="btn secondary" on:click={closeNote}>Close</button>
          <button class="btn" on:click={editNote}>Edit</button>
          <button class="btn danger" on:click={() => deleteNote(selectedNote.id)}>Delete</button>
        </div>
      {/if}
    </div>
  </div>
{/if}

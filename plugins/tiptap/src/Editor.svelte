<script lang="ts">
  import { onDestroy, onMount, createEventDispatcher } from 'svelte'

  import { Editor, type Content } from '@tiptap/core'
  import StarterKit from '@tiptap/starter-kit'
  import BubbleMenu from '@tiptap/extension-bubble-menu'
  import FloatingMenu from '@tiptap/extension-floating-menu'
  import Link from '@tiptap/extension-link'
  import Typography from '@tiptap/extension-typography'

  import H1 from './icons/h-1.svelte'
  import H2 from './icons/h-2.svelte'
  import H3 from './icons/h-3.svelte'
  import Paragraph from './icons/paragraph.svelte'
  import Bold from './icons/bold.svelte'
  import Italic from './icons/italic.svelte'
  import Strikethrough from './icons/strikethrough.svelte'
  import FormatClear from './icons/format-clear.svelte'
  import LinkUnlink from './icons/link-unlink.svelte'
  import ListOrdered from './icons/list-ordered.svelte'

  const emit = createEventDispatcher()

  export let content: Content
  export let editable: boolean = true

  let editor: Editor
  let element: HTMLDivElement
  let floatingMenu: HTMLDivElement
  let bubbleMenu: HTMLDivElement

  onMount(() => {
    editor = new Editor({
      editable,
      element,
      content,
      extensions: [
        StarterKit,
        Typography,
        BubbleMenu.configure({
          element: bubbleMenu,
          tippyOptions: {
            duration: 100,
          },
        }),
        FloatingMenu.configure({
          element: floatingMenu,
          tippyOptions: {
            duration: 100,
          },
        }),
        Link.configure({
          autolink: true,
          linkOnPaste: true,
          openOnClick: true,
          HTMLAttributes: {
            rel: 'noopener noreferrer',
            target: '_blank',
            style:
              'color: blue; text-decoration: dotted; text-decoration-line: underline;',
          },
        }),
      ],

      // force re-render so `editor.isActive` works as expected
      onTransaction() {
        editor = editor
      },
      onUpdate({ editor }) {
        emit('update', editor.getJSON())
      },
    })
  })

  onDestroy(() => {
    editor && editor.destroy()
  })
</script>

<div class="editor" style="width: 100%">
  <div bind:this={floatingMenu} class="menu floating-menu">
    <button
      on:click={() => editor?.chain().focus().toggleHeading({ level: 1 }).run()}
      class:active={editor?.isActive('heading', { level: 1 })}
    >
      <H1 />
    </button>
    <button
      on:click={() => editor?.chain().focus().toggleHeading({ level: 2 }).run()}
      class:active={editor?.isActive('heading', { level: 2 })}
    >
      <H2 />
    </button>
    <button
      on:click={() => editor?.chain().focus().toggleHeading({ level: 3 }).run()}
      class:active={editor?.isActive('heading', { level: 3 })}
    >
      <H3 />
    </button>
    <button
      on:click={() => editor?.chain().focus().toggleBulletList().run()}
      class:active={editor?.isActive('bulletList')}
    >
      <ListOrdered />
    </button>
  </div>

  <div bind:this={bubbleMenu} class="menu bubble-menu">
    <button
      on:click={() => editor?.chain().focus().toggleBold().run()}
      class:active={editor?.isActive('bold')}
    >
      <Bold />
    </button>
    <button
      on:click={() => editor?.chain().focus().toggleItalic().run()}
      class:active={editor?.isActive('italic')}
    >
      <Italic />
    </button>
    <button
      on:click={() => editor?.chain().focus().toggleStrike().run()}
      class:active={editor?.isActive('strike')}
    >
      <Strikethrough />
    </button>
    <button on:click={() => editor?.commands.unsetAllMarks()}>
      <FormatClear />
    </button>
    {#if editor?.isActive('link')}
      <button
        on:click={() => editor?.commands.unsetLink()}
        class:active={editor?.isActive('link')}
      >
        <LinkUnlink />
      </button>
    {/if}
  </div>

  <div bind:this={element} />
</div>

<style>
  .editor .menu {
    display: flex;
    align-items: center;
    justify-content: center;
  }
  .editor .menu button {
    display: flex;
    align-items: center;
    justify-content: center;

    appearance: none;
    background-color: transparent;

    border: 1px solid transparent;
    border-radius: 3px;

    transition: all 0.15s ease-in-out;
  }

  .editor .menu.floating-menu {
    background-color: rgba(255, 255, 255, 0.9);
    border-radius: 5px;
    padding: 0.35rem;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.12);
  }
  .editor .menu.floating-menu button {
    color: rgb(60, 60, 60);
  }
  .editor .menu.floating-menu button:hover {
    background-color: rgba(0, 0, 0, 0.12);
  }
  .editor .menu.floating-menu button.active {
    color: rgba(21, 21, 21);
    background-color: rgba(0, 0, 0, 0.05);
    border-color: rgb(60, 60, 60);
  }

  .editor .menu.bubble-menu {
    background-color: rgba(0, 0, 0, 0.9);
    border-radius: 5px;
    padding: 0.25rem;
    color: white;
  }
  .editor .menu.bubble-menu button {
    color: white;
  }
  .editor .menu.bubble-menu button:hover {
    background-color: rgba(255, 255, 255, 0.12);
  }
  .editor .menu.bubble-menu button.active {
    background-color: rgba(255, 255, 255, 0.05);
    border-color: rgb(255, 255, 255, 0.25);
  }

  .editor-link {
    text-decoration: dotted !important;
    color: rgb(12, 52, 230) !important;
  }
</style>
